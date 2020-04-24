* [Java线程池](#java线程池)



# Java线程池
功用：帮助程序员管理线程

常见的四种线程池：
* CachedThreadPool：一个任务创建一个线程；核心线程数为0，最大线程数是int的最大值。他的队列是synchronousQueue,没有缓冲区域的队列，最大线程数为int的最大值，线程没有任务时会持续60秒
* FixedThreadPool：所有任务只能使用固定大小的线程；核心线程数和最大线程数相同
* SingleThreadExecutor：相当于大小为 1 的 FixedThreadPool;核心线程数和最大线程数都为1
* ScheduledThreadPool:核心线程数不定，最大线程数为int的最大值，线程空闲存活时间为0

除了直接通过Executors的静态工厂方法直接生成以上四种常见的线程池之外，开发人员还可以通过ThreadPoolExecutor的构造方法来定义自己的线程池。
//TODO源码解读
构造方法签名为：
```
public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler)
```
* corePoolSize：核心线程池的大小。线程池创建之后不会立即去创建线程，而是等待线程的到来。当当前执行的线程数大于该值时，线程会加入到缓冲队列，核心线程空闲时不会被销毁，而是会阻塞在获取任务的方法上。
* maximumPoolSize：线程池中创建的最大线程数；
* keepAliveTime：空闲的非核心线程多久时间后被销毁。默认情况下，该值在线程数大于corePoolSize时，对超出corePoolSize值的这些线程即非核心线程起作用。
* unit：TimeUnit枚举类型的值，代表keepAliveTime时间单位
* workQueue：阻塞队列，用来存储等待执行的任务，决定了线程池的排队策略，有以下取值：ArrayBlockingQueue;LinkedBlockingQueue;SynchronousQueue;
* threadFactory：线程工厂，是用来创建线程的。默认new Executors.DefaultThreadFactory();
* handler:线程拒绝策略。

拒绝策略主要有四种：
* 1.AbortPolicy:丢弃任务并抛出RejectedExecutionException
* 2.CallerRunsPolicy：只要线程池未关闭，该策略直接在调用者线程中，运行当前被丢弃的任务。显然这样做不会真的丢弃任务，但是，任务提交线程的性能极有可能会急剧下降。谁调用线程谁执行
* DiscardOldestPolicy：丢弃队列中最老的一个请求，也就是即将被执行的一个任务，并尝试再次提交当前任务。
* DiscardPolicy：丢弃任务，不做任何处理。

### 线程池的工作流程
1.线程池内的线程数量小于定义的核心线程数时，新的任务提交过来，直接创建一个新的线程执行。
2.线程池内的数量大于等于核心线程数时，新的任务提交过来，会先放到阻塞队列里，等待空闲的核心线程来执行。
3.阻塞队列满的时候且线程池内的线程数量小于最大线程数时，会创建新的线程到线程池中。
4.当阻塞队列满当时候且线程池的线程数量等于最大线程数时，新的任务过来，线程池会根据定义的拒绝策略处理新来的任务。







