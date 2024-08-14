---
title: Java反射机制
date: 2023-07-13
update: 2023-08-07

tags:
categories:
  - Java
  - Java语言特性
keywords: Java,反射
description: 关于反射API和JDK动态代理
top_img:
cover:
---


# Java反射机制主要考点

## 定义

Java 反射指：**在运行时能够获取类或对象的成员变量和方法信息以及创建实例的能力**。

优点：**运行时动态地加载类**，不需要在编译时就明确地引用该类，从而提供更大的**灵活性**和**可扩展性**。
缺点：反射要解析字节码文件**执行时性能较低**，编译时**没有类型检查**存在安全问题



## 反射API

JAVA 在 `java.lang.reflect` 包中提供了对反射功能的支持。



### AnnotationElement 接口

该接口声明了很多可以获取注解的方法

AnnotatedElement 接口的实现类：有 AccessibleObject(子类有Constructor/Field/Method)、Class、Package

| AnnotationElement接口方法                                  | 释义                                                      |
| ---------------------------------------------------------- | --------------------------------------------------------- |
| `getAnnotation(Class<A>):<A extends Annotation>`           | 返回Class<A>类型的该元素的注解                            |
| `getAnnotations():Annotation[]`                            | 返回此元素上存在的所有注解                                |
| `getDeclaredAnnotations():Annotation[]`                    | 返回直接存在该元素上的所有注解，忽略继承的注解            |
| `isAnnotationPresent(Class<? extends Annotation>):boolean` | 如果此元素上存在指定类型的注解，则返回true，否则返回false |
|                                                            |                                                           |
| **AccessibleObject(子类有Constructor/Field/Method)方法**   |                                                           |
| `getParameterAnnotations():Annotation[][]`                 | 获得所有参数(一维)的所有注解(二维)                        |
|                                                            |                                                           |



#### 通用: Declared 有无的区别

* 带 Declared 声明的 API，用于获取类中定义的**所有方法**，包括 public、protected、default 和 private 方法，但**不包括继承**的方法。
  若访问私有对象会抛出 java.lang.IllegalAccessException 异常，所以在 try-catch 中需要设置 setAccessible(true) 取消 Java 语言访问检查，或者直接 setAccessible(true)

* 不带 Declared 声明的 API，只能获取到 **public 方法和继承自父 public **的方法，无法获取 protected、default 和 private 方法。



#### Class类

在 javac 将 java 源代码编译为字节码时，会自动将源代码中的类名(符号引用)转换成相应的静态常量属性，用于指向该类的Class对象

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_09_11_20_19.png" alt="image-20230911201951346" style="zoom: 67%;" />

##### 获取Class对象

```java
Class.forName(fullClassName)
Classname.class
objectname.getClass()
```

##### Class对象常用的方法

```java
Object classobject.newInstance()
ClassLoader classobject.getClassLoader()
```



#### Method类

##### 获取Method对象

```java
classobject.getDeclareMethods()  // 不包括继承和实现的方法，不保证顺序
classobject.getDeclaredMethod("methodName",String.class,int.class);  // 需要该成员方法的名称和入口参数的类型作为参数。
classobject.getMethod("methodName",new Class[] {String.class,int.class});  // 包括继承和实现的方法
```

##### Method对象常用的方法

```java
methodobject.invoke(Object obj,Object...args)		利用指定参数args调用执行指定对象obj中的该方法，返回值为object类型
methodobject.setAccessible(boolean flag);  // 当调用私有方法时，需要设置为 true 取消 Java 语言访问检查
```



#### Field类

##### 获取Field对象

```java
classobject.getDeclaredFields()  // 不包括继承和实现的方法，不保证顺序
classobject.getDeclaredField("variableName");  // 需要该成员变量的名称作为参数
classobject.getField("variableName")  // 包括继承和实现的方法
```

##### Field类常用的方法

```java
string fieldobject.getName()		获得成员变量的名称
Class fieldobject.getType()		获得成员变量的类型为class对象如int.class，可以直接sout
fieldobject.get(Object obj)	field字段属性，obj对象，返回的是字段值为Object类型    
fieldobject.set(Object obj,Object value)	
```



#### Constructor类

##### 获取Constructor对象

```java
Classname.getDeclaredConstructors()  // 不包括继承和实现的方法，不保证顺序
classobject.getDeclaredConstructor(String.class,int.class);  // 需要该构造方法的入口参数的类型作为参数
classobject.getDeclaredConstructor(new Class[] {String.class,int.class});
classobject.getConstructor(String.class,int.class);  // 包括继承和实现的方法
```

##### Constructor类的常用方法

```java
Object constructorobject.newInstance(Object...initargs)
constructorobject.setAccessible(boolean flag)  // 当调用私有方法时，需要设置为 true 取消 Java 语言访问检查
constructorobject.isVarArgs()  // 是否允许带有可变数量的参数
constructorobject.getParameterTypes()  // 入口参数类型
constructorobject.getExceptionTypes  // 可能抛出的异常类型
int constructorobject.getModifiers()  // 访问控制修饰符的int形式

Boolean Modifier.isStatic/Final/Public/Protected/Private(int mod)	返回boolean类型
String Modifier.toString(int mod)// 访问控制修饰符的str形式
```



## 应用

### JDK 动态代理

[代理模式笔记#JDK动态代理](../../design-patterns/结构型/代理模式.md#JDK动态代理)


