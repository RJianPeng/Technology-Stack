# 第三章 客户端API：基础知识
## 3.1 概述
用户可以通过org.apache.hadoop.hbase.client包中的HTable类提供的方法，向HBase存储、检索和删除数据。而创建HTable实例是有代价的。每个实例都需要扫描.META表并执行一些操作，这些操作非常耗时，因此推荐用户只创建一次HTable实例并复用。如果用户需要使用多个HTable实例，可以考虑使用HTablePool类。

* 写操作中涉及的列的数目不会影响该行的原子性，行原子性会同时保护到所有的列。

## 3.2 CRUD操作

### 3.2.1 put方法
#### 单行put
1.Put

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
Put构造函数中还有一个rowlock（行锁）参数，若要频繁的重复修改某些行，用户有必要创建一个RowLock实例来防止其他客户端访问这些行。

所有的构造函数都需要row，即行键。

创建Put实例之后就可以向该实例添加数据了：
```
Put add(byte[] family, byte[] qualifier, byte[] value)
Put add(byte[] family, byte[] qualifier, long ts, byte[] value)
Put add(KeyValue kv) throws IOException
```

2.KeyValue

KeyValue是表示一个单元格的实例。

获取Put中的KeyValue:
```
List<KeyValue> get(byte[] family, byte[] qualifier)
Map<byte[], List<KeyValue> getFamilyMap()
```
KeyValue是HBase在存储架构中最底层的类。

KeyValue中数据和左表都是用的byte[]，这样能存储任意类型的数据，保证了最少的内部数据结构开销。

KeyValue中提供了很多比较器，用于KeyValue的排序对比。

KeyValue实例还有一个变量，代表该实例的唯一坐标：类型（Put/Delete/DeleteColumn/DeleteFamily）

3.客户端的写缓冲区

HBase的API配备了一个客户端的写缓冲区，负责收集put操作，然后通过rpc一次性送往服务器。

以下是其方法
```
void setAutoFlush(boolean autoFlush) // 设为false则启动缓冲区
boolean isAutoFlush //检查标识的状态
void flushCommits() throws IOException //强制把数据写入服务端
```
不过用户没必要强制刷写缓冲区，因为数据一旦超出缓冲区指定的大小限制，客户端就会隐式的调用命令对缓冲区进行刷写。
```
(HTable)long getWriteBufferSize()
void setWriteBufferSize(long writeBufferSize) throws IOException //设置客户端缓冲区的大小 默认为2MB
```

4.Put列表
```
void put(List<Put> puts) throws IOException //批量put
```

客户端会对put进行检查，比如确认实例内容是否为空或是否指定了列，如果检查失败，出错的及后面的put都会被留在缓冲区。


5.原子性操作compare-and-set
检查写的方法：boolean checkAndPut(byte[] row,byte[] family, byte[] qualifier, byte[] value, Put put)

只能检查和修改同一行数据，只能保证同一行数据的原子性保证。

### 3.2.2 get方法
1.单行get
```
Result get(Get get) //单行get方法
Get(byte[] row) //Get的构造函数
Get(byte[] row,RowLock rowLock)
```
Get还可以通过设置多种参数筛选目标数据

2.Result类
Result提供的方法如下：
```
byte [] getRow(); //返回行键
byte[] getValue(byte[] family, byte[] qualifier); //返回单元格的值
byte[] value();//返回第一列的值
int size();
boolean isEmpty();
KeyValue[] raw();
List<KeyValue> list();
```

3.Get列表




