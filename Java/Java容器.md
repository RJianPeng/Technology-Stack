* [Collection](#collection)
* [Map](#map)
* [迭代器](#迭代器)
* [AQS](aqs)

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

#### HashMap在Java1.7和Java1.8中的区别
###### JDK1.7中
使用一个Entry数组来存储数据，用key的hashcode取模来决定key会被放到数组里的位置，如果hashcode相同，或者hashcode取模后的结果相同（hash collision），那么这些key会被定位到Entry数组的同一个格子里，这些key会形成一个链表。
在hashcode特别差的情况下，比方说所有key的hashcode都相同，这个链表可能会很长，那么put/get操作都可能需要遍历这个链表
也就是说时间复杂度在最差情况下会退化到O(n)
 
###### JDK1.8中
使用一个Node数组来存储数据，但这个Node可能是链表结构，也可能是红黑树结构
如果插入的key的hashcode相同，那么这些key也会被定位到Node数组的同一个格子里。
如果同一个格子里的key不超过8个，使用链表结构存储。
如果超过了8个，那么会调用treeifyBin函数，将链表转换为红黑树。
那么即使hashcode完全相同，由于红黑树的特点，查找某个特定元素，也只需要O(log n)的开销
也就是说put/get的操作的时间复杂度最差只有O(log n)
但是使用红黑树有一个限制那就是key必须实现了comparable



### HashTable
HashTable：和 HashMap 类似，但它是线程安全的，这意味着同一时刻多个线程可以同时写入 HashTable 并且不会导致数据不一致。但是它是遗留类，不应该去使用它。现在可以使用 ConcurrentHashMap 来支持线程安全，并且 ConcurrentHashMap 的效率会更高，因为 ConcurrentHashMap 引入了分段锁（jdk1.7）。

### ConcurrentHashMap
ConcurrentHashMap：与HashMap类似，但是采用了分段锁，每一段锁控制几个桶，一个ConcurrentHashMap的多个段可以被多个线程同时访问,分段锁（Segment）继承于ReentrantLock。concurrentHashmap里面每个segment都有一个count属性用来统计该segment中键值对的个数，通过size（）获取，在执行size（）的时候，先尝试不加锁，如果连续两次不加锁的结果相同，那么认为这个结果是正确的。如果尝试次数超过3次，就加锁计算。
* java8之后用volatile加CAS替换了segment。

### LinkedHashMap
LinkedHashMap：使用双向链表来维护插入顺序的HashMap，有一个最大缓存空间，超过最大缓存空间的时候会通过LRU（最近最少使用算法）进行替换

#### 哈希表解决哈希冲突的方法
* 开放地址法：线性探测、二次哈希
* 链地址法：<div align="center"> <img src="https://github.com/RJianPeng/Technology-Stack/blob/master/Java/photo/%E9%93%BE%E5%9C%B0%E5%9D%80%E6%B3%95.png"/></div><br>
* 建立公共溢出区：将哈希表分为基本表和溢出表



# 迭代器
迭代器模式：就是提供一种方法对一个容器对象中的各个元素进行访问，而又不暴露该对象容器的内部细节。

因为容器内部的结构不同，很多时候不知道如何去遍历一个容器内的元素，所以为了使对容器内元素操作简单，Java引入了迭代器模式。

## iterator
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

## iterable
与Iterator配合使用的是Iterable接口，这个接口的功能为返回一个Iterator，实现该接口的子接口有：Collection<E>、List<E>、Set<E>。对于实现了Iterable接口的容器，我们可以采用foreach的方式对其进行遍历，foreach可以看成使用迭代器进行遍历。

Iterable源码：
```
public interface Iterable<T> {
    //返回一个迭代器
    Iterator<T> iterator();

    //实现该接口的容器中每个元素执行该参数行为
    default void forEach(Consumer<? super T> action) {
        Objects.requireNonNull(action);
        for (T t : this) {
            action.accept(t);
        }
    }

    /**
     * Creates a {@link Spliterator} over the elements described by this
     * {@code Iterable}.
     *
     * @implSpec
     * The default implementation creates an
     * <em><a href="Spliterator.html#binding">early-binding</a></em>
     * spliterator from the iterable's {@code Iterator}.  The spliterator
     * inherits the <em>fail-fast</em> properties of the iterable's iterator.
     *
     * @implNote
     * The default implementation should usually be overridden.  The
     * spliterator returned by the default implementation has poor splitting
     * capabilities, is unsized, and does not report any spliterator
     * characteristics. Implementing classes can nearly always provide a
     * better implementation.
     *
     * @return a {@code Spliterator} over the elements described by this
     * {@code Iterable}.
     * @since 1.8
     */
    //返回一个分割器
    default Spliterator<T> spliterator() {
        return Spliterators.spliteratorUnknownSize(iterator(), 0);
    }
}
```

使用Iterator的时候不能对集合采用add/remove的操作，否则会抛出ConcurrentModificationException异常。因为Iterator生成之后，你对集合的add/remove操作，iterator无法同步，Java会认为这种操作是线程不安全的。

Iterator在ArrayList中的实现：
```
private class Itr implements Iterator<E> {
 int cursor;       // 下一个返回的元素的位置
 int lastRet = -1; // 上一个返回的元素的位置，没有则为-1
 int expectedModCount = modCount; //iterator的版本号，与ArrayList的版本号相同。（在调用ArrayList.add/remove时版本号会加1）

 Itr() {}

 public boolean hasNext() {
     return cursor != size;
 }

 @SuppressWarnings("unchecked")
 public E next() {
     checkForComodification();
     int i = cursor;
     if (i >= size)
         throw new NoSuchElementException();
     Object[] elementData = ArrayList.this.elementData;
     if (i >= elementData.length)
         throw new ConcurrentModificationException();
     cursor = i + 1;
     return (E) elementData[lastRet = i];
 }

 public void remove() {
     if (lastRet < 0)
         throw new IllegalStateException();
     checkForComodification();

     try {
         ArrayList.this.remove(lastRet);
         cursor = lastRet;
         lastRet = -1;
         expectedModCount = modCount;
     } catch (IndexOutOfBoundsException ex) {
         throw new ConcurrentModificationException();
     }
 }

 @Override
 @SuppressWarnings("unchecked")
 public void forEachRemaining(Consumer<? super E> consumer) {
     Objects.requireNonNull(consumer);
     final int size = ArrayList.this.size;
     int i = cursor;
     if (i >= size) {
         return;
     }
     final Object[] elementData = ArrayList.this.elementData;
     if (i >= elementData.length) {
         throw new ConcurrentModificationException();
     }
     while (i != size && modCount == expectedModCount) {
         consumer.accept((E) elementData[i++]);
     }
     // update once at end of iteration to reduce heap write traffic
     cursor = i;
     lastRet = i - 1;
     checkForComodification();
 }

 final void checkForComodification() {
     if (modCount != expectedModCount)
         throw new ConcurrentModificationException();
 }
}
```
可以看到方法next()最先调用了checkForComodification方法，这个方法会校验迭代器的版本号和容器的版本号是否相同，如果不同就会抛出ConcurrentModificationException异常。而在迭代过程中调用iterator.remove是可以的因为这个方法同步了容器的版本号和迭代器的版本号。这种实现方式是为了实现ArrayList的fail-fast（快速失败）机制。快速失败机制产生的条件：当多个线程对Collection进行操作时，若其中某一个线程通过Iterator遍历集合时，该集合的内容被其他线程所改变，则会抛出ConcurrentModificationException异常。



## fail-fast机制
在系统设计中，快速失效系统一种可以立即报告任何可能表明故障的情况的系统。快速失效系统通常设计用于停止正常操作，而不是试图继续可能存在缺陷的过程。这种设计通常会在操作中的多个点检查系统的状态，因此可以及早检测到任何故障。快速失败模块的职责是检测错误，然后让系统的下一个最高级别处理错误。

Java中的迭代器支持这种机制。












