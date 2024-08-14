---
title: Java注解
date: 2023-08-07
update: 2023-08-07

tags:
categories:
  - Java
  - Java语言特性
keywords: Java,注解
description: 关于Java注解
top_img:
cover:
---

# Annotation注解

> Metadata元数据
>
> 元数据是描述数据的数据，可以**描述**代码间关系或者**代码与其它资源的关系**，元数据可以提供某些信息来协助我们程序的运行。
>
> * 注解：是一种分散式的元数据，与源代码紧绑定
>
> * XML：是一种集中式的元数据，与源代码无绑定
>
> * JSON、YAML 都可以视为元数据



## 注解的基本概念

**注解是一种分散式的元数据，不会直接影响代码执行**

用于只提供元数据(**stereotype注解标识**)、**注解处理器**通常在编译时期通过编译器的插件机制或在运行时期通过反射和动态代理等技术来实现。(处理元数据：写入配置、日志，生成代码)



## 元注解

元注释是注解注解的注解。

| 元注解                          | 元注解Target | 释义                                                         | 成员变量 |
| ------------------------------- | ------------ | ------------------------------------------------------------ | -------- |
| `@Retention({RetentionPolicy})` |              | 指定注解的**保留策略**，即注解被保留的时间长短。<br /><br />`RetentionPolicy.SOURCE`：源代码级别。注解在源码文件中<br />`RetentionPolicy.CLASS`：默认，编译时级别。注解在字节码文件中，但运行时不会被加载到JVM<br />`RetentionPolicy.RUNTIME`：运行时级别。应用程序运行时 JVM 可以通过反射机制来获取注解 |          |
| `@Target({ElemenetType})`       |              | 指定注解可以应用的**目标元素类型**<br /><br />ElementType.TYPE：类、接口(包括注解类型)或枚举定义<br />ElementType.FIELD：字段<br />ElementType.METHOD：方法<br />ElementType.PARAMETER：参数<br />ElementType.ANNOTATION_TYPE：注解<br />ElementType.CONSTRUCTOR：构造器<br />ElementType.LOCAL_VARIABLE：局部变量 |          |
| `@Documented`                   |              | 指定注解是否包含在 Java 文档中                               |          |
| `@Inherited`                    |              | 指定子类是否可以继承父类的注解                               |          |
|                                 |              |                                                              |          |



## 内置注解

| 内置注解             | 元注解Target                                                 | 释义                                                         | 成员变量                                        |
| -------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ----------------------------------------------- |
| @Override            |                                                              | 标明重写父类的某个方法，如果修饰的函数不在父类中存在，那么会报错。 |                                                 |
| @Deprecated          |                                                              | 标明某个类或方法过时，效果是在使用已经过时的元素时会警告。   |                                                 |
| @SuppressWarnings    | TYPE<br />FIELD<br />METHOD<br />PARAMETER<br />CONSTRUCTOR<br />LOCAL_VARIABLE<br />MODULE | 标明要忽略的警告                                             | 例如，unchecked，抑制没有进行类型检查操作的警告 |
| @SafeVarargs         |                                                              | 如果把一个没有定义泛型的集合，赋给一个有定义泛型的集合，会发生堆污染，会报错。@SafeVarargs可以抑制这个报错。 |                                                 |
| @FunctionalInterface |                                                              | 用来限制修饰的接口必须是一个函数式接口，不然会报错。         |                                                 |
|                      |                                                              |                                                              |                                                 |



## 自定义注解

```java
public @interface MyAnnotation{  // @interface表示该注解继承 java.lang.annotation.Anonotation 接口
    String name default "";
    String value();  // String类型的字段value
}
```

注解的字段类型：String、int、数组、Class、primitive、enumerated、annotation

```java
@MyAnnotation(value="111")
public void fun1(){
}
@MyAnnotation("222")  // 字段名为value时，为注解的成员变量赋值时可以省略变量名称
public void fun2(){
}
```



## 注解处理器

Annotation Processor 是 javac 的一个工具，它用来在编译时**扫描和处理注解**。

注解处理器通常在编译时期通过编译器的插件机制或在运行时期通过反射和动态代理等技术来实现。(处理元数据：写入配置、日志，生成代码)

内置注解由内置注解处理器处理，自定义注解需要自定义注解处理器处理。

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface CustomAnnotation {
    String value();
}
```

```java
@SupportedAnnotationTypes("com.example.CustomAnnotation")
public class CustomAnnotationProcessor extends AbstractProcessor {
    @Override
    public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment roundEnv) {
        for (TypeElement annotation : annotations) {
            for (Element element : roundEnv.getElementsAnnotatedWith(annotation)) {
                if (element.getKind().isMethod()) {
                    CustomAnnotation customAnnotation = element.getAnnotation(CustomAnnotation.class);
                    if (customAnnotation != null) {
                        String value = customAnnotation.value();  // 获取自定义注解的字段                 
                        // 元数据提取，Do something with the annotated method, e.g., generate code, log, etc.
                        System.out.println("Annotated method: " + element.getSimpleName() + ", value: " + value);
        }	}	}	}
        return true;
}	}
```

```bash
javac -processor com.example.MyAnnotationProcessor YourClass.java  # 编译时指定注解处理器
```

