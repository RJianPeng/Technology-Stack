* [一、字符串](#一字符串)
* [String](#string)
* [StringBuilder](#stringbuilder)
* [StringBuffer](#stringbuffer)

* [二、Object九大通用方法](#二object九大通用方法)
* [clone方法](#clone方法)
* [equals和hashcode方法](#equals和hashcode方法)
* [wait、notify、notifyAll方法](#waitnotifynotifyall方法)
* [finalize、getClass、toString方法](#finalizegetclasstostring方法)

* [三、Java关键字](#三java关键字)
* [private、public、default和protected](#privatepublicdefault和protected)
* [volatile和synchronized](#volatile和synchronized)

# 一、字符串
## String
在Java8中，String用来存储一组连续的字符串，是定义为final的Java类，因此String不可被继承也不可变，String内部使用char的数组存储内容。
因为String不可变的特性，所以对String的修改实质上都是新建了一个String对象:
```
String str = "abc"; //创建一个字符串
String old = str; //创建一个字符串 将str指向的字符串地址赋值给它
System.out.println(str == old);//判定str和old是否指向同一个字符串
str = "abcd";//修改str的值
System.out.println(str == old);//判定str和old是否指向同一个字符串
```
该代码运行结果为
```
true
flase
```
由于String的不可变的特性，为了节省内存空间和减少新建销毁字符串对象的开销，Java引入了字符串常量池的概念。
字符串常量池，可以理解为一块专门存储String的区域，但是他存储的又不是String对象的实例，String对象的实例还是存储在堆中，字符串常量池中存储的只是对象的引用。（具体可见https://www.zhihu.com/question/29884421）
对于字符串常量池的使用，对开发人员并不是透明的，Java提供了三种创建String的方式：
```
String str1 = "first"; //第一种方式：直接声明
String str2 = "second".intern();  //第二种方式：调用String.intern()方法
String str3 = new String("third");  //第三种方式：new一个String
```
其中第一、第二种方式的创建都是先去字符串常量池中寻找值相同的String对象，如果找到了，就直接将新建的String引用指向这个对象，如果没找到，就在常量池中创建一个String对象，在将引用指向这个对象。而第三种方式就是没有使用常量池，直接在堆中新建对象。
测试代码：
```
String str1 = "first"; //第一种方式：直接声明
String str2 = "second".intern();  //第二种方式：调用String.intern()方法
String str3 = new String("third");  //第三种方式：new一个String

String str11 = "first";
String str22 = "second".intern();
String str33 = new String("third");
System.out.println(str1 == str11);
System.out.println(str2 == str22);
System.out.println(str3 == str33);
```
运行结果：
```
true
true
false
```
### String不可变的优势
* 1.天然的保证了线程安全
* 2.可以缓存String的hash值，因为String不可变所以它的hash值也不可变
* 3.可以保证作为参数不可变

## StringBuilder
相比于String，StringBuilder是可变的字符串对象，对它的修改并不会新建一个对象。
StringBuilder内部也是使用一个char的数组来存储字符串，不同的是数组的长度会在字符串增长时进行动态扩展。
扩展的逻辑：
首先判定是否需要扩展，如果新字符串的长度大于数组长度则进行扩展,否则无需扩展：
```
// overflow-conscious code
if (minimumCapacity - value.length > 0) {
    value = Arrays.copyOf(value,
            newCapacity(minimumCapacity));
}
```
扩展的规则为，旧数组长度*2+2，若该值小于新字符串长度，则采用新字符串长度，否则采用该值：
```
private int newCapacity(int minCapacity) {
    // overflow-conscious code
    int newCapacity = (value.length << 1) + 2;
    if (newCapacity - minCapacity < 0) {
        newCapacity = minCapacity;
    }
    return (newCapacity <= 0 || MAX_ARRAY_SIZE - newCapacity < 0)
        ? hugeCapacity(minCapacity)
        : newCapacity;
}
```


## StringBuffer
StringBuffer，与StringBuilder同样继承于AbstractStringBuilder，所以也是可变的字符串对象，大部分逻辑与StringBuilder相同，包括数组的动态扩展。
StringBuffer与StringBuilder的不同之处在于StringBuffer采用了synchronized进行一个加锁的操作。保证了多线程场景下的线程安全。

# 二、Object九大通用方法
## clone方法
实现对象的复制，需要实现Cloneable接口
### new一个对象和clone一个对象的区别
new 操作符的本意是分配内存。程序执行到 new 操作符时，首先去看 new 操作符后面的类型，因为知道了类型， 
才能知道要分配多大的内存空间。分配完内存之后，再调用构造函数，填充对象的各个域，这一步叫做对象的初始化，构造方法返回后，一个对象创建完毕，可以把他的引用（地址）发布到外部，在外部就可以使用这个引用操纵这个对象。

clone 在第一步是和 new 相似的，都是分配内存，调用 clone 方法时，分配的内存和原对象（即调用 clone 方法 
的对象）相同，然后再使用原对象中对应的各个域，填充新对象的域，填充完成之后，clone 方法返回，一个新的相同 
的对象被创建，同样可以把这个新对象的引用发布到外部。

### 深拷贝和浅拷贝
浅拷贝：被复制对象的所有值属性都含有与原来对象的相同，而所有的对象引用属性仍然指向原来的对象。（只把最外层的对象的属性复制一份，其中的引用类型只复制引用的地址）
深拷贝：在浅拷贝的基础上，所有引用其他对象的变量也进行了clone，并指向被复制过的新对象。（在浅拷贝的基础上，对成员中的引用类型的成员也复制一份）


## equals和hashcode方法
equals和hashcode方法都可以用来判定两个方法是否相同。不过他们的判定方式不同，从Java8的源码来看，equals方法：
```
public boolean equals(Object obj) {
        return (this == obj);
}
```
equals方法是直接通过==来判定两个对象是否相等，==的方式要求对象不止内容相同，而且地址也相同，即必须是同一个对象才会返回true
hashcode方法是通过计算两个对象的hash值，通过hash值来判定是否两个对象是否相同。equals（）返回为true的两个对象hashcode（）一定相等，hashcode（）相等的两个对象equals（）不一定返回为true

### 为何重写hashcode就要重写equals
java.lang.Object中的约定：
* 1. 在一个应用程序执行期间，如果一个对象的equals方法做比较所用到的信息没有被修改的话，则对该对象调用hashCode方法多次，它必须始终如一地返回同一个整数。
* 2. 如果两个对象根据equals(Object o)方法是相等的，则调用这两个对象中任一对象的hashCode方法必须产生相同的整数结果。
* 3. 如果两个对象根据equals(Object o)方法是不相等的，则调用这两个对象中任一个对象的hashCode方法，不要求产生不同的整数结果。但如果能不同，则可能提高散列表的性能。 
* 如果不遵守约定，就会出现问题，比如hashset中，会通过hashcode来分配位置，如果重写了equals方法而没有重写hashcode，那么可能会导致两个equals方法返回为true的两个对象都可以被放入hashset中，这样会出现问题。所以equals需要和hashcode方法同时重写。

## wait、notify、notifyAll方法
wait方法：让该对象所属的线程进入睡眠状态并释放锁，直到以下事件发生：
* 1.其他线程调用了该对象的notify方法，随机唤醒一个调用该对象方法的线程
* 2.其他线程调用了该对象的notifyAll方法，唤醒所有调用该对象的wait方法的线程
* 3.其他线程调用了interrupt中断该线程
* 4.等待时间结束

### wait方法和Thread.sleep方法的区别
* sleep：会让该线程进入等待状态，但是不会释放锁，让出cpu去执行其他线程。
* wait：会让当前获得该对象锁的线程进入等待状态，会释放锁。


## finalize、getClass、toString方法
* finalize:用于释放资源，在垃圾回收机制里面是对象自我拯救的一种方式
* getClass：获取运行时类型
* toString：根据对象内容返回一个字符串，经常重写该方法


# 三、Java关键字
## private、public、default和protected
这四个Java关键字都是关于Java中类成员的访问权限的关键字。
### private
用该关键字修饰的类的成员和方法只有这个对象自己能够访问。

### public
用该关键字修饰的类的成员和方法所有的对象都可以访问。

### default
用该关键字修饰的类的成员和方法同一个包下的类可以访问

### protected
用该关键字修饰的类的成员和方法在同一个包下的类以及这个类的子类可以访问

## volatile和synchronized

### volatile
* volatile可以保证变量的可见性

在Java内存模型中，线程需要将变量从主内存读取到工作内存中再进行操作，高并发的场景下，可能会出现线程读取脏数据到工作内存中的情况（即线程A读取了一个主内存中到变量x，线程B对变量x做了修改并赋值到主内存中，此时线程A的工作内存中仍然是x的脏数据）。volatile则会强制修改完变量后，立刻刷写到主内存并强制其他线程重新从主内存中读取变量，避免了脏数据的问题，保证变量的可见性。
