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

