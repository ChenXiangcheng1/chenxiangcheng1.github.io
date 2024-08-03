---
title: Stream API
date: 2023-10-11
update: 2023-10-11

tags:
categories:
  - Java
  - 类库
  - util
keywords: Java
description: 关于java.util.stream
top_img:
cover:
---



# Stream API

`java.util.stream` 提供了集合的新的函数式的操作方式：流在管道中传输， 在管道的节点上进行处理，过滤filter、映射map、排序sorted、归约reduction等中间操作和终端操作。

Stream 代表的是一种对 Java 对象**序列的运算**和表达的高阶抽象。将要处理的元素集合看作一种流。可以并行操作数据子集，然后组合中间结果以获得最终正确的答案。

应用场景：适合处理集合、序列，提供了一种简单的并行化方式(reduce 无需额外的同步)

[API 文档](https://devdocs.io/openjdk~17/java.base/java/util/stream/package-summary)

可选特征：

* 是否顺序(sequential)，是否在**单线程**中按**顺序执行**操作
* 是否有序(ordered)，是否操作的**顺序**与元素的顺序**一致**



## 对比流与集合

流与集合的不同：

* 没有存储空间。流不是存储元素的数据结构，相反它通过计算操作的管道来传送元素。

* Functional。不会修改源，而是生成新 Stream
  流操作通常是基于不可变性的原则，它们不会直接修改源数据，而是创建新的对象来表示操作的结果。这种方式确保了线程安全性和数据的不变性。
* Laziness-seeking 延迟提供。中间操作懒执行，需要终端操作触发流的处理
* 可能是无界的。短路操作(当已经能够确定整个表达式的结果时，就不再继续计算剩余的表达式部分)使无限流能够在有限时间内完成

* 单一消费，在流的生命周期中，流的元素仅被访问一次。



| 对比流与io | java.io                  | java.util.stream           |
| :--------- | :----------------------- | -------------------------- |
| 存储       | 顺序读写的`byte`或`char` | 顺序输出的任意Java对象实例 |
| 用途       | 序列化至文件或网络       | 内存计算／业务逻辑         |

| 对比流与集合 | java.util.List           | java.util.stream         |
| :----------- | :----------------------- | ------------------------ |
| 元素         | 已分配并存储在内存       | 可能未分配，**实时计算** |
| 用途         | 操作一组已存在的Java对象 | 惰性计算                 |



## 基础用法

创建一个Stream，然后做若干次转换，最后调用一个求值方法获取真正计算的结果：

```java
int result = createNaturalStream() // 创建Stream
             .filter(n -> n % 2 == 0) // 任意个转换
             .map(n -> n * n) // 任意个转换
             .limit(100) // 任意个转换
             .sum(); // 最终计算结果
```



## 生命周期

* 创建流：**从数据源中获取流对象**，`parallelStream()`、`Stream()`
  数据源：集合、数组、generator 函数、I/O 通道等

* 中间操作 - 中间操作返回新流，并且总是懒惰的，即管道的终端操作被执行后才开始遍历管道源。
  `filter()`、`map()`、`distinct()`、`sorted()`
  * 有状态操作
    在处理新元素时合并先前看到的元素状态
    在产生结果之前需要处理整个输入
  * 无状态操作
  * 短路操作 - 提供无限输入时，产生有限流
* 终端操作 - 终端操作被执行后，管道被视作已消费的，不能再被使用。

流操作分为中间操作和终端操作，组合起来形成流管道。
**流管道** = 源 + 中间操作 + 终端操作



## BaseStream<T, S extends BaseStream<T, S>>

流的基础接口

### parallel 并行

`parallel():S` 返回一个等价的并行流，可能会返会自身

示例：

```java
Stream<String> s = ...
String[] result = s.parallel() // 变成一个可以并行处理的Stream
                   .sorted() // 可以进行并行排序
                   .toArray(String[]::new);
```



### unordered

`unordered():S` 返回一个等价的无序流，可能会返会自身
能提高一些有状态操作、终端操作的并行性能



### Stream<T>

接口，继承自 BaseStream<T, Stream<T>>



### 基本类型流

因为Java的范型不支持基本类型，所以我们无法用Stream<int>这样的类型，会发生编译错误。为了保存int，只能使用Stream<Integer>，但这样会产生频繁的装箱、拆箱操作。为了提高效率，Java标准库提供了**`IntStream`**、**`LongStream`**和**`DoubleStream`**这三种使用基本类型的Stream，它们的使用方法和范型Stream<T>没有大的区别，设计这三个Stream的目的是提高运行效率。

```java
// 将int[]数组变为IntStream:
IntStream is = Arrays.stream(new int[] { 1, 2, 3 });
// 将Stream<String>转换为LongStream:
LongStream ls = List.of("1", "2", "3").stream().mapToLong(Long::parseLong);
```



针对 `IntStream`、`LongStream`和`DoubleStream`，还额外提供了以下中间方法：

#### boxed

`boxed():Stream<Integer>` 每个元素装箱为 Integer

#### range

`IntStream.range(int,int):IntStream` 返回顺序有序 IntStream，数据源 [startInclusive, endExclusive)



#### ===



针对 `IntStream`、`LongStream`和`DoubleStream`，还额外提供了以下终端方法：

#### sum

`sum():int`：返回流中所有元素总和

#### average

`average():OptionalDouble`：返回一个 OptionalDouble，描述流中所有元素的算数平均值。如果流为空则返回空 Optional

#### toArray

`toArray():int[]` 返回一个数组，包含流中的元素



## ===创建流===

指定不同的数据源

### Stream

`Stream.of(T...):Stream<T>` 返回**指定元素**的顺序的新流

这种方式没有实质性用途，但测试的时候很方便

示例：

```java
Stream<String> stream = Stream.of("A", "B", "C", "D");
// forEach()方法相当于内部循环调用，
// 可传入符合Consumer接口的void accept(T t)的方法引用：
stream.forEach(System.out::println);  // >> A\nB\nC\nD\n
```



`Stream.generate(Supplier<? extends T>):Stream<T>` 返回无限的生成有序处理无序的流(Returns an **infinite sequential unordered stream**)
unordered 是排序约束，无序
sequential 是执行约束，串行

示例：

```java
class NatualSupplier implements Supplier<Integer> {
    int n = 0;
    public Integer get() {
        n++;
        return n;
}	}

Stream<Integer> natual = Stream.generate(new NatualSupplier());
// 注意：无限序列必须先变成有限序列再打印，直接调用forEach()或者count()这些最终求值操作，会进入死循环
natual.limit(20).forEach(System.out::println);
```



### Arrays

`Arrays.stream(T[],int,int):Stream<T>` 返回以**指定数组**作为源的顺序流

`Arrays.stream(int[],int,int):IntStream` 返回以**指定数组**作为源的顺序流



### Collection

`stream(): Stream<E>` 返回以**指定集合**作为源的顺序流
`parallelStream():Stream<E>` 返回以**指定集合**作为源的可能并行流



### Files

示例：

`Files.lines(Path):Stream<String>` 以流的形式读取文件中的所有行，每个元素代表文件的一行内容
此方法对于按行遍历文本文件十分有用

```java
try (Stream<String> lines = Files.lines(Paths.get("/path/to/file.txt"))) {
    ...
}
```



### Pattern

正则表达式 Pattern 对象

`splitAsStream(CharSequence):Stream<String>` 从匹配该模式的给定序列中创建流，即把一个长字符串分割成 Stream 序列而不是数组

```java
Pattern p = Pattern.compile("\\s+");
Stream<String> s = p.splitAsStream("The quick brown fox jumps over the lazy dog");
s.forEach(System.out::println);
```



## ===中间操作===

[Lambda 笔记#java.util.function](./Lambda.md#java.util.function)



### 无状态

#### map 映射

`map(Function<? super T, ? extends R>):Stream<R>` 返回一个流，该流的元素由给定的 function 的 apply 函数映射结果组成

示例：

```java
List.of("  Apple ", " pear ", " ORANGE", " BaNaNa ")
        .stream()
        .map(String::trim) // 去空格
        .map(String::toLowerCase) // 变小写
        .forEach(System.out::println); // 打印
```



#### flatMap

`flatMap(Function<? super T, ? extends Stream<? extends R>>):Stream<R>` 返回一个流，由提供的 function 函数映射每一个元素。flatMap 操作的作用是将流元素进行一到多的转换，然后 flattening 扁平化结果元素到新流。

示例：

把 Stream<List<Integer>> 转换为 Stream<Integer>

```java
Stream<List<Integer>> s = Stream.of(
        Arrays.asList(1, 2, 3),
        Arrays.asList(4, 5, 6),
        Arrays.asList(7, 8, 9));
Stream<Integer> i = s.flatMap(List::stream);
```



#### filter 过滤

`filter(Predicate<? super T>):Stream<T>` 返回一个流，该流的元素由匹配给定 predicate 谓词的元素组成

示例：

从一组给定的 LocalDate 中过滤掉工作日，以便得到休息日

```java
class LocalDateSupplier implements Supplier<LocalDate> {
    LocalDate start = LocalDate.of(2020, 1, 1);
    int n = -1;
    public LocalDate get() {
        n++;
        return start.plusDays(n);
}	}

Stream.generate(new LocalDateSupplier())
        .limit(31)
        .filter(ldt -> ldt.getDayOfWeek() == DayOfWeek.SATURDAY || ldt.getDayOfWeek() == DayOfWeek.SUNDAY)
        .forEach(System.out::println);
```

```bash
2020-01-04
2020-01-05
2020-01-11
2020-01-12
2020-01-18
2020-01-19
2020-01-25
2020-01-26
```



#### peek(常用)

`peek(Consumer<? super T>):Stream<T>` 返回一个流，当每个元素被消费从结果流中，额外执行提供的 action 行为

```java
Stream<T> peek(Consumer<? super T> action);
```

示例：

```java
Stream.of("one", "two", "three", "four")
    .filter(e -> e.length() > 3)
    .peek(e -> System.out.println("Filtered value: " + e))
    .map(String::toUpperCase)
    .peek(e -> System.out.println("Mapped value: " + e))
    .collect(Collectors.toList());
```



### 有状态

#### sorted 排序

`sorted():Stream<T>` 返回一个流，按自然顺序排列。如果元素不可比较(没有实现 Comparable 接口)则抛出异常ClassCastException。对于有序流(流中元素顺序是确定的)排序稳定，对于无序流排序不稳定。

示例：

```java
List<String> list = List.of("Orange", "apple", "Banana")
    .stream()
    .sorted()
    .collect(Collectors.toList());
System.out.println(list);  // >> [Banana, Orange, apple]
```



`sorted(Comparator<? super T>):Stream<T>` 返回一个流，根据提供的 Comparator 排序。对于有序流(流中元素顺序是确定的)排序稳定，对于无序流排序不稳定。

示例：

```java
List<String> list = List.of("Orange", "apple", "Banana")
    .stream()
    .sorted(String::compareToIgnoreCase)
    .collect(Collectors.toList());
```



#### distinct 去重(好用)

`distinct():Stream<T>` 返回一个流，由不同的元素组成
实现 List 去重，使用 Stream 的 distinct 比转换为 Set 更好，因为 distinct 是稳定的，Set 是无序的。

示例：

```java
List.of("A", "B", "A", "C", "B", "D")
    .stream()
    .distinct()
    .collect(Collectors.toList()); // [A, B, C, D]
```



#### concat

`concat(Stream<? extends T>, Stream<? extends T>):Stream<T>` 创建一个懒的合并流。如果两个流都是有序的则结果是有序流，如果一个并行的则结果是并行流。当结果流关闭时将调用两个输入流的关闭处理器。

示例：

```java
Stream<String> s1 = List.of("A", "B", "C").stream();
Stream<String> s2 = List.of("D", "E").stream();
Stream<String> s = Stream.concat(s1, s2);  // 合并
System.out.println(s.collect(Collectors.toList())); // [A, B, C, D, E]
```



#### skip 跳过

`skip(long n):Stream<T>` 丢弃流的前 n 个元素，返回由剩余元素组成的流。如果此流包含的元素少于n个，则将返回一个空流。将无限流转换成有限流



#### limit 截断

`limit(long n):Stream<T>` 返回一个流，由源流中元素组成，截取长度不超过 long 的部分。
限流转换成有限流

示例：

```java
List.of("A", "B", "C", "D", "E", "F")
    .stream()
    .skip(2) // 跳过A, B
    .limit(3) // 截取C, D, E
    .collect(Collectors.toList()); // [C, D, E]
```



## ===终端操作===

### mapToInt

`mapToInt(ToIntFunction<? super T>):IntStream` 返回 IntStream，该 IntStream 流由应用给定的 ToIntFunction 函数对此流的元素的映射结果组成

```java
IntStream mapToInt(ToIntFunction<? super T> mapper);
```

示例：

```java
int sumOfWeights = widgets.parallelStream()
                       .filter(b -> b.getColor() == RED)
                       .mapToInt(b -> b.getWeight())
                       .sum();
```



### findFirst

`findFirst():Optional<T>` 返回 Optional，描述此流的第一个元素。如果流为空则为空Optional，如果流没有偶遇顺序则任何元素可能被返回



### findAny

`findAny():Optional<T>` 返回一个 Optional，描述流中某个元素。如果流为空则返回空 Optional

示例：

```java
public static CouponStatus toType(int value) {
    return Stream.of(CouponStatus.values())
            .filter(c -> c.value == value)
            .findAny()
            .orElse(null);
}
```



### allMatch 匹配

`allMatch(Predicate<? super T>):boolean` 返回是否此流的所有元素与提供的谓词匹配
如果已能得到结果，则短路
如果流为空则返回 true，predicate 谓词不被计算



### anyMatch

`anyMatch(Predicate<? super T>):boolean` 返回是否流中存在一个元素与提供的谓词匹配
如果已能得到结果，则短路
如果流为空则返回 false，predicate 谓词不被计算



### reduce 归约(好用)

`reduce(T, BinaryOperator<T>):T` 使用identity初始值和associative累加函数对流的元素执行 reduction 归约/聚合操作，返回归约值。
reduce 操作可以更优雅地并行化，而不需要额外的同步(synchronized)大大降低了数据竞争的风险。

```java
// 等价操作：
T result = identity;
for (T element : this stream)
    result = accumulator.apply(result, element)
return result;
```

示例1：

```java
int sum = Stream.of(1, 2, 3, 4, 5, 6, 7, 8, 9).reduce(0, (acc, n) -> acc + n);
System.out.println(sum); // 45
```

示例2：

何将配置文件的每一行配置通过`map()`和`reduce()`操作聚合成一个`Map<String, String>`：

```java
// 按行读取配置文件:
List<String> props = List.of("profile=native", "debug=true", "logging=warn", "interval=500");
Map<String, String> map = props.stream()
        // 把k=v转换为Map[k]=v:
        .map(kv -> {
            String[] ss = kv.split("\\=", 2);
            return Map.of(ss[0], ss[1]);
        })
        // 把所有Map聚合到一个Map:
        .reduce(new HashMap<String, String>(), (m, kv) -> {
            m.putAll(kv);
            return m;
        });
// 打印结果:
map.forEach((k, v) -> {
    System.out.println(k + " = " + v);
});
```

```打印
logging = warn
interval = 500
debug = true
profile = native
```



`reduce(BinaryOperator<T>):Optional<T>` 

示例：

```java
Optional<Integer> opt = stream.reduce((acc, n) -> acc + n);
if (opt.isPresent()) {
    System.out.println(opt.get());
}
```



#### count

`count():long` 返回该流中元素的个数
如果能从源直接计算出 count 总数，则不会执行流管道
这是 reduce 操作的特殊例子

示例：

```java
List<String> l = Arrays.asList("A", "B", "C", "D");
long count = l
    .stream()
    .peek(System.out::println)  // 该中间操作不会向流中注入或移除元素
    .count();  // 不会执行管道
```



#### max

`max(Comparator<? super T>):Optional<T>` 返回流中最大的元素，根据提供的 Comparator
这是 reduce 操作的特殊例子



#### min

`min(Comparator<? super T>):Optional<T>` 返回流中最小的元素，根据提供的 Comparator
这是 reduce 操作的特殊例子



### collect 收集(核心)

`collect(Collector<? super T, A, R>):R` 使用 Collector 收集器对流的元素执行 mutable reduction 可变归约/聚合操作
Collector 封装了被用作参数的 function 函数(描述 reduction 归约/聚合操作)，允许重用收集策略。

> 流操作通常是基于不可变性的原则，它们不会直接修改源数据，而是创建新的对象来表示操作的结果。这种方式确保了线程安全性和数据的不变性。

collect 是可变归约操作，将所需的结果收集到结果容器(Collection)中

示例1：

累积 String 到 List

```java
List<String> asList = stringStream.collect(Collectors.toList())；
```

示例2：

```java
Map<String, List<Person>> peopleByCity = personStream.collect(Collectors.groupingBy(Person::getCity));
```



`collect(Supplier<R>, BiConsumer<R, ? super T>, BiConsumer<R, R>):R` 

参数1 - Supplier 函数，用于构造可变结果容器的新实例。并行执行时可能被多次调用，每次返回刷新值。
参数2 - accumulator 累加器函数，用于折叠 fold 输入元素到结果容器，是一个关联、无干扰、无状态的函数。
参数3 - combiner 组合函数，合并结果容器的内容到另一个结果容器，是一个关联、无干扰、无状态的函数。

```java
<R> R collect(Supplier<R> supplier,
                  BiConsumer<R, ? super T> accumulator,
                  BiConsumer<R, R> combiner);
```

```java
// 等价操作
R result = supplier.get();
for (T element : this stream)
    accumulator.accept(result, element);
return result;
```

示例：

```java
List<String> strings = stream.map(Object::toString)
                          .collect(ArrayList::new, ArrayList::add, ArrayList::addAll);
```

```java
// 等价操作
List<String> strings = stream.map(Object::toString)
                          .collect(Collectors.toList());
```

示例：

```java
String concat = stringStream.collect(
    			StringBuilder::new,
    			StringBuilder::append,
    			StringBuilder::append)
    .toString();
```



### forEach(迭代)

`forEach(Consumer<? super T>):void` 执行一个操作，对流中的每个元素
对于并行流管道，对于任何给定的元素，操作可以在库选择的任何时间和任何线程中执行。因此，如果操作访问共享状态，就需要提供所需的同步机制，以确保线程安全。

示例1：

```java
Stream<String> s = ...
s.forEach(System.out::println);
```

示例2：

1 ArrayList 线程不安全
2 .forEach(s -> results.add(s)) 是一种有状态、有副作用(对返回结果之外的其他状态或资源产生了影响)的方式

```java
ArrayList<String> results = new ArrayList<>();
stream.filter(s -> pattern.matcher(s).matches())
	.forEach(s -> results.add(s));  // Unnecessary use of side-effects!
```

为了避免线程安全问题和副作用，可以使用更安全且适合并行化的归约操作 collect() 来替换该 forEach() 方法。

```java
List<String> results = stream.filter(s -> pattern.matcher(s).matches())
							.collect(Collectors.toList());  // No side-effects!
```



### toArray

`toArray():Object[]` 返回一个数组，包含流中的元素

`toArray(IntFunction<A[]>):A[]` 返回一个数组，包含流中的元素，使用提供的 generator function 方法分配
参数：IntFunction<A[]>，表示接受int值参数并生成结果的函数。根据所需类型和所需数组大小，生成一个新数组的方法，这可以用数组构造函数引用简介地表达。

示例1：

```java
List<String> list = List.of("Apple", "Banana", "Orange");
String[] array = list.stream().toArray(String[]::new);
```

示例2：

```java
Person[] men = people.stream()
    .filter(p -> p.getGender() == MALE)
    .toArray(Person[]::new);
```



## =======

## Collectors

Collector 收集器的实现，实现了各种有用的 reduce 归约/聚合操作

**Collector 收集器实例，表示一个收集器，通过 reduce 归约/聚合操作(BinaryOperator)，将元素累加到一个 collections 集合中**

```java
CollectorImpl(Supplier<A> supplier,
              BiConsumer<A, T> accumulator,
              BinaryOperator<A> combiner,
              Set<Characteristics> characteristics) {
    this(supplier, accumulator, combiner, castingIdentity(), characteristics);
}
```



### toList

`Collectors.toList():Collector<T, ?, List<T>>` 返回一个 Collector，该收集器将输入元素累积到一个新的 ArrayList 列表中。
不能保证返回的 List 的类型、可变性、可序列化性、线程安全。如果需要对返回的List进行更多控制，使用 toCollection

```java
public static <T> Collector<T, ?, List<T>> toList() {
    return new CollectorImpl<>(ArrayList::new, List::add,
                               (left, right) -> { left.addAll(right); return left; },
                               CH_ID);
}
```

示例：

将一组 String 先过滤掉空字符串，然后把非空字符串保存到 ArrayList 中

```java
Stream<String> stream = Stream.of("Apple", "", null, "Pear", "  ", "Orange");
List<String> list = stream.filter(s -> s != null && !s.isBlank())
    					.collect(Collectors.toList());
System.out.println(list);  // >> [Apple, Pear, Orange]
```



### toSet

`Collectors.toSet():Collector<T, ?, Set<T>>` 返回一个 Collector，该收集器将输入元素累积到一个新的 HashSet 集合中。
不能保证返回的 Set 的类型、可变性、可序列化性、线程安全。如果需要对返回的Set进行更多控制，使用toCollection

```java
public static <T> Collector<T, ?, Set<T>> toSet() {
    return new CollectorImpl<>(HashSet::new, Set::add,
                               (left, right) -> {
                                   if (left.size() < right.size()) {
                                       right.addAll(left); return right;
                                   } else {
                                       left.addAll(right); return left;
                                   }
                               },
                               CH_UNORDERED_ID);
}
```



### toMap

`Collectors.toMap(Function<? super T, ? extends K>, Function<? super T, ? extends U>):Collector<T, ?, Map<K,U>>` 返回一个 Collector ，该收集器将元素累计到 HashMap 中，HashMap 的键和值是所提供的 function 函数映射输入元素的结果。
如果映射的键可能重复，请改用 toMap(Function, Function, BinaryOperator) 
不能保证返回的 Map 的类型、可变性、可序列化性、线程安全。

```java
public static <T, K, U> Collector<T, ?, Map<K,U>> toMap(
    		Function<? super T, ? extends K> keyMapper,
    		Function<? super T, ? extends U> valueMapper) {
    return new CollectorImpl<>(HashMap::new,
                               uniqKeysMapAccumulator(keyMapper, valueMapper),
                               uniqKeysMapMerger(),
                               CH_ID);
}
```

示例：

指定两个 Function 函数，分别把元素映射为 HashMap 的 key 和 value

```java
Stream<String> stream = Stream.of("APPL:Apple", "MSFT:Microsoft");
Map<String, String> map = stream
        .collect(Collectors.toMap(
                s -> s.substring(0, s.indexOf(':')),  // 把元素s映射为key
                s -> s.substring(s.indexOf(':') + 1)));  // 把元素s映射为value
System.out.println(map);  // >> {MSFT=Microsoft, APPL=Apple}
```



### toCollection

`toCollection(Supplier<C>):Collector<T, ?, C>` 返回一个 Collector，该收集器按相遇顺序将输入元素累积到新的集合中。集合由提供的 collectionFactory 集合工厂创建。

```
public static <T, C extends Collection<T>> Collector<T, ?, C> toCollection(
				Supplier<C> collectionFactory) {
    return new CollectorImpl<>(collectionFactory, Collection<T>::add,
                               (r1, r2) -> { r1.addAll(r2); return r1; },
                               CH_ID);
}
```

示例：

```java
Set<String> set = people.stream()
   .map(Person::getName)
   .collect(Collectors.toCollection(TreeSet::new));
```



### partitioningBy

`partitioningBy(Predicate<? super T>):Collector<T, ?, Map<Boolean, List<T>>>` 返回一个 Collector ，该收集器根据 Predicate 谓词对输入元素进行分区，并将它们组织为 **Map<Boolean, List<T>>**。

不能保证返回的 Map 或 List 的类型、可变性、可序列化性、线程安全。

```java
public static <T> Collector<T, ?, Map<Boolean, List<T>>> partitioningBy(Predicate<? super T> predicate) {
    return partitioningBy(predicate, toList());
}
```

示例：

```java
 // Partition students into passing and failing
 Map<Boolean, List<Student>> passingFailing = students.stream()
   .collect(Collectors.partitioningBy(s -> s.getGrade() >= PASS_THRESHOLD));
```



### groupingBy

`groupingBy(Function<? super T, ? extends K>):Collector<T, ?, Map<K, List<T>>>` 返回一个 Collector ，该收集器实现了对 T 类型的输入元素的 group by 操作，根据分类 function 函数对元素分组，该收集器返回一个 Map

对于 parallel 并行流管道，请改用 groupingByConcurrent(Function, Collector)

参数：分类 function 函数将元素映射为 K 类型的键
返回：Collector生成 **Map<K, List<T>>** ，key 是将分类 function 函数映射输入元素后所产生的值，值对应的 value 是包含对应输入元素的 ArrayList

不能保证返回的 Map 或 List 的类型、可变性、可序列化性、线程安全。

```java
public static <T, K> Collector<T, ?, Map<K, List<T>>> groupingBy(
    		Function<? super T, ? extends K> classifier) {
    return groupingBy(classifier, toList());
}
```



`groupingBy(Function<? super T, ? extends K>, Collector<? super T, A, D>):Collector<T, ?, Map<K, D>>` 返回一个 Collector ，该收集器实现了对 T 类型的输入元素的 **group by 操作**，根据分类 function 函数对元素分组，然后使用指定的 downstream 下游收集器对给定 key 的关联 value 执行**reduction 操作**

参数：分类 function 函数将元素映射为 K 类型的键、下游 Collector 对 T 类型的元素进行操作 
返回：Collector生成 Map<K, D> 

不能保证返回的 Map 的类型、可变性、可序列化性、线程安全。

```java
public static <T, K, A, D> Collector<T, ?, Map<K, D>> groupingBy(
    	Function<? super T, ? extends K> classifier,
        Collector<? super T, A, D> downstream) {
	return groupingBy(classifier, HashMap::new, downstream);
}
```

示例：

参数1 - 分类函数，提供分组的 key，这里使用 `s -> s.substring(0, 1)`，表示只要首字母相同的 String 分到一组
参数2 - reduction操作函数，提供分组的 value，这里直接使用 `Collectors.toList()`，表示输出为 List

```java
List<String> list = List.of("Apple", "Banana", "Blackberry", "Coconut", "Avocado", "Cherry", "Apricots");
Map<String, List<String>> groups = list.stream()
    .collect(Collectors.groupingBy(
        s -> s.substring(0, 1),
        Collectors.toList()));
System.out.println(groups);  // >> {A=[Apple, Avocado, Apricots], B=[Banana, Blackberry], C=[Coconut, Cherry]}
```

示例2：

要计算每个城市中的一组人名

```java
Map<City, Set<String>> namesByCity = people
    .stream()
	.collect(
		groupingBy(
            Person::getCity,
            mapping(Person::getLastName,toSet())));
```



### mapping

`mapping(Function<? super T, ? extends U>, Collector<? super U, A, R>):Collector<T, ?, R>` 使接受 U 类型元素的收集器 Adapts 适应接受 T 类型元素的收集器，通过对每个输入元素应用映射 function 函数，在累加之前。

参数1 - mapper，要应用于输入元素的 function 函数
参数2 - downstream，将接受映射值的 Collector 收集器

返回 - Collector 收集器，将映射 function 函数应用于输入元素，并将映射结果提供给 downstream 下游收集器

应用场景：multi-level reduction，例如 groupingBy 和 partitioningBy 的 downstream 下游收集器

```java
public static <T, U, A, R> Collector<T, ?, R> mapping(
    						Function<? super T, ? extends U> mapper,
                           	Collector<? super U, A, R> downstream) {
    BiConsumer<A, ? super U> downstreamAccumulator = downstream.accumulator();
    return new CollectorImpl<>(downstream.supplier(),
                               (r, t) -> downstreamAccumulator.accept(r, mapper.apply(t)),
                               downstream.combiner(), downstream.finisher(),
                               downstream.characteristics());
}
```

示例：

示例2：

要计算每个城市中的一组人名

```java
Map<City, Set<String>> namesByCity = people
    .stream()
	.collect(
		groupingBy(
            Person::getCity,
            mapping(Person::getLastName,toSet())));
```



### joining

`Collectors.joining(CharSequence):Collector<CharSequence, ?, String>` 返回一个 Collector ，该收集器按相遇顺序合并输入元素，并按指定分隔符分隔

```java
public static Collector<CharSequence, ?, String> joining(CharSequence delimiter) {
    return joining(delimiter, "", "");
}
```

示例：

```java
String joined = things.stream()
   .map(Object::toString)
   .collect(Collectors.joining(", "));
```
