* -[第一章、Netty——异步和事件驱动](#第一章Netty异步和事件驱动)

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
