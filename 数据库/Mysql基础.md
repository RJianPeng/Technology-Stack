* [Mysql架构](#mysql架构)
* [并发控制](#并发控制)
* [事务](#事务)
* [基础数据类型](#基础数据类型)
* [基础功能特性](#基础功能特性)
* [连接池](#连接池)
* [mysql进阶](#mysql进阶)

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

## mysql的for update
for update操作是在数据库中上锁用的，可以为数据库中的行上一个排它锁。当一个事务的操作未完成时候，其他事物可以读取但是不能写入或更新。

通常是在使用查询的时候使用，使用方式为：
```
select * from table_name where XXX for update;
```
InnoDB默认是行级别的锁，当有明确指定的主键时候，是行级锁（如果没找到数据不会加锁），否则是表级别的（这个时候不管找没找到数据都会是表级锁）。

for update 这种方式使用的时候需要注意：
* 1.仅仅适用于InnoDB，并且必须开启事务。
* 2.和for update nowait区别，for update阻塞其他事务，nowait直接拒绝其他事务。


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
* 时间范围为：'1970-01-01 00:00:01.000000' to '2038-01-19 03:14:07.999999'
* 最高可精确到微秒，类型生命为timestamp(N)。N表示精确程度，如需要精确到毫秒则设置为Timestamp(3)，如需要精确到微秒则设置为timestamp(6)，数据精度提高的代价是其内部存储空间的变大，但仍未改变时间戳类型的最小和最大取值范围（‘1970-01-01 00:00:01' UTC 至'2038-01-19 03:14:07’)。
* 插入记录时，时间戳字段包含DEFAULT CURRENT_TIMESTAMP，如插入记录时未指定具体时间数据则将该时间戳字段值设置为当前时间
* 更新记录时，时间戳字段包含ON UPDATE CURRENT_TIMESTAMP，如更新记录时未指定具体时间数据则将该时间戳字段值设置为当前时间
* 对应的jdbcType为TIMESTAMP 对应Java类型为TimeStamp/Date
* timestamp可能会引发的异常：当MySQL参数time_zone=system时，查询timestamp字段会调用系统时区做时区转换，而由于系统时区存在全局锁问题，在多并发大数据量访问时会导致线程上下文频繁切换，CPU使用率暴涨，系统响应变慢设置假死。


## DATETIME
* 占用八个字节
* 时间范围为‘1000-01-01 00:00:00’至‘9999-12-31 23:59:59’
* DATATIM不受时区影响，TIMESTAMP受时区影响（TIMESTAMP在存取的过程中会出现本地时区时间<->UTC时区时间<->int类型的转换流程）。
* 对应的jdbcType为DATETIME 对应Java类型为TimeStamp
* DATATIME也可以精确至毫秒，声明方式和TIMESTAMP相同

### TIMESTAMP和DATETIME的选择
* 如果服务器时区不一样就建议选择 datetime。

* 如果是想要使用自动插入时间或者自动更新时间功能的，可以使用timestamp，现在current_timestamp在高版本(5.6好像)中也可以给DATETIME使用。

* 如果只是想表示年、日期、时间的还可以使用 year、 date、 time，它们分别占据 1、3、3 字节，而datetime就是它们的集合。


## VARCHAR
varchar(1024) 这里的1024具体指代需要看mysql版本号，4.x版本是字节长度  5.x是字符长度
这个字段在底层存储中占用空间固定为1024
除了char和varchar，其他类型后面括号里的数字都不影响存储

# 基础功能特性
## on duplicate key update
插入数据时，如果插入的数据中的主键在数据库中已经存在，普通的insert的操作会报错，这个时候可以选用insert on duplicate key update
eg:
```
insert into table test(id,name) values(1,'github') on duplicate key update name = 'github'//ID为主键，如果数据库中已经存在ID为1的数据，那么这个时候会更新name字段为github
```
但是on duplicate key update 存在问题，即一个表不一定只有一个唯一索引，使用这种方式有可能会因为预想之外的原因产生更新而不是插入。


## explain的各个字段详解
* id 查询序号：即为sql语句执行的顺序,包含自查询的查询，explain的结果包含多行，以此ID区分执行顺序，ID越大越先执行。
* select type：查询类型，有以下几种值：SIMPLE，PRIMARY，UNION。SIMPLE表示简单的查询，没有union和子查询什么的。PRIMARY，在有子查询的sql中，最外层的查询语句就是primary类型。UNION，union语句中第二个或者是后面那个就是这种查询类型
* table：输出的数据所用的表
* type：连接类型，有多个参数。1.system，表仅有一行，很少出现这种情况。    2.const，表最多有一个匹配行，const用于比较primary key 或者unique索引。因为只匹配一行数据，所以很快。type参数从最佳到最差顺序：
null>system/const>eq_ref>ref>ref_or_null>index_merge>range>index>all。type=null：在优化过程中就已经得到结果，不用再访问表或者索引。 type=const/system：在整个查询过程中这个表最多只会有一条匹配的行,一定是泳道primary key 或者unique key的情况下才会是const。 
type=eq_ref：使用有唯一性索引查找（主键或唯一性索引）。 type=ref：非唯一性索引访问，它返回所有匹配某个单个值的行，它是查找和扫描的混合体。type=ref_or_null：和ref情况类似，只是返回结果中可能包含空行。以上这五种情况都是很理想的索引使用情况。
type=range：对索引进行范围扫描的情况。 type=index：该连接类型与ALL相同都是扫描表，但这种类型只扫描索引树，而ALL是对数据表文件的扫描
* possible_keys：提示可能使用哪个索引会在该表中找到行，不太重要
* keys：实际查询中使用的索引
* key_len：mysql使用的索引长度
* rows：查询时遍历的行数
* extra：包含查询mysql的详细信息。Not exists：不存在信息。 range checked for each record：没有找到合适的索引。 Using index/Using index condition：本次查询使用了覆盖索引，避免访问表的数据行，效率不错。
using temporary：mysql对查询结果进行排序对时候使用了一张临时表。using filesort：mysql对数据不是按照表内的索引顺序进行读取，而是使用了其他字段重新排序。using where：表示本次查询进行了后过滤，所谓后过滤，就是先读取整行数据，再检查此行是否符合where条件。

## 修改字段名称和属性
alter table {table_name} change old_column new_column …

## mysql的常用插入操作
### insert into
插入时，数据库会检查主键和唯一键，出现重复则报错

### replace into
插入时，数据库检查主键和唯一键，出现重复的情况，会delete老的重复数据，insert新的数据进去，多个唯一键的情况下，一行的写入可能导致多行的删除。这种插入方式，如果新的数据部分字段没有值，会丢失原有数据字段的值

### insert ignore
插入时，数据库检查主键和唯一键，出现重复的情况，则忽略这条新数据

## mysql中字符的拼接
### group_concat 
对分组后的列进行聚合拼接，分隔符默认为','
```
eg:select group_concat(column) from table group by other_column
```




# 连接池
## TOMCAT JDBC连接池
```
连接池参数：
initialSize=0 //初始化连接
maxActive=30 //连接池的最大数据库连接数，设为0表示无限制
maxIdle=20 //没有人用连接的时候，最大闲置的连接个数，设置为0时，表示没有限制。
maxWait=1000 //超时等待时间以毫秒为单位
removeAbandoned=true //是否自动回收超时连接
removeAbandonedTimeout=60 //设置被遗弃的连接的超时的时间（以秒数为单位），即当一个连接被遗弃的时间超过设置的时间，则它会自动转换成可利用的连接。默认的超时时间是300秒。
logAbandoned = true //是否在自动回收超时连接的时候打印连接的超时错误
validationQuery=select 1 from dual //给出一条简单的sql语句进行验证
testOnBorrow=true //在从连接池中取出连接时进行有效验证
testOnReturn=true //在将连接放回池子中时进行有效验证
testWhileIdle=true//连接池根据timeBetweenEvictionRunsMills参数，定时检查连接池中的连接是否有效。检查失效的
剔除，如果连接空闲时间超过minEvictableIdleTimeMills也会被剔除
validationInterval // 为了避免高QPS场景下探活的性能损耗，连接池会记录上一次连接探活生效的时间，下一次探活和上一次探活的
时间间隔如果小于validationInterval，那么会直接判定为生效连接
testWhileIdle=true//不检测busy连接
```
关于tomcat jdbc连接池的探活，一般有两种探活机制，定时探活机制和连接出/归池探活（都在上面配置里面了）。




# MYSQL进阶
## 分区表
mysql数据库在5.1之后支持分区，划分方式为水平分区，索引方面支持分区索引-即索引和数据都在分区中。
可通过命令查看数据库是否开启分区：
```
show global variables like '%partition%';
```

### 分区类型
（这部分内容转自：https://blog.csdn.net/wshl1234567/article/details/79072764）
* 1.RANGE分区
range分区：基于属于一个给定连续区间的列值，把多行分配给分区。

* 2.LIST分区
list分区：LIST分区和RANGE分区类似，区别在于LIST分区是基于列值匹配一个离散值集合中的某个值来进行选择，而非连续的

* 3.HASH分区
HASH分区的目的是将数据均匀地分布到预先定义的各个分区中，保证各分区的数据量大致都是一样的。在HASH分区中，MySQL自动完成解析行去哪个分区的工作，用户所要做的只是基于将要进行哈希分区的列指定一个列值或表达式，以及指定被分区的表将要被分隔成的分区数量。

* 4.COLUMNS分区
在前面说了RANGE、LIST、HASH和KEY这四种分区中，分区的条件是：数据必须为整形（interger），如果不是整形，那应该需要通过函数将其转化为整形，如YEAR()，TO_DAYS()，MONTH()等函数。MySQL5.5版本开始支持COLUMNS分区，可视为RANGE分区和LIST分区的一种进化。COLUMNS分区可以直接使用非整形的数据进行分区，分区根据类型直接比较而得，不需要转化为整形。此外，RANGE COLUMNS分区可以对多个列的值进行分区。

* 5.KEY分区
KEY分区和HASH分区相似，不同之处在于HASH分区使用用户定义的函数进行分区，支持字符串HASH分区，KEY分区使用MySQL数据库提供的函数进行分区，这些函数基于与PASSWORD()一样的运算法则。

### 子分区
子分区就是在分区的基础在再进行分区，也称为复合分区,mysql允许在range和list的分区基础上再进行hash或key的子分区。

### 分区和性能
数据库的应用一般分为两种，OLAP（在线事务分析）和OLTP（在线事务处理）。对于OLAP应用，分区的确可以提高系统的性能，因为OLAP会重复的扫描表里的数据，分区可以有效的减少应用需要扫描的数据量大小。但是对于OLTP应用来说，分区作用不大，因为通常不可能会获取一张大表10%的数据，更多的是根据索引获取到想要的几条数据。


### mysql的分区字段必须包含在主键字段里面

###

## mysql强制使用索引
force index（索引名）：在sql中声明强制使用哪条索引。在select *** 后面声明，用于提高查询速度。

## 对于mysql中key和index的理解
key是数据库的物理结构，包含两种含义：约束（偏重于约束和规范数据库的完整性），索引（辅助查询使用的）

index是数据库的物理结构，只是辅助查询的

简单来说mysql中的key和index在定义上不同，前者为约束，后者为索引，但是在使用场景上，使用效果相同，因为在使用其中一个的时候会把另一个属性也带上。




