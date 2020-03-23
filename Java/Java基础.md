* [一、字符串](#一字符串)
* [String](#string)
* [StringBuilder](#stringbuilder)
* [StringBuffer](#stringbuffer)

* [二、Object九大通用方法]
* [clone方法]
* [equals和hashcode方法]
* [wait、notify、notifyAll方法]
* [finalize、getClass、toString方法]




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
字符串常量池，可以理解为一块专门存储String的区域，在Java8中，这块区域在堆中。
（字符串常量池的位置是借鉴了https://blog.csdn.net/MustangJy/article/details/88044964 中的方法，并进行了实测，与该博客中结果一致）
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
