# Hbase



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






















