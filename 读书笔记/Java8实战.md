* [第一章 为什么要关心Java8](#第一章为什么要关心java8)
* [第三章 lambda表达式](#第三章lambda表达式)
* [第四章 引入流](#第四章引入流)
* [第五章 使用流](#第五章使用流)
* [第六章 用流收集数据](#第六章用流收集数据)
* [第七章 并行数据处理与性能](#第七章并行数据处理与性能)
* [第八章 重构、测试和调试](#第八章重构测试和调整)
* [第九章 默认方法](#第九章默认方法)
* [第十章 用Optional取代null](#第十章用optional取代null)
* [第十一章 CompletableFuture:组合式异步编程](#第十一章completablefuture组合式异步编程)

# 第一章、为什么要关心Java8
### 1.1.2 流处理
stream可以把输入的不相关的部分拿到几个CPU上分别执行，这是几乎免费的并行.

* 可以把stream看作把lambda和接口默认方法加入Java8的直接原因.


### 1.1.3 用行为参数化把代码传递给方法
* 将方法作为参数传递给另一个方法---行为参数化

* 我理解的这种方式就是像引用对象一样直接引用方法，这样就可以直接使用方法，而不用通过引用对象再来使用方法。而所谓的函数式接口可以理解为一个方法引用


### 1.2.2
软件工程中复制粘贴的危险--给一个做了更新和修正，却忘了另一个

## 1.3 流
collection主要为了存储和访问数据 stream主要用于描述对数据的计算

## 1.4 默认方法
接口中加入默认方法是为了写出更容易改进的接口



# 第三章、lambda表达式
# 3.1 lambda管中窥豹
lambda表达式可以被赋予一个变量，或传递给一个接受函数式接口作为参数的方法，当然这个lambda表达式的签名要和函数式接口的抽象方法一样
方法的签名即 参数类型->返回类型 比如 test（Apple a,Apple b){return 10;)的签名就是(Apple,Apple)->int
函数式接口的抽象方法的签名叫做函数描述符

Java.util.function里面有常用的函数式接口
主要是 Predicate、Consumer、Function

范型只能绑定到引用类型，装箱拆箱会在性能上付出代价。

lambda类型检查过程：p49

### 3.5.2 
如果一个lambda的主体是一个语句表达式，他就和一个返回void的函数描述符兼容

### 3.5.4 使用局部变量
lambda允许使用自由变量（不是参数，而是在外层作用域中定义的变量），自由变量也有限制：不可变

## 3.6 方法引用
方法引用：Apple::getweight
：：前面是目标引用，后面是方法名

构造函数的方法引用为类名：：new

# 第四章、引入流
## 4.1 流是什么
* 流是Java API的新成员，它允许你以声明式方式处理数据集合。此外，流还可以透明地并行处理。

## 4.2 流简介
* 流的定义：从支持数据处理操作的源生成的元素序列
* 集合讲的是数据 流讲的是计算
* 流只能被消费一次

### 4.3.2 外部迭代和内部迭代
* 外部迭代：显示的对集合进行遍历
* 内部迭代：告诉stream要做什么 他会自己进行迭代。内部迭代可以自动选择一种适合你硬件的数据表示和并行处理

## 4.4 流操作
### 4.4.1 中间操作
除非流水线上触发一个终端操作，否则中间操作不会执行任何处理。中间操作可以连接起来形成一个操作

### 4.4.2 终端操作
终端操作：从流的流水线生成任何不是流的结果


# 第五章、使用流

## 5.1 筛选和切片：
* filter()用谓词进行筛选
* distinct()返回一个元素各异（根据流元素的hashcode方法和equals方法判断）的流
* limit()截断前n个元素返回
* skip()跳过前n个元素

## 5.2 映射：
* map()接受一个函数作为参数，这个函数会应用到每个元素中，并映射成一个新的元素
* flatMap()扁平化多个流为一个流 即flatMap方法接受的方法参数让你把流中的每一个值都换成另一种流元素，然后flatMap把所有的流元素连接起来形成一个新的流

## 5.3 查找和匹配：
* anyMatch()校验流中是否至少有一个元素能够匹配给定的谓词 是终端操作
* allMatch()校验流中是否所有元素能够匹配给定的谓词 终端操作
* noneMatch()校验流中没有元素能够匹配给定的谓词 终端操作
上面三个操作都用到了所谓的短路：短路求值——有些操作不需要处理整个流就能得到结果。
* findAny()返回当前流中任意元素
* findFirst()返回当前流中第一个元素

## 5.4 归约
归约：将流中所有的元素反复结合起来，经过复杂的逻辑得到一个值
* reduce()表达更为复杂的查询 

reduce的使用：
1.reduce接受两个参数，第一个是初始值，第二个是将两个元素结合起来的行为
2.接受一个参数，即对两个元素的行为

* 流的操作分为两种：有状态和无状态。区别在于操作是否需要内部存储什么数据。
并行的使用有状态的操作是危险的

## 5.6 数值流
Stream API提供了原始类型流特化，专门支持处理数值流的方法。

### 5.6.1 原始类型流特化
Java8 引入了三个原始类型特化流来处理数值流：IntStream、DoubleStream和LongStream，分别将流中的元素特化为int、long和double，从而避免了暗含的装箱的成本。每个接口都提供了常用数值归约的新方法，比如对数值流求和的sum，找到最大元素的max。

将流转化为特化版本的常用方法是mapToInt、mapToDouble和mapToLong，这些方法和map()方法的工作方式一样，只是他们返回的是一个特化流，而不是 Stream。

要将特化流转为一般流，可以调用boxed方法，即声明装箱为一般流。

对特化流求和，若流中没有元素，则返回0正常。但对于max()方法来说，若返回为0无法区分流中没有元素还是流中最大值为0.因此这个地方可以使用OptionalInt类，即Optional原始类型特化版本，这样就可以分辨流中没有元素还是流中最大为0。min方法，double、long类型同理。

### 5.6.2 数值范围
Java8引入了两个可以用于IntStream和LongStream的方法，帮助生成范围内的特化流。range()和rangeClosed()。这两个方法都是第一个参数是起始值，第二个参数是结束值，不同的就是一个包含结束值一个不包含结束值。
```
IntStream.range(1,10).forEach((a)->System.out.println(a)); //输出1到9
IntStream.rangeClosed(1,10).forEach((a)->System.out.println(a)); //输出1到10
```

## 5.7 创建流

### 5.7.1 由值创建流
```
Stream<String> stream = Stream.of("java 8","test","end"); //通过Stream.of方法生成一个流 
Stream<String> emptyStream = Stream.empty(); //获得一个空的流
```

### 5.7.2 由数组创建流
```
int[] numbers = {2,3,4,5};
int sum = Arrays.stream(numbers).sum(); //生成一个IntStream 并求和
```

### 5.7.3 由文件生成流
```
try {
    Stream<String> lines = Files.lines(Paths.get("test.txt"), Charset.defaultCharset()); //通过java.nio.file.Files的方法从一个文件中获取一个字符串流
    long uniqueWords = lines.flatMap((a)->Arrays.stream(a.split(" "))).distinct().count();//计算不同的单词的数量
} catch (IOException e) { //错误处理
    e.printStackTrace();
}
```

### 5.7.4 由函数生成流：创建无限流
Stream API提供了两个静态方法来从函数生成流：Stream.iterate和Stream.generate.这两个操作可以创建所谓的无限流：不像从固定集合创建的流那样有固定大小的流。
 * 1.Stream.iterate 迭代
 ```
 Stream.iterate(0,n->n+2) //第一个参数为初始值 第二个参数为迭代的行为
                .limit(10)
                .forEach(a->System.out.println(a));
 ```
 
 * 2.Stream.generate 生成
 ```
 Stream.generate(Math::random) //参数为Supplier 即（）-> T的一个行为
                .limit(5)
                .forEach(a->System.out.println(a));
 ```
 不能直接对无限流进行排序和归约，因为所有元素都需要处理，这永远也完不成。


# 第六章、用流收集数据
流可以用类似于数据库的操作帮助你处理集合，可以把流看作花哨又懒惰对数据集迭代器。他们支持两种类型的操作：中间操作和终端操作。中间操作可以链接起来，将一个流转换成另一个流，而终端操作会消耗流产生一个最终结果。

## 收集器源码功能解析

## 6.2 归约和汇总
但凡要把流中的所有项目合并成一个结果时就可以用收集器。
* 预定义收集器：从Collectors类提供的工厂方法创建的收集器。
### 6.2.1 查找流中的最大值和最小值
```
Collectors.maxBy() //获取最大值的收集器
Collectors.minBy() //获取最小值的收集器
```
使用方式：
```
List<Dish> dishes = new ArrayList<>();
Optional<Dish> optionalDishMax = dishes.stream()
        .collect(Collectors.maxBy(Comparator.comparing(Dish::getCalories))); //maxBy的参数为一个comparator

Optional<Dish> optionalDishMin = dishes.stream()
        .collect(Collectors.minBy(Comparator.comparing(Dish::getCalories))); //minBy的参数为一个comparator
```
### 6.2.2 汇总
```
Collectors.summingInt() //接受一个把对象映射为求和所需的int的函数
```
使用方式：
```
int allCalories = dishes.stream()
                .collect(Collectors.summingInt(Dish::getCalories));
System.out.println(allCalories);
```
还有summingLong和summingDouble，作用一样只是返回类型不同。

汇总不只是求和还可以计算平均数：Collectors.averagintInt()
```
//返回值为double averagingDouble和averagingLong的返回值都是double
double averageCalories = dishes.stream()
                .collect(Collectors.averagingInt(Dish::getCalories)); 
```
Collectors.summarizingInt():一次操作就返回菜单中元素的个数、并得到总和、平均值、最大值和最小值。
```
IntSummaryStatistics intSummaryStatistics = dishes.stream()
                .collect(Collectors.summarizingInt(Dish::getCalories));
double average = intSummaryStatistics.getAverage();//平均值
long count = intSummaryStatistics.getCount();//数量
int max = intSummaryStatistics.getMax();//最大值
int min = intSummaryStatistics.getMin();//最小值
long sum = intSummaryStatistics.getSum();//总和
```
### 6.2.3 连接字符串
Collectors.join()：会把流中每个字符串连接起来。内部使用了StringBuilder将字符串拼接起来.
```
String menu = dishes.stream()
                .map(Dish::getName)
                .collect(Collectors.joining());//字符串拼接方式1：简单的拼接

String menu1 = dishes.stream()
        .map(Dish::getName)
        .collect(Collectors.joining("，"));//字符串拼接方式2，中间加间隔符的拼接

String menu2 = dishes.stream()
        .map(Dish::getName)
        .collect(Collectors.joining(",","(",")"));//字符串拼接方式3：加开头、结尾和间隔符的拼接
```

## 6.3 分组
分组：根据一个或多个属性对集合中的项目进行分组。
Collectors.groupingBy()来实现分组
```
//这里给groupingBy方法传递一个function，这个function称为分类函数，groupingBy方法用它来把流中的元素进行分组。分组操作的结果是个map。
Map<String,List<Dish>> map = dishes.stream()
                .collect(Collectors.groupingBy(Dish::getType));//分类函数返回值为key，流中的元素为value中的值
```

### 6.3.1 多级分组
实现多级分组，可以使用groupingBy进行嵌套生成Map<?,Map<?,List>>这种类型
```
//这里第一个groupingBy方法有两个参数，第一个是分组的key，第二个是collector。 
//可以理解为按照第一个参数将流中的元素分组，之后使用第二个参数将每个组里的元素分别归约起来 这种使用方式也可以用用于在分组中收集数据
Map<String, Map<String, List<Dish>>> map1 = dishes.stream()
        .collect(Collectors.groupingBy(Dish::getType,Collectors.groupingBy(Dish::getName)));
```

### 6.3.2 按子组收集数据
可以采用6.3.1中那种groupingBy的使用方式收集数据。
Collectors.collectingAndThen():可以将收集器的结果转换为另一种类型。
使用方式：
```
//可以看作将第二个参数的方法拼接在第一个参数行为之后
Map<String,Dish> map2 = dishes.stream()
        .collect(Collectors.groupingBy(Dish::getName,
                Collectors.collectingAndThen(Collectors.maxBy(Comparator.comparing(Dish::getCalories))
                        ,Optional::get)));
```

## 6.4 分区
分区是分组的特殊情况：由一个谓词（返回值为boolean的函数）作为分类函数，意味着分组的key是boolean，于是最多可以分为两组。

Collectors.partitioningBy():分区函数，里面的参数第一个为返回值为true/false的分类函数，第二个为collector，与groupingBy的使用方式类似，第二个参数不是必须的。

### 6.4.1 分区的优势
* 保留了分区函数返回true和false的两套流元素列表。

## 6.5 收集器接口
Collector源码解读：
```
// T是流中要收集的元素的类型
// A是累加器的类型，累加器是收集过程中用于累计部分结果的对象
// R是收集操作得到的对象（通常是集合）
public interface Collector<T, A, R> {
    /**
    *返回一个建立新的结果容器的Supplier方法：（）->返回一个容器
    */
    Supplier<A> supplier();

    /**
    *返回一个将元素添加到结果容器的方法：（a,b）->返回为void
    */
    BiConsumer<A, T> accumulator();
    
    /**
    *返回合并两个结果容器的方法：（a,b）-)返回一个c。定义了流并行归约的时候，归约的结果如何合并。
    */
    BinaryOperator<A> combiner();
    
    /**
    *返回一个对结果容器应用最终转换的方法：（a）->返回一个b
    */
    Function<A, R> finisher();
    
    /**
    *返回了一系列特征，也就是一个提示列表，告诉collect方法在执行归约操作的时候可以应用哪些优化（如并行）
    */
    Set<Characteristics> characteristics();

    enum Characteristics {
        /**
         * accumulator函数可以从多个线程同时调用，且该收集器可以并行归约流。
         *如果收集器没有标记为UNORDERED,那它仅在用于无序数据源时才可以并行归约
         */
        CONCURRENT,

        /**
         * 归约结果不受流中项目的遍历和累积顺序的影响
         */
        UNORDERED,

        /**
         * 这表明finisher方法返回的函数是一个恒等函数，可以跳过。
         */
        IDENTITY_FINISH
    }
}
```
### 6.5.2全部融合到一起
Collections.emptyList():返回一个空列表，这个空列表为单例，不允许操作。Java API提供的收集器在需要返回空列表时返回的就是这个单例。

# 第七章、并行数据处理与性能
## 7.1 并行流
* 并行流：就是一个把内容分成多个数据块，并用不同的线程分别处理每个数据块的流。这样一来就可以自动把给定操作的工作负荷分配给多核处理器的所有内核。

### 7.1.1 将顺序流转换为并行流
* 对顺序流调用parallel方法将顺序流变为并行流
* 对并行流调用sequential方法将并行流变为顺序流

最后一次调用parallel或sequential会影响整个流水线，所以不能随意对切换。

可以通过java.util.concurrent.ForkJoinPool.common.parallelism来改变并行流线程池对大小。（一般不用修改）

### 7.1.2 测试流性能
* iterate这个函数每次都要依赖前一次对结果，所以很难分成多个独立的块来并行执行。可以使用rangeClosed，出来的直接是已经拆箱了的流，而且不会依赖上一次的结果。

### 7.1.4 高效使用并行流
是否有必要使用并行流的一些建议：
* 1.如果有疑问，就测试。
* 2.留意装箱，必要的时候可以使用原始类型流避免装箱拆箱的开销。
* 3.有些操作本来在并行流上的性能就比顺序流差。特别是limit和findFirst这些依赖元素顺序的操作。
* 4.还要考虑流的操作流水线的总计算成本。一个元素通过流水线的大致处理成本较高时，使用并行流可能更高效。
* 5.对于较小的数据量，并行不是个好选择。
* 6.要考虑流背后的数据结构是否易于分解。如：ArrayList的拆分效率比LinkedList高得多。
* 7.还要考虑合并代价。

## 7.2 分支/合并框架
分支/合并框架的目的是以递归方式将可以并行执行的任务拆分成更小的任务，然后将子任务的结果合并生成整体结果。他是ExecutorService接口的一个实现，他把子任务分配给ForkJoinPool中的工作线程。是并行流背后使用的基础架构。

### 7.2.1 使用RecursiveTask
要把任务提交到ForkJoinPool，创建RecursiveTask<R>的一个子类，R是返回类型。要定义他，只需要实现他的唯一的抽象方法：
```
protected abstract R compute(); 
```
    
### 7.2.3 工作窃取
在创建小任务的过程中，一般来说分出大量的小任务是一个好的选择，这是因为理想情况下，应该让每个任务都用完全相同的时间完成，让所有的CPU内核都同样繁忙。不幸的是，实际上每个子任务花费的时间可能天差地别，要么是因为划分策略效率低，要么是磁盘访问慢等其他原因。分支合并框架用一种称为工作窃取的技术来解决这个问题。

工作窃取：即线程池中的每个线程都会维护一个存储分配给他的任务的双向链式队列。某个线程早早完成任务时，就随机选一个线程，从它队列的尾部“偷”走一个任务执行，这个过程一直持续下去直到所有的任务都执行完毕。

## 7.3 Spliterator
Java8引入的接口，为了并行执行时拆分流设计。Java8为集合框架中包含的所有数据结构提供了一个默认的Spliterator实现。
```
T 是遍历的元素类型
public interface Spliterator<T> {
    boolean tryAdvance(Consumer<? super T> action);//类似于普通的iterator

    Spliterator<T> trySplit();//专为Spliterator设计，可以把一些元素划分出去给第二个Spliterator接口（通过该方法返回），让他们并行处理。

    long estimateSize();//估计还剩多少元素需要遍历

    int characteristics();//返回Spliterator本身特性集的编码
```

# 第八章、重构、测试和调整
### 8.1.1 改善代码的可读性
可读性即别人理解这段代码的难易程度

推荐的三种简单的重构：
* 1.用lambda表达式取代匿名类
* 2.用方法引用重构lambda表达式
* 3.用stream API重构命令式的数据处理

### 8.1.2 从匿名类到lambda表达式的转换
* 匿名类和lambda表达式中的this和super的含义是不同的。匿名类中的this表示的是类自身，lambda表达式中的this表示的是包含类。
* 匿名类可以屏蔽包含类的变量，lambda表达式不行。
* 在涉及重载的上下文中，用lambda表达式可能会更加晦涩。

### 8.1.5 增加代码的灵活性
两种通用模式来重构代码：有条件的延迟执行和环绕执行
#### 有条件的延迟执行


# 第九章、默认方法
### 默认方法中冲突的解决规则
如果一个类实现的两个接口中，包含了两个相同签名的默认方法，那么这个类就会因为不知道选择哪一个实现而出错。这种情况下有三种规则来判断类具体选择的实现：
* 1.类中的方法优先级最高，类或父类中声明的方法的优先级高于任何声明为默认方法的优先级。
* 2.如果无法一句第一条进行判断，那么子接口的优先级更高：函数签名相同时，优先选择拥有最具体实现的默认方法的接口，即如果B继承了A，那么B就比A更加具体。
* 3.最后如果还是无法判断，继承了多个接口的类必须通过显式覆盖和调用期望的方法，显式的选择使用哪一个默认方法的实现。

# 第十章、用Optional取代null
### 10.1.2 null带来的各种问题
* 1.是错误之源，是目前Java程序开发中最典型的异常。
* 2.它会让你的代码膨胀，让你的代码充斥着深度嵌套的null检查，代码的可读性很糟糕
* 3.本身是毫无意义的，它代表的是在静态类型语言中以一种错误的方式对缺失变量值的建模。
* 4.破坏了Java的哲学，Java一直试图避免让程序员意识到指针的存在，唯一的例外是：null指针
* 5.在Java类型系统上开了个口子，null不属于任何类型，意味着它可以被赋值给任意引用类型的变量，这会导致这个变量传递到系统的另一个部分后，无法获知这个null变量最初的赋值到底是什么类型。


### 10.2 Optional类入门
在你的代码中始终如一地使用Optional，能非常清晰的界定出变量值的缺失是结构上的问题，还是算法上的问题，抑或是数据中的问题。另外，引入Optional类的意图并非是要消除每一个null引用。与此相反，它的目标是帮助你更好的设计出普适的API，让程序员看到方法签名就能知道它是否接受一个Optional的值。这种强制会让你更积极的将变量从Optional中解包出来，直面缺失的变量值。null的检查只会掩盖问题。

###  Optional的使用：
* Optional对象的创建
```
//声明一个空的Optional
Optional<Apple> optionalApple = Optional.empty();

//依据一个非空值创建Optional
Optional<Apple> optionalApple = Optional.of(new Apple());

//可接受null创建的Optional 如果为null源码还是返回的empty（）方法的返回值
Optional<Apple> optionalApple = Optional.ofNullable(null);
```

* 源码解读：
```
public final class Optional<T> {
    /**
     * 单例的内部为空的Optional对象
     */
    private static final Optional<?> EMPTY = new Optional<>();

    /**
     * 如果不为空，optional封装的对象
     */
    private final T value;

    /**
     * 创建一个空的optional的构造方法，因为空的optional是单例，所以这里声明为private
     */
    private Optional() {
        this.value = null;
    }

    /**
     * 转换成相应对象并返回静态的空的Optional对象
     */
    public static<T> Optional<T> empty() {
        @SuppressWarnings("unchecked")
        Optional<T> t = (Optional<T>) EMPTY;
        return t;
    }

    /**
     * 不为空的Optional的构造方法
     */
    private Optional(T value) {
        this.value = Objects.requireNonNull(value);
    }

    /**
     * 依据一个不为空的对象创建的Optional对象的静态工厂方法
     */
    public static <T> Optional<T> of(T value) {
        return new Optional<>(value);
    }

    /**
     * 接受一个可以为空的对象，若为空则调用empty静态方法并返回，否则调用of静态方法返回
     */
    public static <T> Optional<T> ofNullable(T value) {
        return value == null ? empty() : of(value);
    }

    /**
     * 若Optional为空 跑出NoSuchElementException异常，否则返回value。这个是最简单却最不安全的方法。
     */
    public T get() {
        if (value == null) {
            throw new NoSuchElementException("No value present");
        }
        return value;
    }

    /**
     * 若Optional为空则返回false，否则返回true
     */
    public boolean isPresent() {
        return value != null;
    }

    /**
     * 接受一个Consumer的参数，该参数中唯一的方法为void accept(T t)，这里的方法即若Optional不为空，则对里面的对象做操作，无需返回值。
     */
    public void ifPresent(Consumer<? super T> consumer) {
        if (value != null)
            consumer.accept(value);
    }

    /**
     * 这个方法接受一个Predicate的函数式接口，其中的方法为boolean test(T t);这里表示若Optional为空则返回当前的Optional对象，否则对value调用test方法，若为true则返回当前Optional对象，否则返回empty（）方法的返回值。这里是通过predicate的行为对Optional的一个筛选方法。
     */
    public Optional<T> filter(Predicate<? super T> predicate) {
        Objects.requireNonNull(predicate);
        if (!isPresent())
            return this;
        else
            return predicate.test(value) ? this : empty();
    }

    /**
     * 这个方法接受一个Function类型的函数式接口，这个接口里的方法为R apply(T t); 这里的意思为如果该optional为空则返回empty，否则，会对该Optional对象调用apply方法，并将得到的返回值用Optional做一层封装。这里就是通过Function的行为对optional对象的映射方法，获得一个新的Optional对象。
     */
    public<U> Optional<U> map(Function<? super T, ? extends U> mapper) {
        Objects.requireNonNull(mapper);
        if (!isPresent())
            return empty();
        else {
            return Optional.ofNullable(mapper.apply(value));
        }
    }

    /**
     * 这个方法和上面的方法相比不同之处在于：1.强制限制了Function的返回值为Optional类型  2.不会对返回值再次使用Optional进行封装 这两种方法可以分别在返回值不为Optional和返回值为optional时使用，此时他们最终的结果相同。
     */
    public<U> Optional<U> flatMap(Function<? super T, Optional<U>> mapper) {
        Objects.requireNonNull(mapper);
        if (!isPresent())
            return empty();
        else {
            return Objects.requireNonNull(mapper.apply(value));
        }
    }

    /**
     * 若Optional不为空则返回value 否则返回参数other
     */
    public T orElse(T other) {
        return value != null ? value : other;
    }

    /**
     * 和上面的方法差不多，只不过other参数变成了表示一个新建对象的行为的函数式接口
     */
    public T orElseGet(Supplier<? extends T> other) {
        return value != null ? value : other.get();
    }

    /**
     * 若当前optional不为空则返回value，否则抛出一个异常，该异常来源于函数式接口supplier
     */
    public <X extends Throwable> T orElseThrow(Supplier<? extends X> exceptionSupplier) throws X {
        if (value != null) {
            return value;
        } else {
            throw exceptionSupplier.get();
        }
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) {
            return true;
        }

        if (!(obj instanceof Optional)) {
            return false;
        }

        Optional<?> other = (Optional<?>) obj;
        return Objects.equals(value, other.value);
    }

    
    @Override
    public int hashCode() {
        return Objects.hashCode(value);
    }

    
    @Override
    public String toString() {
        return value != null
            ? String.format("Optional[%s]", value)
            : "Optional.empty";
    }
```
Optional提供了基础类型的OptionalInt、OptionalLong等类型，但是不推荐使用，因为基础类型等Optional不支持map、flatmap和filter方法。


# 第十一章、CompletableFuture组合式异步编程
## 11.1 Future接口
要使用Future（返回值），通常只需要将耗时的操作封装在一个Callable（任务内容）中，再将它提交给ExecutorService（执行者）,就可以了。








