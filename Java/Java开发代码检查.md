# sonar代码检查点
### InterruptedException正确处理方式
#### InterruptedException的处理方式及可能存在的问题
在出现中断时，线程中是否被中断的标志位变为true，但是在catch住异常之后，会把标志位重置为false，这样就会丢失线程被中断的状态信息。
可以在catch之后调用：Thread.currentThread().interrupt(); 将标志位改为true。

#### 不应该在thread上使用wait()、notify()、notifyAll()方法
原因：
* 1.在内部，JVM依赖这些方法来更改Thread（Blocked,Waiting...）的状态，因此调用他们会破坏JVM的行为


#### 通过双括号对对象进行初始化的问题
Double Brace Initialization
```
//通过双括号的方式对map进行初始化
final Set<String> map = new HashSet<String>() {{
2     add(‘Alice’);
3     add(‘Bob’);
4     add(‘Marine’);
5 }};
```
这种写法其实是新建了一种HashSet的匿名内部类，通过代码块的方式来进行集合成员的初始化工作：
```
final Set<String> exclusions = new HashSet<String>() {
    {
        add(‘Alice’);
        add(‘Bob’);
        add(‘Marine’);
    }
};
```
这种写法的优缺点：
* 1.对于熟悉的人来说，可读性很高

* 1.可能造成内存泄漏