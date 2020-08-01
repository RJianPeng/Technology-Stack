- [Hbase](#hbase)
  - [应用](#应用)
  - [操作指令](#操作指令)
- [Hive](#hive)
  - [概念](#概念)
  - [Hive中的复合结构](#hive中的复合结构)
  - [操作指令](#操作指令)
  - [操作指令使用进阶](#操作指令使用进阶)
  - [踩坑日记](#踩坑日记)

# Hbase

## 应用
### rowkey的设计原则
* 1.保持rowkey的唯一性，在插入过程中，若有重复的rowkey，后面的数据会覆盖前面的数据。
* 2.散列原则，设计的rowkey应该均匀的分布在hbase的节点上。假设rowkey根据时间来设计，那么新数据会大量聚集在同一个节点上，也就是常说的region热点问题。可能导致大量的client访问到同一个regionserver，导致该机器负载过高。




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




## Hive中的复合结构
### array
```
array(var1,var2,....) as arr//array的声明方式
arr[0] //array的元素调用方式
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



## 踩坑日记
### DAY1 collect_list/set和group by的羁绊
今天妄想使用collect_list强行拼接两个值到一个数组里面，然后疯狂报错select后面的值不在group by的子句中的错误，我看了下代码中根本没有写group by。查询资料后发现：collect_list/set必须和group by搭配使用，后改成array来拼接值到数组中，hql正常运行。














