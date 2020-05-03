* [线程的使用](#线程的使用)
* [线程间的通信方式](#线程间的通信方式)
* [Java线程池](#java线程池)

# 线程的使用
线程的使用：
* 1.继承Thread类，实现run方法，Thread实现了Runnable接口，通过start方法启动
```
public class ThreadDemo {
    public void startTaskAsync(){
        Task task = new Task();
        task.start();
    }
    class Task extends Thread{
        @Override
        public void run(){
            System.out.println("this is Thread demo");
        }
    }
}
```

* 2.实现Runnable接口，实现run方法
```
public class ThreadDemo {
    public void startTaskAsync(){
        Task task = new Task();
        Thread thread = new Thread(new Task());
        thread.start();

    }
    class Task implements Runnable{

        @Override
        public void run() {
            System.out.println("this is Runnable demo");
        }
    }
}
```
* 3.实现Callable接口，可以有返回值，通过FutureTask进行封装，实现call方法
```
public class ThreadDemo {
    public void startTaskAsync() throws InterruptedException {
        FutureTask futureTask = new FutureTask(new Task());
        Thread thread = new Thread(futureTask);
        thread.start();
    }
    class Task implements Callable {
        @Override
        public Object call() throws Exception {
            System.out.println("this is Callable demo");
            return 1;
        }
    }
}
```
2，3两种方式最后还是需要通过Thread来进行启动，且第三种方式是需要返回值的，而且是异步执行的，先返回FutureTask对象，任务执行完毕后再将值写入该对象。（关于这部分异步执行的可以看https://github.com/RJianPeng/Technology-Stack/blob/master/%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/Java8%E5%AE%9E%E6%88%98.md#%E7%AC%AC%E5%8D%81%E4%B8%80%E7%AB%A0completablefuture%E7%BB%84%E5%90%88%E5%BC%8F%E5%BC%82%E6%AD%A5%E7%BC%96%E7%A8%8B ）

实现接口的方式更好一点，因为类只能继承一个类，但是可以实现多个接口，同时如果是继承Thread的话开销较大。
* java8中，以上三种方式都可以通过函数式编程来简洁的实现









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







