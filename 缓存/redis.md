* [一、NoSQL](#nosql)
  * [Redis概述](#redis概述)
  * [Redis指令](#redis指令)
* [二、Redis的特性](#redis的特性)
  * [多数据库](#多数据库)
  * [支持事务](#支持事务)
  * [持久化](#持久化)
  * [数据淘汰策略](#数据淘汰策略)
  * [事件驱动](#事件驱动)
  * [复制](#复制)
  * [Redis单线程](#redis单线程)
  * [redis渐进式哈希](#redis渐进式哈希)
* [三、Redis的性能](#redis的性能)
* [四、Redis哨兵](#redis哨兵)
* [五、Redis底层数据结构](#redis底层数据结构)
* [六、Redis分布式锁](#redis分布式锁)
* [七、Redis常见问题](#redis常见问题)
* [八、Redis的发布订阅](#redis的发布订阅)
* [九、pipeline](#pipeline)
* [十、cluster](#cluster)
* [十一、jedis](#jedis)




# NoSQL

NoSQL:not only sql非关系型数据库

* 支持高并发读写：关系数据库应付不了高并发
* 支持对海量数据的高效率存储和访问
* 支持高可扩展性和高可用性

NoSQL数据库分类：
* 键值对存储：快速查询   存储的数据缺少结构化
* 列存储：查找快 扩展性强 功能局限
* 文档数据库：查询性不是很高
* 图形数据库

特点：易扩展

## Redis概述
支持value的数据类型：

字符串类型 String

列表类型 List：按照插入顺序排序

有序集合类型 sorted-set：与set相比，这其中的每个成员都有一个分数，redis通过这个分数排序，分数可以重复。成员有序，因此，即使访问中部的成员，也是非常高效的。这个类型和set类型都支持不同集合之间的聚合操作。

散列类型 Hash：键值对类型

集合类型 Set：不允许出现相同的元素

应用场景：缓存、任务列表、网站访问统计、数据过期处理、分布式集群架构中session分离

## Redis指令
### String
set key value 新建对象

get key 获取对象

getset key value 先获取值再修改值为value

del key 删除

incr key 对value的值加1（如果是字符串，不能加1，会抛出异常）

incrby key n对value的值加n

decrby key n对value的值减n


### Hash 
键值对

hset key1  key2 value新建对象 若key1 key2都存在则返回0

hmset key1 key2 value2  key3  value3新建多个对象

hgetall key 获取所有对象

hdel key key1删除键为key1的成员

hincrby key key1 n对key的成员中，键为可以的对象的值加n

hexists key key1 判断key中是否有键为key1的对象

hlen key 获取其中的所有对象数量

hkeys key 获取key中所有对象的key

hvalues key 获取key中所有对象的value

### List 
按照插入顺序排序

rpush key value 对key的序列从右边插入一个value

lpush key value 对key的序列从左边插入一个value

lrange key site1 site2从左开始显示从site1到site2的元素

lindex key site1 显示位置为site1的成员

lpop key 弹出最左边的成员（原子操作）

llen key 返回对象的数量


### Set 
不允许出现重复的元素 不同set中的聚合操作

sadd key value 添加

srem key value 删除

smembers key 获取key对应的所有的值

sismember key value 判断value是否在key的值中，在为1，不在为0

sdiff key1 key2 差集运算，而且和key的顺序有关，是key1-key2

sinter key1 key2 交集运算

sunion key1 key2 并集运算

scard key 获得对应value的数量

srandmember key 随机返回一个成员

sdiffstore  key1 key2 key3 把key2和key3的差集存到key1中

sinterstore key1 key2 key3 把key2和key3的交集存到key1中

sunionstore key1 key2 key3 把key2和key3的并集存到key1中


### sorted-set 
与sort的主要区别是，这其中的每个成员都有一个分数，redis通过这个分数排序，分数可以重复。成员有序，因此，即使访问中部的成员，也是非常高效的

zadd key  分数  value 添加

zscore key value 获取对应成员的分数

zcard key 获取成员的数量

zrem key value 删除成员

zrange key start end |withscores（显示分数）范围查找（start和end是位置的范围）

zrevrange key start end |withscores（显示分数）范围查找并排序（start和end是位置的范围）

zremrangebyrank key start end 按照（位置）范围进行删除

zremrangebyscore key start end 按照分数排名进行删除

zrange key start stop [withscore]：把集合排序后,返回名次[start,stop]的元素  默认是升续排列，在z后面加rev即为降序排列  withscores 是把score也打印出来

zrangebyscore key start end  withscore 按照分数范围显示value和对象

zincrby key 常数n value  给某个成员的分数加n

zcount key small big 显示分数在small ,big之间的成员的个数
使用场景：大型积分排行榜 构建索引数据


### 对keys的通用操作：
keys * 获取所有的key

keys my?获取所有以my开头的key

del key 删除某个key

exist key 查看某个key是否存在

rename key newkey对key重命名

expire key 时间长度n 设置超时时间 单位为秒

pexpire key 整数值：设置key的生命周期 毫秒为单位

perisist key：把指定key设置为永久有效

ttl key 查看key所剩的时间 返回为正数为剩余时间 -1为无过期时间 -2为已过期

type key 查看key的类型

### 一些特殊操作
SETNX key value 若key不存在时成功返回 否则失败返回0 可用于分布式加锁

SETEX key seconds value  设置key value对，且定义过期时间，如果key已存在则覆盖value。

# Redis的特性

## 多数据库
一个redis实例可以包含多个数据库，客户端可以指定连接哪个数据库，一个redis实例最多提供16个数据库。

通过select n指令切换到第n+1个数据库

## 支持事务
事务执行期间，不会对其他提供服务 

multi:该语句后面的都看作事务的操作

exec：相当于提交

discard：回滚

## 持久化
数据从内存中同步到硬盘上的过程。

两种方式：RDB AOF
### RDB方式
默认支持，不需配置。

在指定的时间间隔内，将数据库的快照写入磁盘。

优势： 1.redis数据库只有一个文件，方便备份。 

     2.对于灾难恢复而言，能轻松的把文件转移。
     
     3.性能最大化，通过分化子进程（fork）实现。 
     
缺点：1.无法最大限度保证数据的完整性，因为系统一定会在持久化之前出现宕机的情况。

     2.数据集很大时，需要服务器停止
     
RDB的原理：
fork和cow。fork是指redis通过创建子进程来进行RDB操作，cow指的是copy on write，子进程创建后，父子进程共享数据段，父进程继续提供读写服务，写脏的页面数据会逐渐和子进程分离开来。

### AOF方式
日志的形式记录服务器的操作，在启动之初会读取日志，并初始化数据

优势：
     * 1.这种机制有更高的数据安全性
     * 2.采C aa用append模式写日志  写入的时候出现宕机不会破坏已经写入的数据  写了一半崩溃，下次启动redis的时候通过redis-check-aof工具解决数据一致性的问题
     * 3.日志过大，redis自动启动重写机制。redis还会创建一个新的日志，记录重写期间的数据操作，保证重写期间的数据操作不会丢失。
     * 4.日志文件清晰，可以通过这个文件实现数据的重建
     
缺点：
     * 1.AOF文件更大
     * 2.根据同步策略的原因，AOF效率低于RDB

redis提供了三种同步策略：
* 每秒同步：异步完成，效率很高，一旦系统宕机，这一秒钟内修改的数据丢失
* 每修改同步：同步持久化 每一次数据的变化会马上记录到磁盘中 效率很低 但是安全
* 不同步

aof流程:
* 1） 所有的写入命令会追加到aof_buf（缓冲区）中。
* 2） AOF缓冲区根据对应的策略向硬盘做同步操作。
* 3） 随着AOF文件越来越大，需要定期对AOF文件进行重写，达到压缩的目的。
* 4） 了解AOF工作流程之后，下面针对每个步骤做详细介绍。

## 数据淘汰策略
可以设置最大使用量，当内存使用量超出的时候会采用淘汰策略。
<div align="center"> <img src="https://github.com/RJianPeng/Technology-Stack/blob/master/%E7%BC%93%E5%AD%98/photo/redis%E6%B7%98%E6%B1%B0%E7%AD%96%E7%95%A5.png"/></div><br>


## 事件驱动
Redis是事件驱动的数据库

### 文件事件
服务器通过套接字和多个客户端或服务端进行通信，基于reactor模式，使用多路复用监听多个套接字，并将到达的事件传送给事件分派器,分派器会根据套接字产生的事件类型调用相应的事件处理器。 
<div align="center"> <img src="https://github.com/RJianPeng/Technology-Stack/blob/master/%E7%BC%93%E5%AD%98/photo/%E6%96%87%E4%BB%B6%E4%BA%8B%E4%BB%B6.png"/></div><br>



### 时间事件
服务器有一些操作需要在给定的时间点进行，时间事件是对这一类操作的抽象。
时间事件又分为：定时事件和周期事件
redis将时间事件放在一个链表里面，通过遍历找出到达的时间事件执行。

### 事件处理
对于文件事件，redis需要不断监听才能得到到达的事件，但是不能一直监听，否则就无法执行时间事件，所以redis的事件处理操作是：获取最近的时间事件，获取剩余时间，根据剩余时间计算出轮询文件事件的时间

## 复制
slaveof host port 让该服务器成为host服务器的从服务器

一个服务器只能有一个主服务器

主从服务器连接过程：
* 1.主服务器创建快照文件，发送给从服务器，并在缓冲区记录发送期间的数据库操作，发送完快照文件之后发送缓冲区操作。
* 2.从服务器丢弃所有旧的数据，加载新的文件，以及主服务器发送的操作指令
* 3.主服务器每进行一次操作，就向从服务器发送一次相同的操作指令。

主从链：一个主服务器无法负担多个从服务器，可以通过中间服务器实现。

## Redis单线程
redis单线程的意思是说，在处理网络请求模块，redis只使用了一个线程来进行处理，但是在其他的模块，redis还会使用更多的线程来进行处理。redis通过任务封闭，把多个任务封装到了一个线程上进行处理，对于需要多个操作的任务来说，可能会存在多个任务的操作交叉进行。
这样节省了线程切换的开销，速度更快。

采用I/O多路复用，epoll

## redis渐进式哈希
在redis的数据量非常大的情况下，需要把rehash的操作分很多次的进行，否则会对服务器造成过大的压力

详细步骤：

1）为ht[1]分配空间，让字典同时持有ht[1]和ht[0]两个哈希表。

2）在字典中维持一个索引计数器变量rehashidx，并将它的值设置为0，表示rehash工作正式开始

3）在Rehash期间，每次对字典的执行添加，删除，查找或者更新操作时，程序除了执行指定的操作外，还会顺带将ht[0]哈希表在rehashidex索引上的所有键值对rehash到[1]，当rehash完成，程序将rehashidx属性的值增一。

4）随着字典操作的不断执行，最终在某个时间点上，ht[0]的所有键值对都会被rehash至ht[1]，这时程序将rehashidex属性的值设置为-1，表示rehash操作已完成。

渐进式Rehash的好处在于它采取分而治之的方式，将rehash键值对所需的计算工作均摊到了对字典的每个添加，删除，查找和更新操作上，从而避免了集中式rehash而带来的庞大计算量。



# Redis的性能
redis的高性能主要依赖于：
* 1.完全基于内存
* 2.数据结构简单
* 3.采用一个线程处理网络请求，不需要线程的切换
* 4.使用I/O多路复用


# Redis哨兵
哨兵模式-redis集群方案之一，它的功能：
* 1.监控redis实例
* 2.传递信息
* 3.自动故障迁移（主服务出问题的时候，切换一个从服务为主服务）

### 原理就是哨兵通过发送命令，等待服务响应，从而监控redis实例

哨兵模式的工作方式：
1】、每个Sentinel（哨兵）进程以一定的频率向整个集群中的Master主服务器，Slave从服务器以及其他Sentinel（哨兵）进程发送一个 PING 命令。

2】、如果一个实例（instance）距离最后一次有效回复 PING 命令的时间超过 down-after-milliseconds 选项所指定的值， 则这个实例会被 Sentinel（哨兵）进程标记为主观下线（SDOWN）。

3】、如果一个Master主服务器被标记为主观下线（SDOWN），则正在监视这个Master主服务器的所有 Sentinel（哨兵）进程要以每秒一次的频率确认Master主服务器的确进入了主观下线状态。

4】、当有足够数量的 Sentinel（哨兵）进程（大于等于配置文件指定的值）在指定的时间范围内确认Master主服务器进入了主观下线状态（SDOWN）， 则Master主服务器会被标记为客观下线（ODOWN）。

5】、在一般情况下， 每个 Sentinel（哨兵）进程会以每 10 秒一次的频率向集群中的所有Master主服务器、Slave从服务器发送 INFO 命令。

6】、当Master主服务器被 Sentinel（哨兵）进程标记为客观下线（ODOWN）时，Sentinel（哨兵）进程向下线的 Master主服务器的所有 Slave从服务器发送 INFO 命令的频率会从 10 秒一次改为每秒一次。

7】、若没有足够数量的 Sentinel（哨兵）进程同意 Master主服务器下线， Master主服务器的客观下线状态就会被移除。若 Master主服务器重新向 Sentinel（哨兵）进程发送 PING 命令返回有效回复，Master主服务器的主观下线状态就会被移除。


心跳检测的作用：
* 1.检测主服务器的网络连接状态
* 2.辅助实现min-slaves选项
* 3.检测命令丢失（主服务器给从服务器的写命令，根据偏移量来判定）


# Redis底层数据结构
SDS：简单动态字符串
<div align="center"> <img src="https://github.com/RJianPeng/Technology-Stack/blob/master/%E7%BC%93%E5%AD%98/photo/SDS.png"/></div><br>

预分配空间：len<1MB，预先分配len的空间，len>1MB，预先分配1MB的空间
不需要用二进制字符标识字符的结束

链表：
列表的底层实现之一
双向链表，头结点的前置和尾结点的后置都是null

字典：
用来保存键值对的抽象数据类型
redis底层数据库的实现
rehash的时候使用字典结构
<div align="center"> <img src="https://github.com/RJianPeng/Technology-Stack/blob/master/%E7%BC%93%E5%AD%98/photo/%E5%AD%97%E5%85%B8.png"/></div><br>

跳跃表：

跳跃表在 Redis 中，只有两个地方用到：一个是实现有序集合对象，另一个是在集群节点中用作内部数据结构。
<div align="center"> <img src="https://github.com/RJianPeng/Technology-Stack/blob/master/%E7%BC%93%E5%AD%98/photo/%E8%B7%B3%E8%B7%83%E8%A1%A8.png"/></div><br>


# Redis分布式锁
Redis分布式锁特性：
* 高性能(加、解锁时高性能)
* 可以使用阻塞锁与非阻塞锁。
* 不能出现死锁。
* 可用性(不能出现节点 down 掉后加锁失败)。

使用方式：
先拿setnx来争抢锁，抢到之后，再用expire给锁加一个过期时间防止锁忘记了释放。但是为了避免setnx抢占成功后因为某些原因不能执行expire导致死锁，我们可以利用 Redis set key 时的一个 NX 参数可以保证在这个 key 不存在的情况下写入成功。并且再加上 EX 参数可以让该 key 在超时之后自动删除。保证只有一个进程能得到锁且不会死锁（过期删除&原子操作）

加锁：
```
//这个地方是通过redis的set将setnx和expire合并成一个原子性的操作
String result = this.jedis.set(LOCK_PREFIX + key, LOCK_MSG, "NX", "PX", 10 * TIME);
if (LOCK_MSG.equals(result)){
  return true ;
}else {
  return false ;
}
```

加阻塞锁：
```
for (;;){
  String result = this.jedis.set(LOCK_PREFIX + key, LOCK_MSG, "NX", "PX", 10 * TIME);
  if (LOCK_MSG.equals(result)){
      break ;
  }

  //防止一直消耗 CPU 	
  Thread.sleep(DEFAULT_SLEEP_TIME) ;
}
```

解锁：不能直接删除key，需要先判定这个锁是否是自己的才能删除。


# Redis常见问题
* Q:假如Redis里面有1亿个key，其中有10w个key是以某个固定的已知的前缀开头的，如何将它们全部找出来？
* A:使用keys指令可以扫出指定模式的key列表
* Q:如果这个redis正在给线上的业务提供服务，那使用keys指令会有什么问题？
* A:keys指令会导致线程阻塞一段时间，线上服务会停顿，直到指令执行完毕，服务才能恢复。这个时候可以使用scan指令，scan指令可以无阻塞的提取出指定模式的key列表，但是会有一定的重复概率，在客户端做一次去重就可以了，但是整体所花费的时间会比直接用keys指令长。（scan指令是增量式迭代命令）


* Q:如何用redis做消息队列
* A:一般使用list结构作为队列，rpush生产消息，lpop消费消息。当lpop没有消息的时候，要适当sleep一会再重试，或者使用redis的blpop指令，这个指令若无元素则会阻塞。




# Redis的发布订阅
所谓发布订阅，就是消息发布者发布消息及消息订阅者接收消息，二者通过某种媒介关联起来。在Redis的发布订阅中，存在频道channel，客户端订阅了某channel之后，当发布者通过此channel发布消息时，所有订阅该channel的用户都会收到该消息。

当一个客户端通过 PUBLISH 命令向订阅者发送信息的时候，我们称这个客户端为发布者(publisher)。
而当一个客户端使用 SUBSCRIBE 或者 PSUBSCRIBE命令接收信息的时候，我们称这个客户端为订阅者(subscriber)。
为了解耦发布者(publisher)和订阅者(subscriber)之间的关系，Redis 使用了 channel (频道)作为两者的中介 —— 发布者将信息直接发布给 channel ，而 channel 负责将信息发送给适当的订阅者，发布者和订阅者之间没有相互关系，也不知道对方的存在。

Redis发布订阅的底层原理：
redis-server 里维护了一个字典，字典的键就是一个个 channel ，而字典的值则是一个链表，链表中保存了所有订阅这个 channel 的客户端。SUBSCRIBE 命令的关键，就是将客户端添加到给定 channel 的订阅链表中。
发布者通过 PUBLISH 命令向订阅者发送消息时，redis-server 会使用给定的频道作为键，在它所维护的 channel 字典中查找记录了订阅这个频道的所有客户端的链表，遍历这个链表，将消息发布给所有订阅者。

Redis发布订阅的应用场景：
* 1.异步消息通知
* 2.任务执行通知
* 3.数据刷新通知

（发布订阅这部分基本转载自大佬的博客https://juejin.im/post/5cf7bf4051882502f9490be8 ）
# pipeline
redis的get操作(不单单是get命令)是阻塞的，如果循环取值的话，就算是内网，耗时也是巨大的。

Pipeline：redis的管道命令，允许client将多个请求依次发给服务器（redis的客户端，如jedisCluster，lettuce等都实现了对pipeline的封装），过程中而不需要等待请求的回复，在最后再一并读取结果即可。

使用方式（单机版）：
```
//换成真实的redis实例
Jedis jedis = new Jedis();
//获取管道
Pipeline p = jedis.pipelined();
for (int i = 0; i < 100; i++) {
    p.get(i);
}
//获取结果
List<Object> results = p.syncAndReturnAll();
```

# cluster
redis cluster：redis官方的多机器部署方案。一组Redis Cluster是由多个Redis实例组成，官方推荐我们使用6实例，其中3个为主节点，3个为从结点。一旦有主节点发生故障的时候，Redis Cluster可以选举出对应的从结点成为新的主节点，继续对外服务，从而保证服务的高可用性。

### 客户端访问
不同主节点的数据都不同，那么客户端如何知道去哪个主节点访问数据？

Redis Cluster 把所有的数据划分为16384个不同的槽位，可以根据机器的性能把不同的槽位分配给不同的Redis实例，对于Redis实例来说，他们只会存储部门的Redis数据，当然，槽的数据是可以迁移的，不同的实例之间，可以通过一定的协议，进行数据迁移。首先客户端需要保存一份Redis Cluster槽相关的信息，也就是路由表，然后对即将访问的key进行哈希计算，计算出对应的槽位，然后向对应的Redis实例发起查询请求。如果访问的Redis实例中，的确保存着对应槽的数据信息，就会进行返回，否则向客户端返回一个Moved指令，让客户端到正确的地址进行获取。

<div align="center"> <img src="https://github.com/RJianPeng/Technology-Stack/blob/master/%E7%BC%93%E5%AD%98/photo/rediscluster.png"/></div><br>


# jedis

## set
* jedis.set(String key,String value,String nxxx,String expx,long time)
参数含义：
key:key值
value:value值
nxxx: NX|XX,NX-当该key不存在时才能set XX-当key存在时才能set
expx:EX|PX 键值对的过期时间单位，ex：秒  px：毫秒





