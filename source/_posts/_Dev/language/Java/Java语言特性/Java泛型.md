---
title: Java泛型
date: 2023-09-12
update: 2023-09-12

tags:
categories:
  - Java
  - Java语言特性
keywords: Java,泛型
description: 关于Java泛型，涉及泛型类、泛型接口、泛型方法、类型推断、通配符、泛型擦除、子泛型类型关系
top_img:
cover:
---



# 泛型Generic

[Java SE 泛型教程](https://dev.java/learn/generics/)

> 声明(Declaration)：告诉编译器有一个特定的实体存在，通常包括指定实体的名称和类型，但不涉及实体的具体实现或分配资源。
>
> 定义(Definition)：明确地指定实体的名称、类型和其他相关的属性或行为。通常包括为实体分配内存空间或分配其他必要的资源。

**泛型就是将类型参数化为类型参数，在声明时使用类型参数，在定义时指明类型(若使用了泛型通配符则运行时才指明类型)。使代码得到重用。**

Object 也可以实现代码重用，泛型是为了可以**在编译时提供类型检查，编译后类型擦除，由编译器加入转型动作，使得转型是不需要关心且安全的。**

| 具体类型(强制类型转换) | 使用泛型                       |
| ---------------------- | ------------------------------ |
| 运行时                 | 编译时类型检查，运行时类型安全 |
| 需要强制转换           | 无需强制类型转换，方便         |
|                        | 代码重用，更通用               |



## 类型参数

类型参数(type parameter)可以是任意合法的 Java 标识符，通常使用单个大写字母来表示，例如 T、E(Java集合框架广泛使用)、N(数字)、<K, V>、<S, U, V> 等

| <类型参数>                |              | 类型自变量type arguments                                     |
| ------------------------- | ------------ | ------------------------------------------------------------ |
| T                         |              | 类型自变量是，任何非原始类型(任何类类型、任何接口类型、任何数组类型) |
| <>                        |              | 类型自变量是，由编译器进行类型推断                           |
| T extends Class1 & Class2 | 有界类型参数 | 类型自变量是，继承指定类(Class1/Class2)或实现指定接口(Class1/Class2) |
|                           |              |                                                              |



## 泛型类型

泛型类型指泛型类或泛型接口

### 泛型类1

```java
Class A { /* ... */ }
interface B { /* ... */ }
interface C { /* ... */ }
public class Box<T1 extends A & B & C, T2, ..., T3> {
    private T1 t;
    public void set(T1 t) { this.t = t; }
    public T1 get() { return t; }
    public void print() {
        System.out.println(t1);
    }
    ...
}
```



#### 子泛型类型

Holder<B> 是 Holder<T extends A> 的子泛型类型，因为 B 是 A 的子类

Holder<String> 不是 Holder<T extends A> 的子泛型类型，因为 String 不是 A 的子类

```java
class A {}
class B extends A{}
class Holder<T extends A> {
    private T t;
    public void set(T t) { this.t = t; }
    public T get() { return t; }
}
public class GenericLearn {
    public static void print(List<A> list) {
        for (A a : list) {
            System.out.println(a);
    }	}
    public static void main(String[] args) {
        Holder<A> holder1 = new Holder<>();  // 类型推断
        Holder<B> holder2 = new Holder<>();     
        Holder<String> holder3 = new Holder<>();  // 编译错误
        print(listA);        
        print(listB);  // 编译错误， List<B> 不是 List<A> 的子类型，即使 B 是 A 的子型。
}	}
```

只要不改变类型参数，类型之间的子类型关系就会保留下来。

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_08_25_22_47.png" alt="image-20230825224753769" style="zoom:67%;" />



### 泛型接口2

```java
interface Handler<T> {
   void handle(T t);
}

class StringHandler implements Handler<String> {
   @Override
   public void handle(String s) {
       // doNothing
   }
}
```



## 原始类型

原始类型是没有类型自变量的泛型类或泛型接口，例如 `Box` 是泛型类型 `Box<T>` 的原始类型

原始类型执行泛型方法或者将原始类型赋值给泛型类型，会warning: uncheck。uncheck 指编译器没有足够的类型信息确保类型安全

结论：应该避免使用原始类型



## 泛型方法3

```java
class BatchUtil {  // 批量操作的工具类
    public static <T> void batchExec(List<T> list, int batchSize, Consumer<List<T>> action) {
        for (int i = 0; i < list.size(); i += batchSize) {
            int endIndex = i + batchSize > list.size() ? list.size() : i + batchSize;
            List<T> tempList = list.subList(i, endIndex);
            action.accept(tempList);
        }
    }
}
```

```java
public class GenericLearn {
    public static void main(String[] args) {
        List<Integer> list = Lists.newArrayList(1, 2, 3, 4, 5, 6, 7);
        BatchUtil.<Integer>batchExec(list, 3, System.out::println);
    }
}
```

```输出
[1, 2, 3]
[4, 5, 6]
[7]
```



## 类型推断

类型推断是 Java 编译器查看每个方法调用和相应声明以确定使该调用适合的类型参数的能力。

推理算法根据**调用参数，目标类型、明显的预期返回类型**来推断，不使用程序后面的结果。



### 泛型方法的类型推断

```java
static <T> T pick(T a1, T a2) { return a2; }
Serializable s = pick("d", new ArrayList<String>());
```

```java
public class BoxDemo {
  public static <U> void addBox(U u, 
      java.util.List<Box<U>> boxes) {
    Box<U> box = new Box<>();
    box.set(u);
    boxes.add(box);
  }
  public static <U> void outputBoxes(java.util.List<Box<U>> boxes) {
    int counter = 0;
    for (Box<U> box: boxes) {
      U boxContents = box.get();
      System.out.println("Box #" + counter + " contains [" +
             boxContents.toString() + "]");
      counter++;
  }	}
  public static void main(String[] args) {
    java.util.ArrayList<Box<Integer>> listOfIntegerBoxes =
      new java.util.ArrayList<>();
    BoxDemo.<Integer>addBox(Integer.valueOf(10), listOfIntegerBoxes);  // 不必要
    BoxDemo.addBox(Integer.valueOf(20), listOfIntegerBoxes);  // 类型推断
    BoxDemo.addBox(Integer.valueOf(30), listOfIntegerBoxes);
    BoxDemo.outputBoxes(listOfIntegerBoxes);     
  }
}
```

```输出
Box #0 contains [10]
Box #1 contains [20]
Box #2 contains [30]
```



### 泛型类实例化的类型推断

```java
Map<String, List<String>> myMap = new HashMap<String, List<String>>();  // 不必要
Map<String, List<String>> myMap = new HashMap<>();  // 类型推断
```



### 泛型构造函数的类型推断

```java
class MyClass<X> {
  <T> MyClass(T t) {
    // ...
  }
}
MyClass<Integer> myObject = new MyClass<>("");  // 类型推断
```

编译器推断形式类型参数 X 的类型是 Integer，该泛型类的构造函数的形式类型参数 T 的类型是 String



### 目标类型

Java 编译器利用目标类型来推断泛型方法的类型参数。

表达式的目标类型是 Java 编译器所期望的数据类型，具体取决于表达式的出现位置。



#### 泛型方法的类型推断

```java
static <T> List<T> emptyList();
List<String> listOne = Collections.<String>emptyList();  // 不必要
List<String> listOne = Collections.emptyList();  // 类型推断
```

```java
static <T> List<T> emptyList();
void processStringList(List<String> stringList) {
    // process stringList
}
processStringList(Collections.<String>emptyList());  // 不必要
processStringList(Collections.emptyList());  // 类型推断
```



#### Lambda表达式的类型推断

```java
public static void printPersons(List<Person> roster, CheckPerson tester)
public void printPersonsWithPredicate(List<Person> roster, Predicate<Person> tester) 
printPersons(
    people, 
    p -> p.getGender() == Person.Sex.MALE
        && p.getAge() >= 18
        && p.getAge() <= 25);  // lambda表达式返回 CheckPerson类型
printPersonsWithPredicate(
        people,
        p -> p.getGender() == Person.Sex.MALE
             && p.getAge() >= 18
             && p.getAge() <= 25);  // lambda表达式返回 Predicate<Person>类型
```



## 使用通配符的泛型4

在泛型代码中，通配符(?)表示未知类型。声明时使用，为了编译时检查。

通配符捕获：编译器来推断通配符的类型



| 通配符                     |            | 释义                                                         |
| -------------------------- | ---------- | ------------------------------------------------------------ |
| ?                          | 无界通配符 | 任意类型，在某些类型转换时可以避免不必要的 unchecked 错误。<br />应用场景：当前操作可以使用 Object 类中提供的功能实现，使用不依赖于类型参数的方法(即不用关心泛型类型参数的具体类型时) |
| ? extends  Class1 & Class2 | 上界通配符 | 特定类型或该类型的超类型。                                   |
| ? super 类型               | 下界通配符 | 特定类型或该类型的超类型。                                   |
|                            |            |                                                              |

继承关系：

![image-20230826010257643](https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_08_26_01_03.png)

### 通配符使用指南(重要)

“in”变量(提供数据)是用一个上界通配符定义的，使用 `extends` 关键字。
即只读操作

“out”变量(保存数据)是用一个下限通配符定义的，使用 `super` 关键字。

如果可以使用 Object 类中定义的方法访问“In”变量，请使用无边界通配符。
即只使用不需要具体类型的操作

如果代码需要同时以“In”和“out”变量的形式访问变量，请不要使用通配符。



### 无界通配符

**泛型方法中的无界通配符**

```java
public static void printList(List<Object> list) {  // 使用具体类型变量
    for (Object elem : list)
        System.out.println(elem + " ");
    System.out.println();
}
List<Integer> li = Arrays.asList(1, 2, 3);
List<String>  ls = Arrays.asList("one", "two", "three");
printList(li);  // 编译时错误，需要的类型：List<Object>，提供的类型：List<Integer>。虽然Integer是Object的子类，但是List<Integer>不是List<Object>的子类，所以形参与实参不匹配
printList(ls);  // 编译错误
```

```java
public static void printList(List<?> list) {  // 使用无界通配符
    for (Object elem: list)
        System.out.print(elem + " ");
    System.out.println();
}
List<Integer> li = Arrays.asList(1, 2, 3);
List<String>  ls = Arrays.asList("one", "two", "three");
printList(li);
printList(ls);
```



**泛型类中的无界通配符**

```java
class NaturalNumber {
    private int i;
    public NaturalNumber(int i) { this.i = i; }
}
class EvenNumber extends NaturalNumber {
    public EvenNumber(int i) { super(i); }
}
List<EvenNumber> le = new ArrayList<>();
List<? extends NaturalNumber> ln = le;
ln.add(new NaturalNumber(35));  // 编译时错误，需要的类型capture of ? extends NaturalNumber，提供的类型NaturalNumber。使用带有通配符的泛型类型时编译器无法确定通配符的具体类型，为了保证类型安全性防止在运行时出现类型不匹配的问题，会报错。
```



## 泛型类型擦除5

类型擦除确保不会为类型参数创建新的类，因此泛型不会带来运行时额外开销

* 如果类型参数是有界的，Java 编译器将擦除所有类型参数，并将每个类型参数替换为其边界

* 如果类型参数为无界的，则用 `Object` 替换。



### 泛型方法擦除

```java
public static <T> int count(T[] anArray, T elem) {
    int cnt = 0;
    for (T e : anArray)
        if (e.equals(elem))
            ++cnt;
        return cnt;
}
```

擦除后：

```java
public static int count(Object[] anArray, Object elem) {
    int cnt = 0;
    for (Object e : anArray)
        if (e.equals(elem))
            ++cnt;
        return cnt;
}
```



### 桥接方法的类型擦除

```java
public class Node<T> {
    public T data;
    public Node(T data) { this.data = data; }
    public void setData(T data) {
        System.out.println("Node.setData");
        this.data = data;
}	}
public class MyNode extends Node<Integer> {
    public MyNode(Integer data) { super(data); }
    public void setData(Integer data) {
        System.out.println("MyNode.setData");
        super.setData(data);
}	}
MyNode mn = new MyNode(5);
Node n = mn;            // A raw type - compiler throws an unchecked warning
n.setData("Hello");     // Causes a ClassCastException to be thrown.
Integer x = mn.data;    
```

擦除后：

```java
public class Node {
    public Object data;
    public Node(Object data) { this.data = data; }
    public void setData(Object data) {
        System.out.println("Node.setData");
        this.data = data;
}	}

class MyNode extends Node {    
    public MyNode(Integer data) { super(data); }
    public void setData(Object data) {  // 编译器生成的桥接方法  // 因为类型擦除后方法签名不匹配，setData(Integer):void 不会覆盖 setData(Object):void，将setData(Object):void委托给setData(Integer):void
        setData((Integer) data);  // 委托给原始方法
    }
    public void setData(Integer data) {  // 原始方法
        System.out.println("MyNode.setData");
        super.setData(data);
    }
}
MyNode mn = new MyNode(5);
Node n = (MyNode)mn;  // 原始类型 - 编译器抛出一个未检查警告。n实际上指向mn
n.setData("Hello");   // 抛出ClassCastException异常，类型转换异常。n.setData("Hello")调用nm.setData("Hello")
Integer x = (String)mn.data; 
```



### 影响

```java
class A {...}
class B extends A {...}
List<?> list = new ArrayList<>();  // 类型擦除为 List<Object>
Object obj = list.get(0);
list.add(new A());  // compile error
list.add(new Object());  // 可以
```

使用无界通配符后，只能对 List 执行 add null、clear()、获得迭代器、remove()、捕获通配符读取元素。不能存储新元素、修改现有元素。
这是因为 List 的 add(E) 方法，其参数是泛型类，运行时类型擦除为 List<Object>

```java
Class A {...}
Class B entends A {...}
List<? super B> list = new ArrayList<>();
Object obj = list.get(0);
B b = list.get(0);
A a = list.get(0);
list.add(new B());  // 可以
list.add(new A());  // 编译错误
list.add(new Object());  // 编译错误
```

使用下届通配符后



## 使用泛型的限制6

Java 的泛型只是编译期的泛型，一旦编译成字节码，泛型就被擦除了。所以**任何需要具体类型的操作**，比如实例化对象等，**都无法进行**。

```java
class Holder<T> {    
    private T t = new T();  // 编译错误，需要具体类型的操作无法进行
    private T[] array = new T[0];
    private Class<T> clazz = T.class;
}
```

泛型擦除不接受基本类型
