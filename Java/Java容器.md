* [Collection](#collection)
* [Map](#map)

# Collection

## List

### ArrayList
ArrayList：基于动态数组实现，支持随机访问。扩容的时候是变成原来的1.5倍，创建一个新的数组

### LinkedList
LinkedList：和ArrayList一样支持动态扩展，基于双向链表实现；

与ArrayList的比较：
* 1.基于双向链表实现，ArrayList基于动态数组实现   
* 2.不支持随机访问  
* 3.删除或添加元素更快（这一点是数组和链表对比的共性）

### Vector
Vector：使用了synchronized进行同步，访问速度更慢，线程安全的，开销比较大，扩容时变成原来的2倍。

### CopyOnWriteArrayList
CopyOnWriteArrayList：读写分的ArrayList升级版，写操作通过ReentranLock加锁,写操作是复制一个新的数组进行写操作，结束之后将旧的数组指向新的数组，在写操作的时候允许读操作。适合读多写少的情形。

## Set

### HashSet
HashSet：基于哈希表实现，支持快速查找，不支持有序性查找，查找的时间复杂度为O(1)。内部通过hashMap实现，但是是简化的hashmap，只有hashmap的key

### TreeSet
TreeSet：基于红黑树（是一种平衡搜索二叉树，左边子节点比父节点小，右边子节点比父节点大）实现，支持有序性操作。查找效率为O(logN）

### LinkedHashSet
LinkedHashSet：具有HashSet的查找效率，同时使用双向链表维护插入的顺序

# Map

### TreeMap
TreeMap：基于红黑树实现

### HashMap
HashMap：基于哈希表实现，使用的是拉链发解决冲突，插入键值对的时候采用头插法，默认容量是16（必须是2的n次方） 

与HashTable的比较：
* 1.HashTable使用synchronized进行同步 
* 2.HashMap可以输入key为null的键值对,但是无法调用key为null的hashCode()方法，所以特殊指定0个桶来存放  
* 3.HashMap的迭代器是Fail-Fast迭代器，hashtable是enumerator迭代器
* 4.HashMap不能保证Map中元素的顺序不变。

#### 为啥默认容量必须是2的n次方
为了散列更均匀.在HashMap的源码中，对key的哈希值的计算是通过先计算其哈希值，然后再与数组长度取模得出的。在源码中，取模的操作如下：
```
static int indexFor(int h, int length) {  
       return h & (length-1);  
} 
```
这里是通过将哈希值与数组长度-1的值做一次&（与）操作，得出来的即为在数组中的位数。
设想如果数组长度为15，15-1的二进制为1110，这样就会导致与的结果里不可能出现末尾为1的结果，导致哈希的碰撞几率大大提高。
如果数组长度为16，16-1的二进制为1111，与的结果相对于上面的会更加均匀的分布在数组上。


### HashTable
HashTable：和 HashMap 类似，但它是线程安全的，这意味着同一时刻多个线程可以同时写入 HashTable 并且不会导致数据不一致。但是它是遗留类，不应该去使用它。现在可以使用 ConcurrentHashMap 来支持线程安全，并且 ConcurrentHashMap 的效率会更高，因为 ConcurrentHashMap 引入了分段锁（jdk1.7）。

### ConcurrentHashMap
ConcurrentHashMap：与HashMap类似，但是采用了分段锁，每一段锁控制几个桶，一个ConcurrentHashMap的多个段可以被多个线程同时访问,分段锁（Segment）继承于ReentrantLock。concurrentHashmap里面每个segment都有一个count属性用来统计该segment中键值对的个数，通过size（）获取，在执行size（）的时候，先尝试不加锁，如果连续两次不加锁的结果相同，那么认为这个结果是正确的。如果尝试次数超过3次，就加锁计算。
java8之后用volatile加CAS替换了segment 

### LinkedHashMap
LinkedHashMap：使用双向链表来维护插入顺序的HashMap，有一个最大缓存空间，超过最大缓存空间的时候会通过LRU（最近最少使用算法）进行替换

#### 哈希表解决哈希冲突的方法
* 开放地址法：线性探测、二次哈希
* 链地址法：<div align="center"> <img src="https://github.com/RJianPeng/Technology-Stack/blob/master/Java/photo/%E9%93%BE%E5%9C%B0%E5%9D%80%E6%B3%95.png"/></div><br>
* 建立公共溢出区：将哈希表分为基本表和溢出表



# 迭代器
迭代器模式：就是提供一种方法对一个容器对象中的各个元素进行访问，而又不暴露该对象容器的内部细节。

因为容器内部的结构不同，很多时候不知道如何去遍历一个容器内的元素，所以为了使对容器内元素操作简单，Java引入了迭代器模式。

在Java中迭代器为一个接口，提供了迭代器基本的定义：
```
public interface Iterator<E> {
    //判断是否存在下一个元素
    boolean hasNext();

    //返回下一个元素
    E next();

    //移除当前元素
    default void remove() {
        throw new UnsupportedOperationException("remove");
    }

    //对容器中的每一个元素执行action的操作
    default void forEachRemaining(Consumer<? super E> action) {
        Objects.requireNonNull(action);
        while (hasNext())
            action.accept(next());
    }
}
```







