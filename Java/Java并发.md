* [Java线程](#java线程)
* [线程间的通信方式](#线程间的通信方式)
* [Java线程池](#java线程池)
* [线程安全工具类](#线程安全工具类)
* [乐观锁和悲观锁](#乐观锁和悲观锁)

# Java线程
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

## Thread的start和run方法的区别
* 1.start（）方法来启动线程，真正实现了多线程运行。这时无需等待run方法体代码执行完毕，可以直接继续执行下面的代码；通过调用Thread类的start()方法来启动一个线程， 这时此线程是处于就绪状态， 并没有运行。 然后通过此Thread类调用方法run()来完成其运行操作的， 这里方法run()称为线程体，它包含了要执行的这个线程的内容， Run方法运行结束， 此线程终止。然后CPU再调度其它线程。

* 2.run（）方法当作普通方法的方式调用。程序还是要顺序执行，要等待run方法体执行完毕后，才可继续执行下面的代码； 程序中只有主线程——这一个线程， 其程序执行路径还是只有一条，这样就是直接在当前线程中执行任务而没有达到多线程的目的。

## Thread常见方法
Thread.join()方法：
A线程在B线程里面调用了A的join方法，则必须等到A线程执行完毕B线程才会继续执行。

Thread.yield()方法：
译为线程让步。顾名思义，就是说当一个线程使用了这个方法之后，它就会把自己CPU执行的时间让掉 但是是让给优先级大于等于这个线程的线程。

## 线程的状态
线程状态如图<div align="center"> <img src="https://github.com/RJianPeng/Technology-Stack/blob/master/Java/photo/%E7%BA%BF%E7%A8%8B%E7%8A%B6%E6%80%81.png" width="700px"> </div><br>

（ 图片转自https://github.com/CyC2018/CS-Notes )



# 线程间的通信方式
## volatile和synchronized

### volatile
* volatile可以保证变量的可见性

在Java内存模型中，线程需要将变量从主内存读取到工作内存中再进行操作，高并发的场景下，可能会出现线程读取脏数据到工作内存中的情况（即线程A读取了一个主内存中到变量x，线程B对变量x做了修改并赋值到主内存中，此时线程A的工作内存中仍然是x的脏数据）。volatile则会强制修改完变量后，立刻刷写到主内存并强制其他线程重新从主内存中读取变量，避免了脏数据的问题，保证变量的可见性。

### synchronized
原理：每个对象头都有个监视器锁（monitor），当monitor被占用时会处于锁定状态，线程执行到synchronized的时候会尝试获取monitor的权限。线程进来的时候会先进行尝试性抢占然后再到等待队列中，所以是非公平的。锁一共有四种状态：无锁状态  偏向锁  轻量级锁  重量级锁。锁只能升级不能降级。

synchronized底层对对象锁的使用：
* 1，当没有被当成锁时，这就是一个普通的对象，Mark Word记录对象的HashCode，锁标志位是01，是否偏向锁那一位是0。

* 2，当对象被当做同步锁并有一个线程A抢到了锁时，锁标志位还是01，但是否偏向锁那一位改成1，前23bit记录抢到锁的线程id，表示进入偏向锁状态。

* 3，当线程A再次试图来获得锁时，JVM发现同步锁对象的标志位是01，是否偏向锁是1，也就是偏向状态，Mark Word中记录的线程id就是线程A自己的id，表示线程A已经获得了这个偏向锁，可以执行同步锁的代码。

* 4，当线程B试图获得这个锁时，JVM发现同步锁处于偏向状态，但是Mark Word中的线程id记录的不是B，那么线程B会先用CAS操作试图获得锁，这里的获得锁操作是有可能成功的，因为线程A一般不会自动释放偏向锁。如果抢锁成功，就把Mark Word里的线程id改为线程B的id，代表线程B获得了这个偏向锁，可以执行同步锁代码。如果抢锁失败，则继续执行步骤5。

* 5，偏向锁状态抢锁失败，代表当前锁有一定的竞争，偏向锁将升级为轻量级锁。JVM会在当前线程的线程栈中开辟一块单独的空间，里面保存指向对象锁Mark Word的指针，同时在对象锁Mark Word中保存指向这片空间的指针。上述两个保存操作都是CAS操作，如果保存成功，代表线程抢到了同步锁，就把Mark Word中的锁标志位改成00，可以执行同步锁代码。如果保存失败，表示抢锁失败，竞争太激烈，继续执行步骤6。

* 6，轻量级锁抢锁失败，JVM会使用自旋锁，自旋锁不是一个锁状态，只是代表不断的重试，尝试抢锁。从JDK1.7开始，自旋锁默认启用，自旋次数由JVM决定。如果抢锁成功则执行同步锁代码，如果失败则继续执行步骤7。

* 7，自旋锁重试之后如果抢锁依然失败，同步锁会升级至重量级锁，锁标志位改为10。在这个状态下，未抢到锁的线程都会被阻塞。

## volatile和synchronized的区别

1.volatile是线程同步的轻量级实现，所以volatile的性能要比synchronize好；volatile只能用于修饰变量，synchronize可以用于修饰方法、代码块。随着jdk技术的发展，synchronize在执行效率上会得到较大提升，所以synchronize在项目过程中还是较为常见的；
 
2.多线程访问volatile不会发生阻塞，所以没有原子性；而synchronize会发生阻塞；

3.volatile能保证变量在私有内存和主内存间的同步(可见性在一定程度上保证有序性)，但不能保证变量的原子性；(lock)synchronize可以保证变量原子性；

4.volatile保证变量在多线程之间的可见性；synchronize是多线程之间访问资源的同步性；所有同步操作都要保证其 原子性与可见性,有序性;



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

### 线程池执行任务
线程池对任务的执行：
* execute()：无返回值
* submit()：有返回值 FutureTask



# 线程安全工具类

## ConcurrentHashMap
与HashMap类似，但是采用了分段锁，每一段锁控制几个桶，一个ConcurrentHashMap的多个段可以被多个线程同时访问,分段锁（Segment）继承于ReentrantLock
java8里面改用volatile和CAS实现线程安全

## CopyOnWriteArrayList
读写分离的数组，写操作通过ReentranLock加锁,写操作是复制一个新的数组进行写操作，结束之后将旧的数组指向新的数组，在写操作的时候允许读操作

## BlockingQueue
该接口有以下实现 1.FIFO队列 LinkedBlockingQueue    ArrayBlockingQueue(长度固定)  2.优先级队列 PriorityBlockingQueue
提供了put()和take()方法，如果队列为空take()会阻塞，如果队列满了put()会阻塞
用来实现生产者和消费者问题，是线程池中参数之一

## AtomicInteger
能保证这个类型的数据在增减的时候保持原子性，通过CAS实现

## CountDownLatch
用来控制一个线程等待多个线程，维护了一个计数器，每次调用countDown（）方法会让计数器的值减1，计数器为0的时候，那些调用了await（）而等待的线程开始继续

## CyclicBarrier
用来控制多个线程互相等待，只有在多个线程都到达时才会继续进行，与CountDownLatch相似，await()执行之后计数器才会减1，并进行等待，等计数器为0的时候，这些调用了await的线程才会继续。与CountDownLatch不同的是可以使用reset（）方法循环使用计数器或者await屏障通过之后重置

## FutureTask
实现了RunnableFuture接口，这个接口继承于Runnable和Future接口。可用于异步获取执行结果或取消执行任务的场景。通过传入Runnable或Callable的任务给FutureTask，可以直接调用其run方法或放入线程池，之后可以通过FutureTask的get方法获取返回值。
另外，FutureTask还可以确保即使调用了多次run方法，它都只会执行一次Runnable或者Callable任务，或者通过cancel取消FutureTask的执行等。

## Semphore
类似于信号量，控制对互斥资源的访问线程数  acquire()获取  release()释放。信号量为0的时候acquire阻塞。

## Condition
实现线程之间的协调，通过await（）使线程等待，通过signal和signalAll()进行唤醒，相对于wait方法，await可以指定等待的条件，更加灵活。await方法还可以将与其Condition绑定的Reetranlock的锁释放掉，为唤醒的时候重新获得。condition的获得必须通过reetrantlock

Condition的使用方式：
```
ReentrantLock reentrantLock = new ReentrantLock();
//通过reentrantLock获取condition
Condition condition = reentrantLock.newCondition();
//等待condition
condition.await();
//随机唤醒一个等待condition的线程
condition.signal();
//唤醒所有等待condition的线程
condition.signalAll();
```

# 乐观锁和悲观锁
乐观锁就是很乐观，每次拿数据的时候都认为别人不会修改，所以不会上锁，但在更新的时候会检查下数据在中途是否有被修改过。适用于读多的情况，能提高吞吐量

悲观锁就是很悲观，在拿数据的时候认为别人一定会修改，所以一定会上锁，直到操作结束。
悲观锁的缺点：多线程情况下，加锁解锁会造成很大的开销，造成性能问题。

## CAS
是乐观锁这种思想的一种实现。当多个线程尝试修改一个共享变量的时候，只有一个线程会修改成功，其他线程会被告知失败。
CAS技术中，需要三个操作数，需要读写的内存位置（V），预期原值（A），新值（B），如果V中的值与A相匹配则认为没有线程修改过这个值，则将V中的该位置更新为B
AtomicInteger就是CAS


CAS缺点：
* 1.ABA问题
比如一个线程从V中取出了A，另一个线程也从V中取出了A。第一个线程把A变成了B然后又变成了A，这是第二个线程仍然能正常的操作
* 2.循环时间开销大
自旋CAS（不成功就一直尝试）如果长时间不成功会造成很大开销
* 3.只能保证一个共享变量的原子性操作

## 死锁产生的四个条件
* 1.互斥
* 2.占有且等待
* 3.非抢占
* 4.循环等待









