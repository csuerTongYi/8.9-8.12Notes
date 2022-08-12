###### 1，redis定义：

​	（1）开源、C编写、支持网络交互、持久化的key-value的数据库，可以用作缓存、数据库和消息中间件。
​	（2）value支持多种数据类型，字符串（strings）、散列（hash）、列表（list）、集合（set）、有序集合（sorted set）

###### 2，redis特性

​	（1）速度快，
​		单节点读11万次/秒，写8万1千次/秒
​		数据存放在内存中
​	（2）持久化
​		数据的更新将异步保存到硬盘。
​		a.什么是==持久化==？-->RDB(Redis DataBase)，AOF(Append Only File)。
​			i.RDB持久化是指在指定的时间间隔内将内存中的数据集快照写入磁盘，默认。
​			ii.默认情况下Redis没有开启AOF方式的持久化，开启AOF持久化后，每执行一条会更改Redis中的数据的命令，Redis就会将该命令写入硬盘中的AOF文件
​	（3）支持多种编程语言
​	（4）操作原子性

###### 3，redis常使用在缓存、消息队列、计数器、排行榜等。

###### 4，strings数据类型

​	（1）基础的数据类型，在遇到数值操作时，如果strings是“数值”，redis会将字符串类型转为数值。string是动态字符串（类似java的ArrayList），采用的是预分配内存空间的方式。
​	（2）常见命令：
​		set key value，设置key和value
​		get key，通过key获取value
​		mset key value key value ...，批量设置key和value
​		mget key key key ...，通过key批量获取value
​		incr key，自增计数
​		decr，自减
​		incrby，自增指定数字
​		decr，自减指定数字
​	（3）使用广泛，常用于存储json序列化后的字符串，再放进redis进行缓存。使用场景：

​			缓存经常要查询的数据，如登录用户信息

​			计数器，如论坛点赞数评论数转发数

​			共享session，在负载均衡的情况下，不同的用户登录信息可能保存在不同的服务器上，但是如果网络刷新就会重新分配资源使得用户重新登录，此时可以用redis将用户的session进行集中管理，每次关于用户登录信息可以在redis上读取。

###### 5，hash

​		（1）是一个string类型的field（字段）和value（属性）的映射表，一个hash可以存放多个field和value，适合用于存储对象。底层数据结构是ziplist和dict。ziplist是压缩链表，连续的内存空间，元素紧密存储，无空隙，在hash对象存储的值较小的时候使用。dict为字典，为了扩展哈希表(rehash)的时候，能够方面的操作哈希表。

​		（2）redis在rehash的时候采用的是==渐进式rehash==的方法。渐进式 rehash 会在 rehash 的同时，保留新旧两个 hash 结构，查询时会同时查询两个 hash 结构，在后续的定时任务中以及 hash 的子指令中，逐渐将旧 hash 的内容迁移到新的 hash 结构中。旧hash的最后一个元素被移除后，内存回收。

​		（3）常用命令：

​				hset key field value：通过设置key对应的field设置value

​				hget key field：通过key对应的field获取value

​				hdel key field1 field2 ...：删除field

​				hlen key：计算 field 个数

​				hmset key field1 value1 field2 value2：批量设置 field-value

​				hmget key field1 value1 field2 value2：批量获取 field-value

​				hexists key field：判断key下的field是否存在 || key是否存在

​				hkeys key：获取key对应的所有 field

​				hvals key：获取key对应的所有 value

​				hgetall key：获取所有的 field-value

```c
127.0.0.1:6379> hset ali first taobao
(integer) 1
127.0.0.1:6379> hset ali second zhifubao
(integer) 1
127.0.0.1:6379> hset ali third mayijinrong
(integer) 1
127.0.0.1:6379> hget ali second
"zhifubao"
127.0.0.1:6379> hget ali third
"mayijinrong"
127.0.0.1:6379> hdel ali first second
(integer) 2
127.0.0.1:6379> hget ali first
(nil)
127.0.0.1:6379> hget ali second
(nil)
127.0.0.1:6379> hget ali third
"mayijinrong"
127.0.0.1:6379> hexists ali first
(integer) 0
127.0.0.1:6379> hexists ali third
(integer) 1
127.0.0.1:6379> hset ali forth aliyun
(integer) 1
127.0.0.1:6379> hkeys ali
1) "third"
2) "forth"
127.0.0.1:6379> hvals ali
1) "mayijinrong"
2) "aliyun"
127.0.0.1:6379> hgetall ali
1) "third"
2) "mayijinrong"
3) "forth"
4) "aliyun"
```

​				（4）使用场景：

​					用于对象存储，存储对象信息，如对象属性的属性名为field，属性值为value

​					用于存储用户购买记录，如用户信息id为key，商品信息id为field，商品数量为value

###### 5，list

​	（1）list在redis中是一个双端列表，相当于java里的LinkedList，同理插入删除快，查询慢。当列表弹出最后一个元素后，回收内存。

​	（2）使用列表的一些方法

```java
lpush + lpop = Stack (左进左出)
lpush + rpop = Queue （左进右出）
lpush + brpop = Message Queue (消息队列)
lpush + ltrim = Capped Collection (有限集合)
// lpush：将一个或多个值插入到列表头部。（左）
// rpush：将一个或多个值插入到列表尾部。（右）
// rpop：从列表的右端弹出一个值，并返回被弹出的值
// lpop：从列表的左端弹出一个值，并返回被弹出的值
// lrange：获取列表在给定范围上的所有值
// lindex：通过索引获取列表中的元素
```

​		（3）使用场景：

​				电商软件的热销榜

​				文章列表

​				实现工作队列，使用list的push操作将任务放在列表里，再用pop取出list的任务，如消息队列==MQ==。

###### 6，set

​	（1）无序的，自动去重，底层用两种数据结构：dict+intset。当集合中最后一个元素被删除，回收内存。

​	（2）常用命令：

​		sadd key value：向集合添加一个或多个成员

​		scard key：获取集合的成员数

​		smembers key：获取集合的所有成员

​		sismember key member：判断member是否是key集合的成员。

###### 7，zset

​	（1）集合和有序集合基本一致，zset的每个元素都会关联一个double类型的权重参数--score，使得集合中的元素根据score的值进行有序排列。元素少的时候底层使用ziplist，元素多的时候底层使用字典dict和跳表==skiplist==。

​	（2）有序集合的元素不能重复，但是score能够重复。

​	（3）常用命令：

​			zadd key score1 member1 score2 member2...：添加成员，返回结果代表成功添加成员的个数 

​			zcard key：计算成员个数

​			zrank key member 和 zrevrank key member：计算成员的排名，zrank 从低到高，zrevrank 从高到低

###### 8，





































###### 参考文献

[1]黄健宏. Redis设计与实现 : The design and implementation of Redis[M]. 机械工业出版社, 2014.

​		







​				
​	