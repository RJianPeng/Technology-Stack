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

* [四、接口和抽象类](#四接口和抽象类)
* [接口](#接口)
* [抽象类](#抽象类)
* [多态](#多态)

* [五、代码编写小细节](#五代码编写小细节)


* [六、基本类型和封装类型](#六基本类型和封装类型)

* [七、异常](#七异常)
* [检查异常](#检查异常)
* [非检查异常](非检查异常)

* [八、Java内部类](#java内部类)
* [成员内部类](#成员内部类)
* [局部内部类](#局部内部类)
* [匿名内部类](#匿名内部类)
* [静态内部类](#静态内部类)

* [九、范型](#九范型)


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
用该关键字修饰的类的方法同一个包下的类可以访问,一般用于接口中修饰静态方法，不能用来修饰成员变量。

### protected
用该关键字修饰的类的成员和方法在同一个包下的类以及这个类的子类可以访问



## final
final关键字用法：
* 1.修饰类时：表示这个类不允许被继承
* 2.修饰方法时：表示这个方法不允许被子类覆盖和重写，类的private方法默认加了final修饰
* 3.修饰变量时：表示这个变量为常量，不允许修改它的值。被修饰的变量只能在声明时/构造器中/初始化代码块中赋值，且之后都无法修改它的值。

## throw和throws
### throw
声明抛出某种异常，且抛出的异常需要在后续通过catch进行捕获处理或者是交由方法的调用方处理。

### throws
声明这部分的代码可能出现哪些异常，并将这些异常交给上一层处理

# 四、接口和抽象类
## 抽象类：
* 1.抽象类是用abstract class声明的特殊的类
* 2.抽象类不能实例化（除了这点外和普通的类几乎没啥区别）
* 3.抽象类中可以有用abstract修饰的抽象方法，抽象方法没有具体实现。
* 4.抽象类中也可以包含有具体实现的普通方法
* 5.抽象类中可以有main方法并且可以运行

## 接口：
* 1.接口是用interface声明的
* 2.接口不能被实例化
* 3.接口中的方法都是抽象方法，Java8引入了使用default修饰的默认方法，这个引入主要是为了以兼容的方式解决Java API这样的类库的演进问题。而接口的默认方法也会造成一定的问题：假设两个接口有相同签名的默认方法，如果其中一个接口继承了另一个接口，那这个类会选择子接口中的实现。否则实现这两个接口的类必须重写这个方法，否则会冲突报错。也由于类可以实现多个接口，结合默认方法使用，在某些情况下更适合复用代码。
* 4.接口中不能包含实例变量静态方法，包括main方法

### 对比
* 1.一个类可以实现多个接口，只能继承一个类
* 2.子类通过extends声明继承抽象类并提供抽象方法的实现，类通过implements声明实现接口并提供接口中的抽象方法的实现。
* 3.接口不能有构造器，抽象类可以有构造器。
* 4.抽象类中可以包含实例变量，接口中不行。

## 多态
首先，什么是多态，我认为多态就是同一个接口，使用不同的实例就会执行不同的操作。
* 重载和重写是多态的不同表现形式。
* 重载：在一个类里面，函数名相同但是参数类型不同。
* 重写：在继承关系中，子类对父类中的方法进行重新定义。此时的多态表现形式需要父子继承关系，向上转型。向上转换即将父类的引用类型指向子类的实例，比如 A a, B b,A为B的父类，A a=new B ();此时就是实例化了一个类为B的对象，向上转型为它的父类对象。此时，a既可以调用父类中的功能，也可以调用子类中的功能，但是必须是在父类中声明过的功能。即父类引用可以指向子类实例。


# 五、代码编写小细节

### 变量声明
```
int a ,b = 1; //这种方式只是声明了a的变量，并没有初始化a，只初始化了b。

int a = 1;
int b = 1; //这种方式才初始化了两个变量
```


### String.substring方法
```
String str = "hello";
str.substring(0,2);//这个的返回值为he，返回的是第一个参数开始到第二个参数前一个子字符串
```

### 代码运行顺序
* 父类静态初始化块
* 子类静态初始化块
* 父类初始化块
* 父类构造器
* 子类初始化块
* 子类构造器


## 六、基本类型和封装类型
Java中存在8中基本类型：

int      4字节 Integer

long     8字节 Long

short    2字节 Short

double   8字节 Double

float    4字节 Float

char     2字节 Character

byte     1字节 Byte

boolean  1字节 Boolean

基本类型都存在其对应的封装类型，基本类型和封装类型之间的类型转换称为封箱/拆箱。转换过程都是隐式的。
* 基本类型的参数传递是值传递，封装类型的参数传递是引用传递。


# 七、异常
关于Java中各种常见异常的关系如图：
<div align="center"> <img src="https://github.com/RJianPeng/Technology-Stack/blob/master/Java/photo/%E5%BC%82%E5%B8%B8.png"/></div><br>

## 检查异常
除了Error 和 RuntimeException的其它异常。javac强制要求程序员为这样的异常做预备处理工作（使用try…catch…finally或者throws）。在方法中要么用try-catch语句捕获它并处理，要么用throws子句声明抛出它，否则编译不会通过。这样的异常一般是由程序的运行环境导致的。因为程序可能被运行在各种未知的环境下，而程序员无法干预用户如何使用他编写的程序，于是程序员就应该为这样的异常时刻准备着。如SQLException , IOException,ClassNotFoundException 等。

## 非检查异常
Error 和 RuntimeException 以及他们的子类。javac在编译时，不会提示和发现这样的异常，不要求在程序处理这些异常。所以如果愿意，我们可以编写代码处理（使用try…catch…finally）这样的异常，也可以不处理。对于这些异常，我们应该修正代码，而不是去通过异常处理器处理 。

* 最后，检查异常和非检查异常都是针对编译过程来说的


# 八、Java内部类

## 成员内部类
最普通的内部类，定义在一个类的内部，可以有访问控制来修饰。内部类可以无条件的调用外部类的方法，成员变量等。如果在外部类中有和内部类重名的方法或成员变量，则默认情况下是访问的自己的方法和成员变量。可以通过
外部类.this.成员变量
外部类.this.成员方法
这种方式来对外部类对成员变量和方法进行显示的调用。

对内部类的实例化需要依靠外部类：
```
Outer outer = new Outer();
Outer.Inner inner = outer.new Inner();
```

## 局部内部类
定义在方法或者代码段中的类，局部内部类的访问仅限于方法内或者该作用域内。不能由访问权限修饰符.
```
public class Inner{
    AtomicInteger atomicIntegerInner;
    public void test(){
        class T{
            int a = 0;
        }
        T t = new T();
    }
}
```

## 匿名内部类
定义并实例化一个临时的类
```
Thread t = new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("匿名内部类");
            }
        });
```

## 静态内部类
不能使用外部类的非static方法和变量

不依赖与外部类就可以实例化：
```
Outer.Inner inner = new Outer.Inner();
```

# 九、范型
java的泛型在编译期有效，在运行期会被删除

注意这个地方因为类型擦除可能会出现方法里面集合类型不同，但是报错方法重复
即：
List<String>、List<T>类型擦除之后为List。

List<String>[]、List<T>类型擦除之后为List。
* 这就是类型擦除

注意：
* 1.范型







