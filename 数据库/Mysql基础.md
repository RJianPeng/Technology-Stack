* [Mysql架构](#mysql架构)
* [并发控制](#并发控制)
* [基础数据类型](#基础数据类型)
* [事务](#事务)

# Mysql架构
Mysql架构图如下：
<div align="center"> <img src="https://github.com/RJianPeng/Technology-Stack/blob/master/%E6%95%B0%E6%8D%AE%E5%BA%93/photo/MYSQL%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%80%BB%E8%BE%91%E6%9E%B6%E6%9E%84%E5%9B%BE.png"/></div><br>

## 连接器
每个客户端连接都会在服务器进程中拥有一个线程，这个连接的查询只会在这个单独的线程中执行。服务器会负责缓存线程，因此不需要为每一个新建的连接创建或者销毁线程。连接器负责跟客户端建立连接、获取权限、维持和管理连接。连接方式是TCP。

Q：如何查看服务器的空闲连接数？
A：show processlist 状态为sleep的连接都为空闲连接。客户端长时间未响应会断开连接，超市时间设置为wait_timeout。想要继续操作需要重新连接。
Q：除了重连还有其他方法吗？
A：使用长连接。但是长连接会导致内存使用的很快。Mysql在执行过程中临时使用的内存是管理在连接对象里面的。只有连接断开才会释放。因此可能导致OOM问题。可以选择执行mysql_reset_connection解决，这个指令会初始化连接中的资源，释放部分内存。

## 查询缓存
Mysql执行查询的时候会先去看看之前是否执行过同样的查询操作，如果存在直接返回。缓存中以key-value的格式存储查询操作和返回值。
缓存的失效非常容易，对表的更新都会导致对这个表查询的缓存全部清空，导致缓存命中率非常低。可以将query_cache_type设置为DEMAND关闭缓存，设置为SQL_CACHE使用缓存。（Mysql8.0后取消了缓存）

## 解析器&优化器
Mysql会解析查询，并创建内部数据结构（解析树），然后对其进行各种优化，包括重写查询、决定表的读取顺序以及选择合适的索引。我们可以通过特殊的关键字提示优化器，影响它的优化。也可以用explain请求展示优化过程。


# 并发控制
## 读写锁

## 锁粒度
### 表锁
### 行级锁


# 事务
事务：一组原子性的SQL查询

## 事务的ACID
### 原子性（atomicity）
一个事务为一个不可分割的最小工作单元，所有操作要么全部成功要么全部失败回滚。

### 一致性（consistency）
数据库总是从一个一致性的状态切换到另一个一致性的状态。

### 隔离性（isolation）
通常来说，一个事务所做的修改在最终提交以前，对其他事务是不可见的。

隔离性包含四种隔离级别，每一种级别都规定了一个事务中所做的修改，哪些在事务内和事务间是可见的，较低级别对隔离通常可以执行更高的并发，系统对开销也更低。
* 1.未提交读（READ UNCOMMITTED）
* 2.提交读（READ COMMITTED）
* 3.可重复读（REPEATABLE READ）
* 4.可串性化（SERIALIZABLE）

### 持久性（durability）
一个事务所做的修改在提交之后，其所做的修改就会永久保存到数据库中。






# 基础数据类型

## DECIMAL
* 常用于保存准确精确度的值
* DECIMAL声明：decimal(M,N),M表示数字长度，N表示小数点后面有多少位。decimal(M),即默认为decimal（M，0）
* 对应jdbcType为DECIMAL 对应Java类型为BigDecimal




## TIMESTAMP
* 是默认的时间戳类型。
* 占用四个字节。
* 最高可精确到微秒，类型生命为timestamp(N)。N表示精确程度，如需要精确到毫秒则设置为Timestamp(3)，如需要精确到微秒则设置为timestamp(6)，数据精度提高的代价是其内部存储空间的变大，但仍未改变时间戳类型的最小和最大取值范围（‘1970-01-01 00:00:01' UTC 至'2038-01-19 03:14:07’)。
* 插入记录时，时间戳字段包含DEFAULT CURRENT_TIMESTAMP，如插入记录时未指定具体时间数据则将该时间戳字段值设置为当前时间
* 更新记录时，时间戳字段包含ON UPDATE CURRENT_TIMESTAMP，如更新记录时未指定具体时间数据则将该时间戳字段值设置为当前时间
* 对应的jdbcType为TIMESTAMP 对应Java类型为TimeStamp
* timestamp可能会引发的异常：当MySQL参数time_zone=system时，查询timestamp字段会调用系统时区做时区转换，而由于系统时区存在全局锁问题，在多并发大数据量访问时会导致线程上下文频繁切换，CPU使用率暴涨，系统响应变慢设置假死。

## DATATIME
* 占用八个字节
* 时间范围为‘1000-01-01 00:00:00’至‘9999-12-31 23:59:59’
* DATATIM不受时区影响，TIMESTAMP受时区影响（TIMESTAMP在存取的过程中会出现本地时区时间<->UTC时区时间<->int类型的转换流程）。
* 对应的jdbcType为DATETIME 对应Java类型为TimeStamp
* DATATIME也可以精确至毫秒，生命方式和TIMESTAMP相同

