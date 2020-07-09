# 第三章 客户端API：基础知识
## 3.1 概述
用户可以通过org.apache.hadoop.hbase.client包中的HTable类提供的方法，向HBase存储、检索和删除数据。而创建HTable实例是有代价的。每个实例都需要扫描.META表并执行一些操作，这些操作非常耗时，因此推荐用户只创建一次HTable实例并复用。如果用户需要使用多个HTable实例，可以考虑使用HTablePool类。

* 写操作中涉及的列的数目不会影响该行的原子性，行原子性会同时保护到所有的列。

## 3.2 CRUD操作

### 3.2.1 put方法
#### 单行put
向HBase中存储数据：
```
void put(Put put) throws IOException
```

Put对象的构造函数：
```
Put(byte[] row)
Put(byte[] row, RowLock rowLock)
Put(byte[] row, long ts)
Put(byte[] row, long ts, RowLock,rowLock)
```
所有的构造函数都需要row，即行键。

创建Put实例之后就可以向该实例添加数据了：
```
Put add(byte[] family, byte[] qualifier, byte[] value)
Put add(byte[] family, byte[] qualifier, long ts, byte[] value)
Put add(KeyValue kv) throws IOException
```

KeyValue是表示一个单元格的实例。

获取Put中的KeyValue:
```
List<KeyValue> get(byte[] family, byte[] qualifier)
Map<byte[], List<KeyValue> getFamilyMap()
```
KeyValue是HBase在存储架构中最底层的类。




