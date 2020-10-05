
# 概念
## kafka简介
（节选自https://www.jianshu.com/p/d3e963ff8b70）
kafka是一个分布式消息队列。具有高性能、持久化、多副本备份、横向扩展能力。生产者往队列里写消息，消费者从队列里取消息进行业务逻辑。一般在架构设计中起到解耦、削峰、异步处理的作用。

kafka对外使用topic的概念，生产者往topic里写消息，消费者从读消息。为了做到水平扩展，一个topic实际是由多个partition组成的，遇到瓶颈时，可以通过增加partition的数量来进行横向扩容。单个parition内是保证消息有序。

kafka的运行流程大概是：Producer往broker中的指定topic分片中写消息，然后Consumer异步的从broker中的topic中消费消息进行业务逻辑。

关于broker、topics、partitions的一些元信息用zk来存，监控和路由啥的也都会用到zk。

## 生产
kafka中消息的写入都是批量的，发往同一个partition的消息会积攒起来，由一个单独的线程一次性发送过去。

消息投递语义：
kafka支持3种消息投递语义
At most once：最多一次，消息可能会丢失，但不会重复
At least once：最少一次，消息不会丢失，可能会重复
Exactly once：只且一次，消息不丢失不重复，只且消费一次（0.11中实现，仅限于下游也是kafka）

在业务中，常常都是使用At least once的模型，如果需要可重入的话，往往是业务自己实现。


## 消费
订阅topic是以一个消费组来订阅的，一个消费组里面可以有多个消费者。同一个消费组中的两个消费者，不会同时消费一个partition。换句话来说，就是一个partition，只能被消费组里的一个消费者消费，但是可以同时被多个消费组消费。因此，如果消费组内的消费者如果比partition多的话，那么就会有个别消费者一直空闲。

## 文件组织
kafka的数据，实际上是以文件的形式存储在文件系统的。topic下有partition，partition下有segment，segment是实际的一个个文件，topic和partition都是抽象概念。

## 分区partition


# 小知识点
## 消费者开始消费的位置
消费者从头开始消费某个队列的消息需要满足两个条件：
* 1.使用一个全新的"group.id"（就是之前没有被任何消费者使用过）
* 2.指定"auto.offset.reset"参数的值为earliest，如果该值设置为latest，那么消费者不会消费之前的消息，而是从接入之后进来的新消息开始消费。

注意：从kafka-0.9版本及以后，kafka的消费者组和offset信息就不存zookeeper了，而是存到broker服务器上，所以，如果你为某个消费者指定了一个消费者组名称（group.id），那么，一旦这个消费者启动，这个消费者组名和它要消费的那个topic的offset信息就会被记录在broker服务器上。


## kafka中队列的分片数越多速度越快？
（这部分节选自：https://www.jianshu.com/p/cdfc3df9e4c6）
在kafka中，partition（分片）和队列的关系就像泳池和泳道，每个泳道都可以独立的传输消息。而kafka的吞吐量在资源充足的情况下的确是分片越多，速度越快。但是前提是资源充足即节点数量，这个很重要。

然而更多的分片意味着更高的开销：
* 1.越多的分区需要打开更多的文件句柄（在kafka的broker中，每个分区都会对照着文件系统的一个目录）
* 2.更多的分区会导致端对端的延迟（主要是副本的同步延迟，在一个broker上的副本从其他broker的leader上复制数据的时候只会开启一个线程）
* 3.越多的partition意味着需要更多的内存
* 4.越多的partition会导致更长时间的恢复期
通常情况下，越多的partition会带来越高的吞吐量，但是同时也会给broker节点带来相应的性能损耗和潜在风险，虽然这些影响很小，但不可忽略，因此需要根据自身broker节点的实际情况来设置partition的数量以及replica的数量。
