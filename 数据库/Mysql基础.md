* [基础数据类型](#mysql基础数据类型)







# 基础数据类型

## DECIMAL
* 常用于保存准确精确度的值
* DECIMAL声明：decimal(M,N),M表示数字长度，N表示小数点后面有多少位。decimal(M),即默认为decimal（M，0）




## TIMESTAMP
* 是默认的时间戳类型。
* 占用四个字节。
* 最高可精确到微秒，类型生命为timestamp(N)。N表示精确程度，如需要精确到毫秒则设置为Timestamp(3)，如需要精确到微秒则设置为timestamp(6)，数据精度提高的代价是其内部存储空间的变大，但仍未改变时间戳类型的最小和最大取值范围（‘1970-01-01 00:00:01' UTC 至'2038-01-19 03:14:07’)。
* 插入记录时，时间戳字段包含DEFAULT CURRENT_TIMESTAMP，如插入记录时未指定具体时间数据则将该时间戳字段值设置为当前时间
* 更新记录时，时间戳字段包含ON UPDATE CURRENT_TIMESTAMP，如更新记录时未指定具体时间数据则将该时间戳字段值设置为当前时间
* 对应的jdbcType为TIMESTAMP
* timestamp可能会引发的异常：当MySQL参数time_zone=system时，查询timestamp字段会调用系统时区做时区转换，而由于系统时区存在全局锁问题，在多并发大数据量访问时会导致线程上下文频繁切换，CPU使用率暴涨，系统响应变慢设置假死。

## DATATIME
* 占用八个字节
* 时间范围为‘1000-01-01 00:00:00’至‘9999-12-31 23:59:59’
* DATATIM不受时区影响，TIMESTAMP受时区影响（TIMESTAMP在存取的过程中会出现本地时区时间<->UTC时区时间<->int类型的转换流程）。
* 对应的jdbcType为TIMESTAMP
* DATATIME也可以精确至毫秒，生命方式和TIMESTAMP相同

