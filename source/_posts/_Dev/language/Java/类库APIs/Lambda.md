---
title: 函数式编程
date: 2023-10-11
update: 2023-10-11

tags:
categories:
  - Java
  - 类库
keywords: Java
description: 关于Lambda表达式、java.util.function、方法引用
top_img:
cover:
---



# 函数式编程

编程范式：抽象程度越低越贴近机械，越高越贴近抽象计算，各范式也不是对立的概念

* 面向过程

* OOP
* 函数式编程(Functional Programming)，函数是一等公民(可以被传递、组合和存储)。即把函数作为基本运算单元，函数可以作为变量，可以接收函数，还可以返回函数。
  函数式编程起源：是数学家阿隆佐·邱奇研究的一套函数变换逻辑 Lambda Calculus(λ-Calculus演算)，所以也把函数式编程称为 Lambda 计算。



## 匿名函数

Java 不支持单独定义函数

函数式接口：是指**只定义了一个抽象方法的接口(不算default、static、Object的方法)**。`@FunctionalInterface  // 声明该接口是函数式接口`
函数：是指函数式接口的实现(可以被传递、组合和存储)
匿名函数：匿名内部类、Lambda 表达式，都用于创建匿名函数。(Lambda 表达式只支持函数式接口)

[内部类笔记#匿名内部类](../OOPs/内部类.md#匿名内部类(主要))



## Lambda表达式基础

Lambda 函数本质上还是对象，编程时省略了类定义，函数方法名 (**匿名函数**)

示例：

```java
Comparator<int[]> comparator = (o1, o2) -> {
    if (o1[0] == o2[0])
        return Integer.compare(o1[1], o2[1]);
    return Integer.compare(o1[0], o2[0]);
};
```



### 类型推断

**编译器能够推断** Lambada 表达式的**参数类型**、Lambada 表达式的**目标类型**

也可以自己指定参数类型，例如 `(int[] iarray) -> {}`



### 返回

如果只有一个返回值，**return 写法可省略**，例如 `()->"Hello world!"`



### 变量捕获

Lambda表达式内部可以捕获外部的局部变量，但这些变量必须是隐式或显式声明为 final 的，或者实际上是不可变的。这是因为 Lambda 表达式在创建时会保存对外部变量的引用，而这些变量在 Lambda 表达式的执行过程中不能被修改。

final 局部变量、实例变量(this.xx)、静态变量



## java.util.function

是一个函数式编程包，包含了一组函数式接口，用于支持函数式编程和 Lambda 表达式。
一般只有自己写框架时才会使用(使方法更加灵活)

相关应用：[Stream API笔记](./Stream_API.md)



### 总结

functional，直接引用抽象概念，而不是对象

1. `Consumer<T>`：接受一个输入参数，但没有返回值。
   **消费者，需要提供一个参数给方法消费**
   **T to void 的一元函数accept**
2. `Supplier<T>`：不接受任何参数，但返回一个结果。
   **生产者，返回一个结果**
   **to R 的无参函数get**
3. `Function<T, R>`：接受一个输入参数，并将其转换为输出结果。
   **函数**
   **T to R 的一元函数apply**
4. `Predicate<T>`：接受一个输入参数，返回一个布尔值结果。
   **谓词**
   **T to boolean 的一元函数test**
5. `UnaryOperator<T>`：接受一个输入参数，并返回与输入参数类型相同的结果。
   **T to T 的一元函数apply**
6. `BinaryOperator<T>`：接受两个相同类型的输入参数，并返回一个结果。
   **(T, T) to T 的二元函数apply**



### Function

表示一个函数，接受一个参数生成一个结果。

```java
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
    default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
        Objects.requireNonNull(before);
        return (V v) -> apply(before.apply(v));
    }
    default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (T t) -> after.apply(apply(t));
    }
    static <T> Function<T, T> identity() {
        return t -> t;
    }
}
```

示例：

```java
this.roots = map.get(1).stream()
        .map(r -> {
            return new CategoryPureVO(r);
        })
        .collect(Collectors.toList());
```



### Supplier

表示一个结果的提供者。需要返回一个值

```java
@FunctionalInterface
public interface Supplier<T> {
    T get();
}
```

示例：

```java
return optionalTheme.orElseThrow(() -> new NotFoundException(30003));
```



### Consumer

表示一个操作，接受单个输入参数但不返回结果

```java
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);
    default Consumer<T> andThen(Consumer<? super T> after) {
        Objects.requireNonNull(after);
        return (T t) -> { accept(t); after.accept(t); };
    }
}

```

示例1：

```java
t2.ifPresent(t -> System.out.println(t));
t2.ifPresent(System.out::println);
```

```java
Consumer<String> printer = System.out::println;
```



### Predicate

表示一个参数的谓词(刻画客体性质或关系的词返回真假，boolean值)

```java
@FunctionalInterface
public interface Predicate<T> {
    boolean test(T t);
    default Predicate<T> and(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) && other.test(t);
    }
    default Predicate<T> negate() {
        return (t) -> !test(t);
    }
    default Predicate<T> or(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) || other.test(t);
    }
    static <T> Predicate<T> isEqual(Object targetRef) {
        return (null == targetRef)
                ? Objects::isNull
                : object -> targetRef.equals(object);
    }
    @SuppressWarnings("unchecked")
    static <T> Predicate<T> not(Predicate<? super T> target) {
        Objects.requireNonNull(target);
        return (Predicate<T>)target.negate();
    }
}
```

示例：

`filter(Predicate<? super T>):Stream<T>`



### BiFunction

表示接受两个参数并生成结果的函数。

```java
@FunctionalInterface
public interface BiFunction<T, U, R> {
    R apply(T t, U u);
    default <V> BiFunction<T, U, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (T t, U u) -> after.apply(apply(t, u));
    }
}
```



#### BinaryOperator<T>

继承自BiFunction<T,T,T>，表示对两个相同类型的操作数的运算，生成与操作数相同类型的结果。
适用于操作数和结果都属于同一类型的情况。

```java
@FunctionalInterface
public interface BinaryOperator<T> extends BiFunction<T,T,T> {
    public static <T> BinaryOperator<T> minBy(Comparator<? super T> comparator) {
        Objects.requireNonNull(comparator);
        return (a, b) -> comparator.compare(a, b) <= 0 ? a : b;
    }
    public static <T> BinaryOperator<T> maxBy(Comparator<? super T> comparator) {
        Objects.requireNonNull(comparator);
        return (a, b) -> comparator.compare(a, b) >= 0 ? a : b;
}	}
```



## 方法引用

方法引用是 Lambda 表达式的简化写法，可以直接引用已经存在的方法(要求方法签名(参数和返回)一致)

### 静态方法引用

`RefType::staticMethod`

```java
DoubleUnaryOperator sqrt1 = a -> Math.sqrt(a);
DoubleUnaryOperator sqrt2 = Math::sqrt;
```

示例：

```java
String[] array = new String[] { "Apple", "Orange", "Banana", "Lemon" };
Arrays.sort(array, String::compareTo);
System.out.println(String.join(", ", array));  // >> Apple, Banana, Lemon, Orange
```



### 未绑定方法引用

`RefType::instanceMethod`

示例1：

```java
Function<String, Integer> toLength = s -> s.length();
Function<String, Integer> toLength = String::length;
```

示例2：

```java
BiFunction<String, String, Integer> indexOf = (sentence, word) -> sentence.indexOf(word);
BiFunction<String, String, Integer> indexOf = String::indexOf;
```



### 绑定方法引用

`expr::instanceMethod`
expr 是一个表达式，返回一个对象

示例：

```java
Consumer<String> printer = System.out::println;
```



### 构造方法引用

`ClassName::new`

示例1：

```java
Supplier<List<String>> newListOfStrings = () -> new ArrayList<>();
Supplier<List<String>> newListOfStrings = ArrayList::new;
```



#### 示例2：

List\<String> 转换为 List\<Person>

```java
class Person {
    String name;
    public Person(String name) {
        this.name = name;
    }
}

List<String> names = List.of("Bob", "Alice", "Tim");
List<Person> persons = ???
```

**方案1：**

传统的做法，先定义一个ArrayList<Person>，然后用for循环填充这个List

```java
List<String> names = List.of("Bob", "Alice", "Tim");
List<Person> persons = new ArrayList<>();
for (String name : names) {
    persons.add(new Person(name));
}
```

**方案2：**

构造方法引用 Class::new

```java
List<String> names = List.of("Bob", "Alice", "Tim");
List<Person> persons = names.stream().map(Person::new).collect(Collectors.toList());
```

为什么 map() 可以传入构造方法引用？
因为这里的 Steam 的 map() 需要传入 Function 函数式接口
Function 泛型替换后为 Person apply(String)，与构造方法(会隐式返回this实例)签名一致，所以可以使用方法引用



### 总结

| Name        | Syntax 语法               | Lambda equivalent 对等值                    |
| ----------- | ------------------------- | ------------------------------------------- |
| Static      | `RefType::staticMethod`   | `(args) -> RefType.staticMethod(args)`      |
| Unbound     | `RefType::instanceMethod` | `(arg0, rest) -> arg0.instanceMethod(rest)` |
| Bound       | `expr::instanceMethod`    | `(args) -> expr.instanceMethod(args)`       |
| Constructor | `ClassName::new`          | `(args) -> new ClassName(args)`             |



## 进阶

[4. Combining Lambda Expressions 和 5. Writing and Combining Comparators](https://dev.java/learn/lambdas/)













