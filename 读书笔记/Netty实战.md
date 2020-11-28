* -[第一章、Netty——异步和事件驱动](#第一章Netty异步和事件驱动)
* -[第二章、你的第一款Netty应用程序](#第二章你的第一款Netty应用程序)

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



# QA
### ChannelFuture是Future和回调的结合，能够避免我们手动去查询结果是否完成，那么ChannelFuture是什么时候知道该调用监听器的回调方法的呢？
