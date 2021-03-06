* -[第一章、Netty——异步和事件驱动](#第一章Netty异步和事件驱动)
* -[第二章、你的第一款Netty应用程序](#第二章你的第一款Netty应用程序)
* -[第三章、Netty的组件和设计](#第三章Netty的组件和设计)
* -[第四章、传输](#第四章传输)
* -[第五章、ByteBuf](#第五章ByteBuf)
* -[第六章、ChannelHandler和ChannelPipeline](#第六章ChannelHandler和ChannelPipeline)

# 第一章、Netty——异步和事件驱动

## Java网络编程方案

### 1.原始的套接字API
用serverSocket的阻塞方法accept()使得服务端阻塞等待客户端的连接。这种方案一个线程只能处理一个客户端的连接，这就造成了 1.在任何时候都可能有大量的线程处于休眠状态，只是等待输入或输出数据就绪。 
2.虚拟机里面需要为每个线程的调用栈分配内存 3.在线程上下文的切换时也会带来很大的开销。

### 2.NIO
Java的套接字API也提供了非阻塞调用，底层用的是操作系统的事件通知API，也称为多路复用，即select(),poll(),epoll()这些方法。

Java中的java.nio.channels.Selector是Java的非阻塞I/O实现的关键。它使用了事件通知API以确定在一组非阻塞套接字中有哪些已经就绪能够进行I/O相关的操作，所以可以用一个单一的线程便可以处理多个
并发的连接。

这种模型提供了更好的资源管理： 1.使用较少的线程便可以处理许多连接，因此也减少了内存管理和上下文切换的成本。 2.当没有I/O操作需要处理的时候，线程也可以被用于其他任务。


### Netty的特性
<div align="center"> <img src="https://github.com/RJianPeng/Technology-Stack/blob/master/%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/photo/Netty-feature.jpeg"/></div><br>


异步：异步事件也可以具有某种有序的关系。通常，你只有在已经问了一个问题之后才会得到一个和它对应的答案，而在你等待它的同时你也可以做点别的事情。

异步和可伸缩性的联系：
* 1.非阻塞网络调用使得我们可以不必等待一个操作的完成。完全异步的I/O正式基于这种特性构建的，而且更进一步，异步方法会立即返回，并且在完成时，会直接或稍后某个事件通知用户。
* 2.选择器使得我们能够通过较少的线程便可监视许多连接上的事件。

## Netty的核心组件
#### Channel
是Java NIO的一个基本构造。可以看作是传入或者传出数据的载体，因此它可以被打开或者被关闭，连接或者断开连接

#### 回调
一个方法，一个提供给另一个方法使用的方法的引用，前者可以在适当的时候调用后者。

#### Future
提供了另一种在操作完成时通知应用程序的方式。这个结果可以看作是一个异步操作的结果的占位符；它将在未来的某个时刻完成，并提供对其结果的访问。JDK预设了interface，java.util.concurrent.Future，其所提供的实现只允许手动检查对应的操作是否完成或者一直阻塞到它完成。（可见https://github.com/RJianPeng/Technology-Stack/blob/master/%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/Java8%E5%AE%9E%E6%88%98.md#%E7%AC%AC%E5%8D%81%E4%B8%80%E7%AB%A0completablefuture%E7%BB%84%E5%90%88%E5%BC%8F%E5%BC%82%E6%AD%A5%E7%BC%96%E7%A8%8B）

所以Netty提供了自己的实现ChannelFuture，这个实现提供了几种方法，让我们可以注册监听器而不用手动检查是否完成。

#### 事件和ChannelHandler
Netty使用不同的事件来通知我们状态的改变或者是操作的状态，这使得我们能够基于已经发生的事件来触发适当的动作。每个事件都可以分发给ChannelHandler类中的某个用户实现的方法。这是一个很好的将事件驱动范式直接转换为应用程序构件块的例子。

ChannelHandler，是一个接口族的父接口，它的实现负责接收并响应事件通知。

# 第二章、你的第一款Netty应用程序
每一个Chennel都拥有一个与之相关联的ChannelPipeline，其持有一个ChannelHandler的实例链。

@Shareable，表示多个Channel可以共享这个ChannelHandler

### Netty服务器
所有的Netty服务器都需要以下两个部分：
* 1.至少一个ChannelHandler——该组件实现了服务器对从客户端接受的数据的处理，即它的业务逻辑。
* 2.引导-配置服务器的启动代码。

# 第三章、Netty的组件和设计
### Channel、EventLoop和ChannelFuture
channel——socket；
EventLoop——控制流、多线程处理、并发；
ChannelFuture——异步通知

#### Channel
基本的I/O操作（bind(),connect(),read(),write())依赖于底层网络传输提供的原语。

#### EventLoop
EventLoop定义了Netty的核心抽象，用于处理连接的生命周期中所发生的事件。

* 一个EventLoopGroup包含一个或多个EventLoop
* 一个EventLoop在他的生命周期只和一个Thread绑定
* 所有由EventLoop处理的I/O事件都将在它专有的Thread上被处理。
* 一个channel在它的生命周期内只注册于一个EventLoop
* 一个EventLoop可能会被分配给一个或多个Channel

<div align="center"> <img src="https://github.com/RJianPeng/Technology-Stack/blob/master/%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/photo/Channel%26EventLoop%26Channel.jpeg"/></div><br>

### ChannelHandler和ChannelPipeline

#### ChannelHandler
充当了处理入站和出站数据的应用程序逻辑的容器。它的方法是由网络事件触发的。
* ChannelInboundHandler：接收入站事件和数据

#### ChannelPipeline
##### 为ChannelHandler链提供了容器，并定义了用于在该链上传播入站和出站事件流的API。
ChannelHandler安装到ChannelPipeline的过程如下：
* 1.一个ChannelInitializer的实现被注册到了ServerVootstrap中
* 2.当ChannelInitializer.initChannel()方法被调用时，该方法中ChannelInitializer将在ChannelPipeline中注册一组自定义的ChannelHandler
* 3.ChannelInitializer将它自己从ChannelPipeline中移除，可以理解为ChannelInitializer为ChannelPipeline中的第一个

Netty能够确保数据只会在具有相同定向类型的两个ChannelHandler之间传递（channelInboundHandler和ChannelOutboundHandler为不同的），当ChannelHandler被添加到ChannelPipeline时，它将被分配一个ChannelHandlerContext，其代表了ChannelHandler和ChannelPipeline之间的绑定，这个对象主要用于写出站数据。

在Netty中，有两种发消息的方式：直接写到Channel中；也可以写到ChannelHandlerContext中。前一种方式会导致消息从ChannelPipeline的尾端开始流动，后者将导致消息从ChannelPipeline的下一个ChannelHandler开始流动。

//TODO 需要demo来看下实际情况

##### 为什么需要ChannelHandler的适配器
因为这些适配器提供了定义在对应接口中的所有方法的默认实现，所以在开发过程中我们只需要重写部分需要的方法就可以得到一个ChannelHandler了。

#### 引导
即引导客户端进行链接建立或引导服务端监听连接建立的Bootstrap（客户端）和ServerBootstrap（服务端）

服务端会有两个EventLoopGroup，第一个负责为传入连接请求创建Channel，第二个会为该Channel分配一个EventLoop


# 第四章、传输
传输的核心是Channel接口，它被用于所有的I/O操作。每个Channel都会被分配一个Channelpipeline和ChannelConfig。

ChannelConfig包含了该Channel的所有配置设置，并且支持热更新。

NIO提供了一个所有I/O操作的全异步实现，它利用了子jdk的基于选择器的API。选择器背后的概念是充当一个注册表，在那里你可以在Channel的状态发生变化时得到通知。选择器运行在一个检查状态变化，并对其做出相应响应的线程上，在应用程序对状态的改变做出响应之后，选择器将会被重置，并将重复这个过程。 

<div align="center"> <img src="https://github.com/RJianPeng/Technology-Stack/blob/master/%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/photo/selector.jpeg"/></div><br>

# 第五章、ByteBuf
Netty的ByteBuf用于替代JDK的ByteBuffer，是Netty的数据容器。

##### ByteBuf的优点
* 1.可以被用户自定义的缓冲区类型扩展
* 2.通过内置的复合缓冲区实现了透明的零拷贝（即不用从内核空间拷贝到用户空间）
* 3.容量可以按需增长（动态增长）
* 4.读写模式不用切换（ByteBuffer的flip方法）
* 5.读和写使用了不同的索引（读写时字符的起始位置）
* 6.支持方法的链式调用 ？
* 7.支持引用计数 
* 8.支持池化 

##### ByteBuf的工作方式
维护两个不同的索引，一个用于读取，一个用于写入。写入的时候writeIndex会递增，读取的时候readIndex会递增追赶writeIndex，当他们俩相等时，则说明ByteBuf中已无未读取的内容。
ByteBuf就是由两个索引分别控制读和写位置的字节数组。这两个索引的推进需要调用readerIndex（index）或writeIndex（index） TODO：这个地方需要demo看看实际推进情况 顺便看看ByteBuf空间整理的底层代码

###### ByteBuf的使用模式
* 1.堆缓冲区：将数据存储在堆空间中，这种方式能够提供快速的分配和释放。

* 2.直接缓冲区：直接缓冲区的内容会被驻留在常规的会被垃圾回收的堆空间之外。

* 3.复合缓冲区：提供了一个将多个缓冲区合并成单个缓冲区的虚拟表示。CompositeByteBuf

#### 派生缓冲区
* duplicate()
* slice()
* slice(int,int)
* Unpooled.unmodifiableBuffer()
* order(ByteOrder)
* readSlice(int)
这些方法都将返回一个新的ByteBuf实例，它具有自己的读索引、写索引、和标记索引，返回的ByteBuf和源实例是共享的。

ByteBuf有两种类别的读写操作：
* 1.get()和set()方法，从给定的索引开始，而且保持索引不变。
* 2.read()和write()方法，从给定的索引开始，并且会根据已经访问过的字节数对索引进行调整。


### ByteBuf分配
#### 按需分配：ByteBufAllocator接口
为了降低分配和释放内存的开销，Netty通过interface ByteBufAllocator实现了ByteBuf的池化（应该是类似于string常量池的东西），它可以用来分配我们所描述过的任意类型的ByteBuf实例。可以通过CHannel或者ChannelHandlerContext的alloc()方法获取一个ByteBufAllocator的引用。

Netty有两种ByteBufAllocator的实现：PooledByteBufAllocator和UnpooledByteBufAllocator，前者支持池化，后者不支持池化（这一点也和string的常量池很像）

#### 引用计数
Netty还实现了ByteBuf和ByteBufHolder的引用计数，它们都实现了ReferenceCounted接口。当引用计数为0时，即释放该实例资源。对池化来说很重要，降低了开销。

# 第六章、ChannelHandler和ChannelPipeline
Channel的四个状态：
* 1.ChannelUnregistered：Channel已经创建，但还未注册到EventLoop
* 2.ChannelRegistered：Channel已经被注册到了EventLoop
* 3.ChannelActive：Channel处于活动状态，现在可以接收和发送数据了
* 4.ChannelInactive：Channel没有连接到远程节点

这些状态改变的时候，会生成对应的事件，并调用ChannelInboundHandler对应的回调方法。

状态周期流转：2->3->4->1（校验方式，继承并实现一个ChannelInboundHandler的channelActive，channelRead，channelRegistered和channelUnregistered方法，通过日志查看调用顺序）

ChannelHandler的生命周期方法：
* 1.handlerAdded 把ChannelHandler添加到ChannelPipeline中时被调用
* 2.handlerRemoved 把ChannelHandler从ChannelPipelin中移除时调用
* 3.exceptionCaught 处理过程中出现错误调用

Netty有两个重要的ChannelHandler子接口：
* 1.ChannelInboundHandler 处理入站数据及各种状态变化

* 2.ChannelOutboundHandler 处理出站数据并且允许拦截所有的操作。它的方法将被CHannel、CHannelPipeline以及ChannelHandlerContext调用。它的大部分方法中都有个ChannelPromise参数，以便在操作完成时通过调用它的setSuccess和setFailure方法从而使ChannelFuture不可变。

Netty提供了class ResourceLeakDetector帮助你诊断潜在的资源泄漏问题。

#### ChannelHandlerContext接口
这个接口的主要功能是管理它所关联的ChannelHandler和在同一个ChannelPipeline中的其他ChannelHandler之间的交互。

有ChannelHandler加入到ChannelPipeline中时就会创建ChannelHandlerContext，ChannelHandlerContext和ChannelHandler的绑定关系时不变的。

### ChannelHandlerContext
代表了ChannelHandler和ChannelPipeline的关联，每当有ChannelHandler添加到ChannelPipeline中时，都会创建ChannelHandlerContext。

Channel和ChannelHandler中也有些Context的相同方法，调用Context的方法是从当前ChannelHandler开始传播，调用Channel和ChannelHandler的方法 是从整个ChannelPipeline开始传播。


# QA
### ChannelFuture是Future和回调的结合，能够避免我们手动去查询结果是否完成，那么ChannelFuture是什么时候知道该调用监听器的回调方法的呢？
猜想：异步执行完成，将结果set进属性里面的时候调用监听器的回调方法
