---
title: Optional
date: 2023-10-11
update: 2023-10-11

tags:
categories:
  - Java
  - 类库
  - util
keywords: Java
description: 关于java.util.Optional
top_img:
cover:
---



# Optional

Optional 类
Optional 是一个容器对象，可能所包含的值存在或为非空值。提供了基于所包含的值存在或缺席的方法。

Optional 主要用作方法返回类型，赋予所包含对象一些判空的方法，用于**简化代码**(无需显式的空值检查)，让接收 Optional 的人需要逻辑处理可能为空的值的，避免空指针问题(接收对象后需要判空, 如空抛异常, 非空执行)，



示例：

```java
// Controller 层
Optional<Theme> optionalTheme = this.themeService.findByName(themeName);
return optionalTheme.orElseThrow(() -> new NotFoundException(30003));

// Repository 层
Optional<Theme> findByName(String name);
```



## 构建方法

`Optional.empty()` 返回空 Optional 对象

`Optional.of(T):Optional<T>` 返回 Optional 对象，描述给定的非空值

`Optional.ofNullable(T):Optional<T>` 返回 Optional 对象，描述给定值，如果值为空则返回空 Optional



## 中间方法

### map

`map(Function<? super T, ? extends U>):Optional<U>` 如果值存在则返回 Optional，描述 Function 函数映射的值，否则返回一个空 Optional

示例

```java
t2.map(t -> t + 'b')
  .ifPresent(System.out::println);
```



### filter

`filter(Predicate<? super T>):Optional<T>` 如果值存在并且与给定的 Predicate 谓词匹配则返回一个描述该值的 Optional，否则返回空 Optional



### or

`or(Supplier<? extends Optional<? extends T>>):Optional<T>` 如果值存在则返回 Optional，否则返回 Supplier 函数产生的 Optional



## 终端方法

空值处理方法，返回不同的值

### isPresent

`isPresent():boolean` 如果值存在则返回true，否则返回false



### ifPresentOrElse

`ifPresentOrElse(Consumer<? super T>, Runnable):void` 如果值存在则用该值执行给定的Consumer<? super T>操作，否则执行给定的基于空的Runnable操作



### orElse

`orElse(T):T` 如果值存在则返回该值，否则返回参数值



### orElseThrow

`orElseThrow():T` 如果值存在则返回该值，否则抛出 NoSuchElementException

`get():T` 如果值存在则返回该值，否则抛出 NoSuchElementException

`orElseThrow(Supplier<? extends X>):T` 如果值存在则返回该值，否则抛出由 Supplier 函数产生的异常

示例：

```java
String targetURL = instances.stream()
                // 数据变换
                .map(instance -> instance.getUri().toString() + "/users/{id}")
                .findFirst()
                .orElseThrow(() -> new IllegalArgumentException("当前没有实例"));
```



### orElseGet

`orElseGet(Supplier<? extends T>):T` 如果值存在则返回该值，否则返回 Supplier 函数产生的结果。



## 其他

### Stream

`stream():Stream<T>` 如果值存在则返回顺序流，只包含该值，否则返回一个空流。
用于将 optional 元素流转换为值元素流



