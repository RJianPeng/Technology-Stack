* [第五章 使用流](#第五章使用流)



# 第五章、使用流
## 5.6 数值流
Stream API提供了原始类型流特化，专门支持处理数值流的方法。

### 5.6.1 原始类型流特化
Java8 引入了三个原始类型特化流来处理数值流：IntStream、DoubleStream和LongStream，分别将流中的元素特化为int、long和double，从而避免了暗含的装箱的成本。每个接口都提供了常用数值归约的新方法，比如对数值流求和对sum，找到最大元素对max。

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
