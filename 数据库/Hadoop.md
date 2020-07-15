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

## UDF 用户自定义开发的函数
https://www.cnblogs.com/jifengblog/p/9278972.html


## 操作指令
### distribute by指令
distribute by 类似于partition by指令，对数据进行分区，可以结合sort by使用

### collect list/set
collect list/set，结合group by指令使用，将分组中的某一列转为数组/集合并返回

### lateral view
将一个结果集与一个数组/集合进行笛卡尔积
https://blog.csdn.net/guodong2k/article/details/79459282
### explode

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




























