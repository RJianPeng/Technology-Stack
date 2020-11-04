- [Hbase](#hbase)
  - [概念](#概念)
  - [应用](#应用)
  - [操作指令](#操作指令)
  - [小进阶](#小进阶)
- [Hive](#hive)
  - [概念](#概念)
  - [Hive中的复合结构](#hive中的复合结构)
  - [操作指令](#操作指令)
  - [操作指令使用进阶](#操作指令使用进阶)
  - [关于数据倾斜的故事](#关于数据倾斜的故事)
  - [踩坑日记](#踩坑日记)

- [YARN](#yarn)

- [Tez](#tez)
# Hbase

## 概念
### compaction
compaction类似与Java中的垃圾回收，用于合并文件，清除过期的多余版本的数据，以短期的I/O操作消耗换取较稳定的查询性能。

HBase中提供了两种compaction，minor和major
这两种compaction方式的区别是：
* Minor操作（小合并）只用来做部分文件的合并操作，不做任何删除数据、多版本数据的清理工作。
* Major操作（大合并）是对Region下的HStore下的所有StoreFile执行合并操作最后合并成一个HFile，且会对文件有删除操作，最终的结果是整理合并出一个文件。

为什么需要compaction
* HBase是基于一种LSM-Tree（Log-Structured Merge Tree）存储模型设计的，写入路径上是先写入WAL（Write-Ahead-Log）即预写日志，再写入memstore缓存，满足一定条件后执行flush操作将缓存数据刷写到磁盘，生成一个HFile数据文件。随着数据不断写入，磁盘HFile文件就会越来越多，文件太多会影响HBase查询性能，主要体现在查询数据的io次数增加。为了优化查询性能，HBase会合并小的HFile以减少文件数量，这种合并HFile的操作称为Compaction，这也是为什么要进行Compaction的主要原因。

（更多内容可见：https://blog.csdn.net/u011598442/article/details/90632702）

### TTL与MinVersion的关系
如果当前存储的所有时间版本都早于TTL，至少MIN_VERSION个最新版本会保留下来。这样确保在你的查询以及数据早于TTL时有结果返回。

### 数据写入流程
* 1.client连接zookeeper获取相关的regionserver的信息
* 2.向regionserver发出写请求，一次将数据写入MemoryStore和HLog，只有这两个都写入成功，此次写入才算成功。写入MemoryStore和HLog是原子性的，要么都成功要么都失败。
* 3.当写入MemoryStore的数据大小达到某个阈值之后，会讲MemoryStore flush成StoreFile文件，当StoreFile文件增长到一定数量之后，会触发Compact机制，将多个StoreFile合并成一个大的StoreFile文件，如果Region大到一定阈值，会将Region一分为二。

一个Region只存储一个Column Family的数据，或者一个Column Family的一部分。当Region达到一定大小之后，会根据Rowkey的排序，划分多个Region每一个Region又划分多个Store对象，每个Store对象，包括一个MemoryStore 和一个到多个StoreFile，MemoryStore是数据在内存中的实体，并且是有序的。StoreFile是HFile的一层封装。

HLog是WAL的一种实现，预写日志，用于保证数据不会丢失。每一个HBase的更新操作，都会先写入MemoryStore中，然后更新到HLog中，只有成功更新到HLog之后，这条数据才会更新成功。这样，就算MemoryStore中数据丢失，也可以通过HLog恢复。 注意，一般的WAL预写日志，是先写入日志，再写入内存的，而HBase是先写入内存，后写入日志，依托MVCC模式，确保数据不会丢失。

### 数据读取流程
* 1.跟Zookeeper简历连接，通过访问Meta Regionserver节点信息，HBase的meta表缓存到本地，获取要访问的表的Region的信息。
* 2.对该regionserver发起请求
* 3.Regionserver先扫描自己的Memorystore，如果没有找到，就会扫描BlockCache加速读内容的缓冲区，如果还没有找到，就会到StoreFile中查找这条数据，然后将这条数据返回给client

### 数据删除流程
不会立即删除，而是先在指定存储单元上标记删除，等到下一次region合并或者分裂的时候才会移除数据


### HBase隔离方案rsgroup
这个方案主要解决多个业务部分共享集群资源的场景，这一场景集群出现问题可能会影响到所有的业务部分，所以采用rsgroup的方案对同一集群上的多个业务进行隔离。
具体操作为：
* 1.将不同的regionserver分到不同的group
* 2.将不同的表分到不同的group

rsgroup缺点：隔离不彻底，hdfs层还是共用，如果datanode出现异常，还是会影响到多个业务。

### WAL解析
HBase的Write Ahead Log (WAL)提供了一种高并发、持久化的日志保存与回放机制。每一个业务数据的写入操作（PUT / DELETE）都会记账在WAL中。

WAL最重要的作用是灾难恢复。和MySQL 的BIN log类似，它记录所有的数据改动。一旦服务器崩溃，通过重放log，我们可以恢复崩溃之前的数据。这也意味如果写入WAL失败，整个操作将认为失败。

HLog是实现WAL的类。一个HRegionServer对应一个HLog实例，这样主要是如果一个region对应一个HLog，会需要并发写入大量的文件。HLog内部通过AtomicLong保证线程安全。

## 应用
### rowkey的设计原则
* 1.保持rowkey的唯一性，在插入过程中，若有重复的rowkey，后面的数据会覆盖前面的数据。
* 2.散列原则，设计的rowkey应该均匀的分布在hbase的节点上。假设rowkey根据时间来设计，那么新数据会大量聚集在同一个节点上，也就是常说的region热点问题。可能导致大量的client访问到同一个regionserver，导致该机器负载过高。

### hbase的预分区设计
HBase默认建表时有一个region，这个region的rowkey是没有边界的，即没有startkey和endkey，在数据写入时，所有数据都会写入这个默认的region，随着数据量的不断  增加，此region已经不能承受不断增长的数据量，会进行split，分成2个region。在此过程中，会产生两个问题：1.数据往一个region上写,会有写热点问题。2.region split会消耗宝贵的集群I/O资源。基于此我们可以控制在建表的时候，创建多个空region，并确定每个region的起始和终止rowky，这样只要我们的rowkey设计能均匀的命中各个region，就不会存在写热点问题。自然split的几率也会大大降低。当然随着数据量的不断增长，该split的还是要进行split。像这样预先创建hbase表分区的方式，称之为预分区。




## 操作指令

### get命令
用于获取hbase中某行的数据
get '表名'，'rowkey'

### scan命令 
扫描hbase全表的数据并返回
scan '表名' //扫描全表
scan '表名' {COLUMNS => ['base:weight','base:height']} //扫描全表并返回特定的列数据
scan '表名',{ROWPREFIXFILTER => 'c1'} //扫描以指定开头的rowkey的数据
scan '表名',{FILTER => "(QualifierFilter (<,'binary:name')) AND (QualifierFilter (=,'substring:jack'))"}//按列进行筛选
scan '表名',{FILTER=>"ColumnPrefixFilter('na') AND (ValueFilter(=,'substring:1') OR ValueFilter(=,'substring:3'))"}//以指定列的前缀查找数据 substring是参数中含有的值

## 小进阶


# Hive
https://www.docs4dev.com/docs/zh/apache-hive/3.1.1/reference/LanguageManual_SortBy.html hive中文手册
## 概念

### UDF 用户自定义开发的函数
https://www.cnblogs.com/jifengblog/p/9278972.html

### UDTF
UDTF：User-Defined Table-Generating Functions，用来解决输入一行输出多行的问题（explode这种）。

如何编写自己的UDTF函数：

实现org.apache.hadoop.hive.ql.udf.generic.GenericUDTF中的initialize,process,close三种方法。
其中initialize方法用于返回返回值的个数和列名，
process是真正处理数据的地方，在这个方法中每调用一次forward函数则产生一行数据，
close最后调用，对需要清理的方法进行清理。

如何使用UDTF函数：
* 1.直接在select后面使用
* 2.配合lateral view使用


以上整理自：
https://www.cnblogs.com/ggjucheng/archive/2013/02/01/2888819.html


### hive中内部表和外部表的区别与创建方法
Hive 创建内部表时，会将数据移动到数据仓库指向的路径；若创建外部表，仅记录数据所在的路径，
不对数据的位置做任何改变。在删除表的时候，内部表的元数据和数据会被一起删除，
而外部表只删除元数据，不删除数据。这样外部表相对来说更加安全些，数据组织也更加灵活，方便共享源数据。

需要注意的是传统数据库对表数据验证是 schema on write（写时模式），而 Hive 在load时是不检查数据是否
符合schema的，hive 遵循的是 schema on read（读时模式），只有在读的时候hive才检查、解析具体的
数据字段、schema。
读时模式的优势是load data 非常迅速，因为它不需要读取数据进行解析，仅仅进行文件的复制或者移动。
写时模式的优势是提升了查询性能，因为预先解析之后可以对列建立索引，并压缩，但这样也会花费要多的加载时间。

### hive中的分桶表
分桶和分区：分区提供了一个隔离数据和优化查询的便利的方式，但是分区的数量过多时会对NameNode造成比较大的压力。这种场景时可以考虑使用分桶来将数据集分解成容易管理若干部分。

分桶表的创建语句：
```
create table bucket_tab(id bigint comment '标示位',name string comment '名称') comment'分桶表' clustered by(id) into 4 buckets;
```

分区又分桶表的创建语句：
```
create table bucket_tab(id bigint comment '标志位',
                        name string comment '名称'
			)comment '分区分桶表'
			partitioned by(time string comment'时间')
			clustered by(id) sorted by(name) into 4 buckets;
```
分桶表使用注意：
* 1.set hive.enforce.bucketing = true；强制hive为目标表的分桶初始化过程设置一个正确的reducer个数，否则就需要我们自己来设置和分桶个数相匹配的reducer个数。例如 set mapred.reduce.tesks = 96

分桶的好处：
* 1.分桶加快了join查询的速度：在join的时候，如果左右两个表以相同的方式划分桶，这样处理左边表的某个桶时就能只取右边表中对应的那个桶，这种便采用了map-side join的方式，避免全表进行笛卡尔积的比对。

* 2.抽样操作更快：table_sample()方法


## Hive中的复合结构
### array
```
array(var1,var2,....) as arr//array的声明方式
arr[0] //array的元素调用方式
size(arr)//获取数组的长度
(ps:length(字符串)//获取字符串长度)
```

### map
```
map(key1,val1,key2,val2,...) as mapTest//map的声明方式
mapTest['key1'] //map中value的获取方式
```

### struct
```
struct(val1,val2,val3,...) as structTest //struct的声明方式
//struct有点像Java中的类的概念，使用时需要预先定义好属性名和类型，获取值时需要根据属性名获取
```



## 操作指令
### distribute by指令
distribute by 类似于partition by指令，对数据进行分区，可以结合sort by使用

### collect list/set
collect list/set，结合group by指令使用，将分组中的某一列转为数组/集合并返回

### lateral view 和 explode指令
explode（） extable as a:将一个数组或map转换成多行，即其中一个元素为一行

posexplode（） extable as a,b:功能和explode类似，但是将数组或map转换成多行的时候会保存其在数组或map中的序号，a即其序号从0开始，b即其元素

lateral view 将两个结果集进行笛卡尔积，经常喝explode搭配使用


https://blog.csdn.net/guodong2k/article/details/79459282

#### posexplode指令
该指令相对于explode()指令会多产生一个出参，该参数可以理解为该元素在原集合中的下表，且该参数会在序号参数的前面。
//TODO 下次遇到使用场景留个demo

### insert指令
insert into 在现有的数据基础上插入

insert overwrite将之前的数据删除再插入新的数据

### get_json_object
json的解析函数
```
SELECT get_json_object('{"a": "a", "b": "b"}', '$.a')
	, get_json_object('{"a": "a", "b":  "b"}', '$.b')
	, get_json_object('{"a": "a", "b":  "b"}', '$.c');
  /**
  *返回为"a","b",null
  */
```
### str_to_map 
字符串转为map格式，官方定义是：

str_to_map(text, delimiter1, delimiter2)；
Splits text into key-value pairs using two delimiters. Delimiter1 separates text into K-V pairs, and Delimiter2 splits each K-V pair. Default delimiters are ',' for delimiter1 and '=' for delimiter2.
将一个字符串转成map，delimiter1将字符串分为多个K-V对，默认为逗号（,）。delimiter2将每个K-V中的key和value分开，默认为等号（=）


### json_tuble
用来同时解析json字符串的多个字段
```
json_tuple(json_str,'key1', 'key2', 'key3') as value1,value2,value3 //同时解析json_str中的key为key1,key2,key3的值装载到value1,value2,value3中。
```


### unix_timestamp
unix_timestamp() 得到当前时间戳 只能精确到秒级
如果参数date满足yyyy-MM-dd HH:mm:ss形式，则可以直接unix_timestamp(string date) 得到参数对应的时间戳 
如果参数date不满足yyyy-MM-dd HH:mm:ss形式，则我们需要指定date的形式，在进行转换 

### left semi join
```
select a.id left a semi join b on a.id = b.id;
//这种left join左表遇到右表中多个符合条件的数据时，除了第一个会跳过后面的，即左表的数据只会返回一次。且最后产生的结果只允许出现左表中的数据
```

### 关于排序
#### order by
order by 全局排序，全局只使用一个reducer来进行排序的工作。

#### sort by
sort by 局部排序，排序工作分多个reducer来实现，所以不能保证全局有序，只能保证局部有序。

#### distribute by
一般搭配sort by使用，单独的功能为定义如何将map出来的数据分发到各个reducer上，搭配sort by可以指定分配规则并进行局部排序

#### cluster by
cluster by的功能就是distribute by 和sort by 相结合，有点区别的是cluster只能是按照降序来排序。

### 分组排序函数
#### row_number()
row_number() over(partition by a order by b asc) rank，这个方法即将数据根据a分组，然后在每组内根据b的升序对组内所有行对数据进行排序并在每行后面产生一个新列名为rank，rank的值不会重复。

#### rank()
rank() over(partition by a order by b asc) rank,这个方法和row_number()差不多，唯一的区别就是rank的值存在重复，但是序号会跳跃，比如1，2，2，4。两个2后面跟着的是4

#### dense_rank()
dense_rank() over(partition by a order by b asc) rank,这个方法和上面的两个都差不多，不同在于rank值存在重复但不会跳跃，比如1,2,2,3。两个2后面跟着的是3。

#### str_to_map()
```
str_to_map(text, delimiter1, delimiter2]) //使用两个分隔符将文本拆分为键值对。 Delimiter1将文本分成多个K-V对，Delimiter2分割每个K-V对。

select str_to_map('aaa:11&bbb:22', '&', ':');//结果为 aaa:11  bbb:22
```

### regexp及相关函数
* 1.regexp
```
A REGEXP B //匹配字符串A和正则表达式B
```

* 2.regexp_extract
```
 regexp_extract(string subject, string pattern, int index) //将字符串subject按照pattern正则表达式的规则拆分，返回index指定的字符
 eg:
 select regexp_extract('IloveYou','I(.*?)(You)',1) from test1 limit 1;  //返回love
```

* 3.regexp_replace
```
regexp_replace(string A, string B, string C) //将字符串A中的符合Java正则表达式B的部分替换为C
eg:
select regexp_replace("IloveYou","You","") from test1 limit 1;//返回Ilove
```
### 开窗函数
普通的聚合函数聚合的行集是组,开窗函数聚合的行集是窗口。因此,普通的聚合函数每组(Group by)只返回一个值，而开窗函数则可为窗口中的每行都返回一个值。简单理解，就是对查询的结果多出一列，这一列可以是聚合值，也可以是排序值，同时不会影响其他的基础字段。



## 操作指令使用进阶
### 使用union横向连接查询结果
原查询语句
```
select ep.productid,productname,count(st.tduserid),count(distinct sl.tduserid),count(distinct sn.tduserid),avg(sl.interval_level)
from(select productid,productname from xxx.product where productid = '3006090') ep
join(select tduserid,productid from xxx_page_ex where l_date <= '2019-04-07' and l_date >= date_add('2019-04-07', -6)) st
on ep.productid=st.productid
join(select tduserid,interval_level,productid from xxx_launch_ex where l_date <= '2019-04-07' and l_date >= date_add('2019-04-07', -6)) sl
on st.productid=sl.productid
join(select tduserid,productid from xxx_newuser_ex where l_date <= '2019-04-07' and l_date >= date_add('2019-04-07', -6)) sn
on sl.productid=sn.productid
group by ep.productid,productname;
```
改用union后的hql语句
```
select '2019-04-07' dates,
       '3006090' productid,
       max(pro) productname,
       sum(pv) pv,
       sum(uv) uv,
       cast(sum(duration) as decimal(10,4)) duration,
       sum(new_uv) new_uv
from 
(select productname pro,
       '0' pv,
       '0' uv,
       '0' duration,
       '0' new_uv
       from xxx.product where productid = '3006090'
union all
select '0' pro,
       count(tduserid) pv,
       '0' uv,
       '0' duration,
       '0' new_uv
       from xxx_page_ex where l_date <= '2019-04-07' and l_date >= date_add('2019-04-07', -6) and
       productid = '3006090'
union all
select '0' pro,
       '0' pv,
       count(distinct tduserid) uv,
       avg(interval_level) duration,
       '0' new_uv
       from xxx_launch_ex where l_date <= '2019-04-07' and l_date >= date_add('2019-04-07', -6) and
       productid = '3006090'
union all
select '0' pro,
       '0' pv,
       '0' uv,
       '0' duration,
       count(distinct tduserid) new_uv
       from xxx_newuser_ex where l_date <= '2019-04-07' and l_date >= date_add('2019-04-07', -6) and
       productid = '3006090'
) t;
```


### join的三种优化方式
* 1.在map端产生join
当链接的两个表是一个比较小的表和一个特别大的表的时候，我们把比较小的table直接放到内存中去，然后再对比较大的表格进行map操作。join就发生在map操作的时候，每当扫描一个大的table中的数据，就要去去查看小表的数据，哪条与之相符，继而进行连接。这里的join并不会涉及reduce操作。map端join的优势就是在于没有shuffle。
在实际开发过程中的使用方式：
```
set hive.auto.convert.join=true;
```

* 2.common join
common join也叫做shuffle join，reduce join操作。这种情况下生再两个table的大小相当，但是又不是很大的情况下使用的。具体流程就是在map端进行数据的切分，一个block对应一个map操作，然后进行shuffle操作，把对应的block shuffle到reduce端去，再逐个进行联合

* 3.SMBJoin
smb是sort  merge bucket操作，首先进行排序，继而合并，然后放到所对应的bucket中去，bucket是hive中和分区表类似的技术，就是按照key进行hash，相同的hash值都放到相同的bucket中去。然后对同一个bucket中的元素进行join操作。

## 关于数据倾斜的故事
关于资源倾斜的问题，多的不提，直入主题。
### 大表和小表join产生的数据倾斜
使用hive的mapJoin，在map阶段完成表的join工作，如果mapJoin启动成功，运行过程只会看到map而没有join对应的reducer。
mapJoin的启用方式：
set hive.auto.convert.join=true(默认是false) //hive是否根据表的大小，选择将common join转化为map join
set hive.mapjoin.smalltable.filesize=25000000(默认值25M) //设置小表大小上限，即大表小表判断的阈值
set hive.auto.convert.join.noconditionaltask = true; //是否将多个mapjoin转化为一个mapjoin 主要针对多个小表join大表的问题
set hive.auto.convert.join.noconditionaltask.size =10000000; //上面那个参数启动时这个才有用，表示将多个mapjoin转化为一个mapjoin时，其小表总和的最大值限制

mapJoin是解决小表join大表时产生数据倾斜问题的最好方式，所谓mapJoin就是在map阶段完成join工作，将所有的小表全量复制到每个map任务节点，然后再将小表缓存在每个map节点的内存里与大表进行join工作，普通的commonJoin则是将数据全部取出，再分发到各个reduce节点进行join工作。大表没有数据分布就不会产生数据倾斜。

对于小表join大表的数据倾斜问题，还可以通过规避的方式进行解决：具体比如给 Join的两个表都增加一列Join key，原理很简单：将小表扩充一列join key，并将小表的总数复制数倍，join key 各不相同，比如第一次为1，复制一次joinkey为2，依次类推；将大表扩充一列join key 为随机数，这个随机数为小表里的joinkey的随机值，如1-5的随机值。这样就实现了将一个大表拆分几分同时处理，而且这样小表扩充了几倍，大表就被对应地分成几份处理。这种方式也可以提高笛卡尔乘积小表join大表的性能。

### 大表和大表join产生的数据倾斜
情况1 大表与大表关联时，其中一张表关联的key有较多的空值，这部分数据容易倾斜到一个reduce上，可以在自查询中先过滤这部分数据，再进行关联。或者是给这些异常的值赋一个随机的值来分散它们。

情况2 当key值都是有效值时，解决办法为设置以下几个参数
set hive.exec.reducers.bytes.per.reducer = 1000000000 //每个节点的reduce默认处理1G大小的数据
set hive.optimize.skewjoin = true; //对倾斜的数据使用skew join，就是将倾斜的数据先写入hdfs，然后启动一轮mapjoin专门做部分特殊值的计算。
set hive.skewjoin.key = 100000 //超过这个数量的就是特殊值，使用skew join

除了skew join 还可以考虑SMB Join解决大表join大表时产生的数据倾斜问题
### group by造成的数据倾斜
group by的时候，可能出现某种类型的数据量特别多，而其他类型数据的数据量特别少，就会造成数据倾斜。解决方式：
set hive.map.aggr = true ;//这个配置项代表是否在map端进行耦合
set hive.groupby.skewindata=true;//这个配置项为true的时候，生成的查询计划会有两个MR Job，第一个MR Job会将输入随机分不到Reduce中，这样处理的结果是相同的Group By Key有可能被分发到不同的Reduce中，从而达到负载均衡的目的。 需要注意的是，这个为true时，hive不支持多列上的去重操作。

### count(distinct)以及其他参数不当造成的数据倾斜
情况1 设置的reduce个数太少
set mapred.reduce.tasks=800;//设置reducer的个数。默认是设置hive.exec.reducers.bytes.per.reducer这个参数，设置了后hive会自动计算reduce的个数，因此两个参数一般不同时使用。

情况2 当hql中包含count(distinct)时
使用sum...group by替代

### hive中的数据抽样方法
#### tablesample()函数
```
create table xxx_new as select * from xxx tablesample(10 percent) //从原表中抽取百分之十的数据放入临时表中

tablesample(n M) //指定抽样数据的大小，单位为M。

tablesample(n rows) //指定抽样数据的行数，其中n代表每个map任务均取n行数据，map数量可通过hive表的简单查询语句确认

select * from table_01 tablesample(bucket 1 out of 10 on rand())//通过随机的方式将数据分放到十个桶中，并抽取第一个桶中的数据
```


## 踩坑日记
### DAY1 collect_list/set和group by的羁绊
今天妄想使用collect_list强行拼接两个值到一个数组里面，然后疯狂报错select后面的值不在group by的子句中的错误，我看了下代码中根本没有写group by。查询资料后发现：collect_list/set必须和group by搭配使用，后改成array来拼接值到数组中，hql正常运行。

### DAY2 我没踩但我觉得可以记一下之hive的insert
hive的insert后面变量名与表中的变量名可以不同，只要顺序是一样的就OK。就酱。




# YARN
## 简介
Yarn是Hadoop集群的资源管理系统。它主要包括两部分功能：
* 1. ResourceManagement 资源管理
* 2. JobScheduling/JobMonitoring 任务调度监控

Yarn的另一个目标就是拓展Hadoop，使得它不仅仅可以支持MapReduce计算，还能很方便的管理诸如Hive、Hbase、Pig、Spark/Shark等应用。这种新的架构设计能够使得各种类型的应用运行在Hadoop上面，并通过Yarn从系统层面进行统一的管理，也就是说，有了Yarn，各种应用就可以互不干扰的运行在同一个Hadoop系统中，共享整个集群资源。

（详解可见https://blog.csdn.net/suifeng3051/article/details/49486927）





