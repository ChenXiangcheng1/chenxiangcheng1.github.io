---
title: Reference
date: 2023-07-13
update: 2023-10-12

tags:
categories:
  - Java
  - 类库
keywords: Java
description: 关于Object的方法、深拷贝和浅拷贝
top_img:
cover:
---



# Object

Object 是所有对象的超类



## `getClass():Class<?>` 

返回此对象的运行时类对象



## `hashCode():int` 

返回哈希码值。当自定义类作为键存储在哈希表(如HashMap)中时需要重写该方法 。每当重写 hashCode() 方法时，也需要重写 equals() 方法，以维护 hashCode() 方法的通用契约，即相等的对象必须具有相等的哈希码。

[哈希笔记]()

```java
// Object
@IntrinsicCandidate
public native int hashCode();
// String
public int hashCode() {
    int h = hash;
    if (h == 0 && !hashIsZero) {
        h = isLatin1() ? StringLatin1.hashCode(value)
                       : StringUTF16.hashCode(value);
        if (h == 0) {
            hashIsZero = true;
        } else {
            hash = h;
        }
    }
    return h;
}
```



## `equals(Object):boolean`  

每当重写 hashCode() 方法时，也需要重写 equals() 方法，以维护 hashCode() 方法的通用契约，即相等的对象必须具有相等的哈希码。

```java
// Object
public boolean equals(Object obj) {
    return (this == obj);
}
// String
public boolean equals(Object anObject) {  
    if (this == anObject) {
        return true;
    }
    return (anObject instanceof String aString)
            && (!COMPACT_STRINGS || this.coder == aString.coder)
            && StringLatin1.equals(value, aString.value);
}
// 自定义类
public boolean equals(Object o) {
    if (this == o) return true;
    if (o == null || getClass() != o.getClass()) return false;  // 空或类型不通 false
    Student student = (Student) o;
    return grade == student.grade && cls == student.cls && Objects.equals(firstName, student.firstName) && Objects.equals(lastName, student.lastName);
}
```



使用 equals() 方法：

这里需要注意的是，在使用 equals() 前要判断非空才可以调用 equals() 方法，不然 null 调用 equals() 方法会报 null point exception**

```java
if(obj1 != null) {
	obj1.euqals(obj2);
}
```



### `equals()` 和 `==` 的区别

* equals()：
  Object 的 equals() 方法中，是判断 `this == obj`，比较两个对象的内存地址
  在子类重写后的 equals() 方法中，是与 hashCode 比较等价的自定义逻辑，一般都是比较两个对象的值
* ==：**比较两个对象的内存地址**，即是否是引用相同的对象



**其他语言：**

Java 中，== 是比较内存地址
Python 和 C# 中，== 是比较值



## `clone():Object`

clone() 方法是个 protected 方法，具体行为取决于怎么实现，会判断对象是否实现 Cloneable 接口

```java
@IntrinsicCandidate
protected native Object clone() throws CloneNotSupportedException;
```



### 对象的浅拷贝和深拷贝

**浅拷贝**概念：只拷贝引用本身，拷贝对象与原对象**共享引用的内部对象**，改变其中一个对象的内部对象将影响另一个对象
**深拷贝**概念：拷贝引用及其内部对象，**不再共享内部对象**，互补影响

* Object 浅拷贝：子类没有实现 Cloneable 接口
  Object 深拷贝：子类实现 Cloneable 接口，重写 clone() 方法

* 直接引用赋值是对对象浅拷贝，共享内部对象

* 方法的形参是对实参浅拷贝，共享内部对象

* 一维数组深拷贝/二维数组浅拷贝

`System.arraycopy(Object, int, Object, int, int): void` 源数组中位于 srcPos 至 srcPos+length-1 位置的组件将分别复制到目标数组的 destPos 至 destPos+length-1 位置

* 二维数组深拷贝：自己循环赋值

总结：

| 浅拷贝：共享内部对象                                         | 深拷贝：不再共享内部对象                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 引用直接赋值                                                 |                                                              |
|                                                              | 方法的基本类型+String形参对实参是深拷贝                      |
| 方法的引用形参对实参是浅拷贝                                 |                                                              |
| `System.arraycopy(Object, int, Object, int, int): void`  二维数组浅拷贝 | `System.arraycopy(Object, int, Object, int, int): void`  一维数组深拷贝 |
|                                                              | 二维数组自己循环赋值                                         |
|                                                              | 子类重写 clone() 方法，实现 Cloneable 接口                   |
| 使用方法：Lombok@With自动生成、手动重写clone方法             | 使用方法：实现 Cloneable 接口                                |



### Cloneable 接口

实现对象深拷贝

[设计模式笔记#原型模式](../../design-patterns/创建型/原型模式.md)

```java
@Override
public Object clone() {
    try {
        Graph cloned = (Graph) super.clone();        
        cloned.adj = new TreeSet[V];  // 拷贝引用的内部非基本类型对象
        for (int v = 0; v < V; v++) {
            cloned.adj[v] = new TreeSet<>();
            for (int w : this.adj[v]) {
                cloned.adj[v].add(w);
            }
        }
        return cloned;
    } catch (CloneNotSupportedException e) {
        e.printStackTrace();
    }
    return null;
}
```



## `toString():String`



## `notify():void`

[Java线程笔记#notify](./Java线程.md#`notify():void`)



## `notifyAll():void`

[Java线程笔记#notifyAll](./Java线程.md#`notifyAll():void`)



## `wait():void`

[Java线程笔记#wait](./Java线程.md#`wait():void`)



# Objects

JDK17 提供，用于对对象进行操作，或在操作前检查某些条件。

`Objects.equals(Object, Object): boolean`  
