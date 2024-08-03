---
title: Spring AOP
date: 2023-07-13
update: 2023-08-17

tags:
categories:
  - Spring
  - Spring Core
  - 核心
keywords: Spring,SpringCore,AOP
description: 关于SpringCore中AOP的底层实现，Spring AOP的最佳实践
top_img:
cover:

---



# AOP

> AOP(Aspect Oriented Programming)：面向切面编程，是OOP的扩展。将业务功能代码与切面代码分离，实现了关注点分离

软件工程的基本原则——关注点分离：不同的问题交给不同的部分去解决，每部分专注于自己的问题

* 面向切面编程 AOP 正是此种技术的体现

* 通用化功能 (事务管理、缓存、日志、安全检查) 代码的实现，对应的就是所谓的切面 (Aspect)

* 业务功能代码和切面代码分开后，架构将变得高内聚低耦合

* 确保功能的完整性：切面最终需要被合并到业务中 (Weave织入)



## 术语概念

| AOP概念    |                    | 释义                                                       |
| ---------- | ------------------ | ---------------------------------------------------------- |
| **Aspect** | **切面**           | 就是切入点和通知的组合                                     |
| **Target** | **目标对象**       | 被增强的对象                                               |
| Join Point | 连接点             | 指的是可以被拦截到的点                                     |
| Pointcut   | 切入点             | 真正被拦截到的点，支持正则表达式                           |
| **Adⅵce**  | **通知(一般切面)** | 拦截后要做的事，(方法拦截器是通知的具体实现)               |
| Advisor    | 特殊切面           | =切入点+通知                                               |
| Weaving    | 织入               | 将Advice应用到Target的过程，被应用了增强后产生一个代理对象 |



what：@Aspect

where：@Pointcut

when：@Before @AfterReturning



## 三种织入方式

* 编译时织入：需要特殊的 Java 编译器，如 AspectJ
* 类加载时(加载字节码时)织入：需要特殊的 Java编译器，如 AspectJ 和 AspectWerkz
* **运行时(动态)织入**：Spring 采用的方式，通过动态代理的方式，实现简单。动态代理会有性能上的开销但是不需要特殊的编译器或类加载器



MethodInterceptor 方法层面的增强
Introduction（引介）是特殊的 IntroductionInterceptor 通知，可以不修改类代码并在运行期为类动态地添加一些方法或Field，是类层面的增强



# Spring AOP

最佳实践：使用注解

[文档#@Aspect支持](https://docs.spring.io/spring-framework/reference/core/aop/ataspectj.html)

>  Spring AOP 的底层原理就是代理模式，默认策略是若目标对象是接口则通过JDK动态代理(反射)实现，若目标对象是类则通过Cglib动态代理(字节码增强生成子类)实现
>
>  Spring AOP 的作用就是增强目标对象，创建代理对象。
>
>  Spring AOP 提供了对 AspectJ 注解的支持



## 底层实现原理-代理模式

设计模式之代理模式 = 接口(通知) + 真实实现类(目标对象) + 代理类(代理对象)

**Spring 里代理模式的实现**

*  `getBean()` 方法包含了真实实现类的逻辑，返回的是目标代理类的实例 (所以 Spring AOP 只能作用于 Spring 容器中的 Bean）

*  由 AopProxyFactory 根据 AdvisedSupport 对象的配置来决定使用哪种代理
*  默认策略是若目标对象是接口则通过 JDK动态代理 来实现，否则通过 Cglib动态代理 来实现

[动态代理笔记#JDK动态代理](../../design-patterns/结构型/代理模式.md#JDK动态代理)

[动态代理笔记#cglib动态代理](../../design-patterns/结构型/代理模式.md#cglib动态代理)



### 源码

```java
// AbstractAutoProxyCreator
protected Object createProxy(Class<?> beanClass, @Nullable String beanName, @Nullable Object[] specificInterceptors, TargetSource targetSource) {  // 
    return this.buildProxy(beanClass, beanName, specificInterceptors, targetSource, false);
}
```

```java
// AbstractAutoProxyCreator
private Object buildProxy(Class<?> beanClass, @Nullable String beanName,
        @Nullable Object[] specificInterceptors, TargetSource targetSource, boolean classOnly) {

    if (this.beanFactory instanceof ConfigurableListableBeanFactory clbf) {
        AutoProxyUtils.exposeTargetClass(clbf, beanName, beanClass);
    }

    ProxyFactory proxyFactory = new ProxyFactory();
    proxyFactory.copyFrom(this);

    if (proxyFactory.isProxyTargetClass()) {
        // Explicit handling of JDK proxy targets and lambdas (for introduction advice scenarios)
        if (Proxy.isProxyClass(beanClass) || ClassUtils.isLambdaClass(beanClass)) {
            // Must allow for introductions; can't just set interfaces to the proxy's interfaces only.
            for (Class<?> ifc : beanClass.getInterfaces()) {
                proxyFactory.addInterface(ifc);
            }
        }
    }
    else {
        // No proxyTargetClass flag enforced, let's apply our default checks...
        if (shouldProxyTargetClass(beanClass, beanName)) {
            proxyFactory.setProxyTargetClass(true);
        }
        else {
            evaluateProxyInterfaces(beanClass, proxyFactory);
        }
    }

    Advisor[] advisors = buildAdvisors(beanName, specificInterceptors);
    proxyFactory.addAdvisors(advisors);
    proxyFactory.setTargetSource(targetSource);
    customizeProxyFactory(proxyFactory);

    proxyFactory.setFrozen(this.freezeProxy);
    if (advisorsPreFiltered()) {
        proxyFactory.setPreFiltered(true);
    }

    // Use original ClassLoader if bean class not locally loaded in overriding class loader
    ClassLoader classLoader = getProxyClassLoader();
    if (classLoader instanceof SmartClassLoader smartClassLoader && classLoader != beanClass.getClassLoader()) {
        classLoader = smartClassLoader.getOriginalClassLoader();
    }
    return (classOnly ? proxyFactory.getProxyClass(classLoader) : proxyFactory.getProxy(classLoader));
}
```

```java
// ProxyFactory
public Object getProxy(@Nullable ClassLoader classLoader) {
    return createAopProxy().getProxy(classLoader);
}
```

```java
// // ProxyCreatorSupport
protected final synchronized AopProxy createAopProxy() {
    if (!this.active) {
        activate();
    }
    return getAopProxyFactory().createAopProxy(this);
}
```

```java
// ProxyCreatorSupport
public ProxyCreatorSupport() {
    this.aopProxyFactory = new DefaultAopProxyFactory();
}
```

```java
// DefaultAopProxyFactory
public AopProxy createAopProxy(AdvisedSupport config) throws AopConfigException {
    if (!config.isOptimize() && !config.isProxyTargetClass() && !this.hasNoUserSuppliedProxyInterfaces(config)) {
        return new JdkDynamicAopProxy(config);
    } else {
        Class<?> targetClass = config.getTargetClass();
        if (targetClass == null) {
            throw new AopConfigException("TargetSource cannot determine target class: Either an interface or a target is required for proxy creation.");
        } else {
            return (AopProxy)(!targetClass.isInterface() && !Proxy.isProxyClass(targetClass) && !ClassUtils.isLambdaClass(targetClass) ? new ObjenesisCglibAopProxy(config) : new JdkDynamicAopProxy(config));  // 若目标对象是接口则使用JDK动态代理，否则使用Cglib动态代理
}	}	}
```



## 核心API-Advice

* 前置通知(MethodBeforeAdvice)
* 后置通知 (AfterReturningAdvice)
* 异常通知 (AfterThrowingAdvice)
* 最终通知 (AfterAdvice)
* 环绕通知 (AroundAdvice)



## ProxyFactoryBean

ProxyFactoryBean 是 Spring AOP 中的一个工厂类，用于根据配置创建代理对象，将切面逻辑织入到目标对象的方法中。

只能针对一个目标类，效率低下。不如自动代理



### 1一般切面 Advice

Advice 对目标类所有方法进行拦截

针对目标类的所有方法

```xml
<bean id="myBeforeAdvice" class="com.zjbti.aop.demo3.MyBeforeAdvice"/>
<!-- 配置切面 -->
<bean id="studentDaoProxy" class="org.springframework.aop.framework.ProxyFactoryBean">    
    <property name="target" ref="studentDao"/>  <!-- 配置目标对象 -->    
    <property name="proxyInterfaces" value="com.zjbti.aop.demo3.StudentDao"/>  <!-- 实现的接口 -->    
    <property name="interceptorNames" value="myBeforeAdvice"/>  <!-- 配置通知 -->
    <property name="optimize" value="true"></property>
    <!-- 其他配置:proxyTargetClass,是否对类代理而不是接口，true时使用CGLib代理-->
    <!-- singleton,是否为单例，默认单例-->
    <!-- optimize:true时，强制使用CGLib-->
</bean>
```



### 2特殊切面 Advisor

Advisor=切入点(目标对象)+通知，是一种特殊切面，针对目标类的某些方法

​	PointcutAdvisor：切入点通知，可以拦截目标类的具体哪些方法。具体实现有：
​		`DefaultPointcutAdvice`
​		`JdkRegexMethodPointcutAdvice`
​		`RegexpMethodPointcutAdvisor` 通过正则方法表达式定义切入点
​	IntroductionAdvisor：引介通知 (不要求掌握)

```xml
<!-- 配置切面 -->
<bean id="myAdvisor" class="org.springframework.aop.support.RegexpMethodPointcutAdvisor">
    <!-- <property name="pattern" value=".*"/>-->
    <!-- <property name="pattern" value=".*save.*"/>-->
    <property name="patterns" value=".*save.*,.*find.*"/>  <!-- 配置切入点 -->
    <property name="advice" ref="myAroundAdvice"/>  <!-- 配置通知 -->
</bean>
<!-- 配置切面-->
<bean id="customerDaoProxy" class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="target" ref="customerDao"/>  <!-- 配置目标对象 -->
    <property name="proxyTargetClass" value="true"/>
    <property name="interceptorNames" value="myAdvisor"/>  <!-- 配置通知 -->
</bean>
```



## AutoProxyCreator

ProxyFactoryBean，每个Bean都需要XML配置，不好维护。

AutoProxyCreator 是 BeanPostProcessor 的实现， `getBean()` 直接返回代理对象，能根据少量配置自动创建代理。



### 3基于BeanName

BeanNameAutoProxyCreator

根据Bean名称自动创建代理。代理某些Bean的所有方法

```xml
<!-- 配置切面 -->
<bean class="org.springframework.aop.framework.autoproxy.BeanNameAutoProxyCreator">  <!-- 作用于BeanPostProcessor，在初始化时就织入了，不需要id属性 -->
    <!-- 配置目标对象 -->
    <property name="beanNames" value="*Dao"/>  <!-- 对所有以Dao结尾Bean所有方法使用代理 -->
    <!-- 配置通知 -->
    <property name="interceptorNames" value="myBeforeAdvice"/>
</bean>
```



### 4基于Advisor (常用)

`DefaultAdvisorAutoProxyCreator`会扫描 Spring 容器中的 Advisor，并根据这些 Advisor 自动创建代理对象。代理某些Bean的某些方法

```xml
<!-- 配置切面 -->
<bean class="org.springframework.aop.support.RegexpMethodPointcutAdvisor">  
    <property name="pattern" value="com\.zjbti\.aop\.demo6\.CustomerDao\.save"/>  <!-- 配置目标对象 -->
    <property name="advice" ref="myAroundAdice"/>  <!-- 配置通知 -->
</bean>

<!-- 配置自动代理 -->
<bean class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator"></bean>
<!-- DefaultAdvisorAutoProxyCreator 默认情况下会扫描 Spring 容器中的 Advisor，并根据这些 Advisor 创建代理对象 -->
```



### 5基于@Aspect注解

AnnotationAwareAspectJAutoProxyCreator

根据Bean中的AspectJ注解进行自动代理

```java
@Aspect  // 将一个Java类定义为切面类
@Component
public class RequestLogAspect {
    private static final Logger logger = LoggerFactory.getLogger(RequestLogAspect.class);
    @Pointcut("execution(public * cn.cxc.framework.web..*.*(..))")  // 切入点
    public void webLog() {
    }
    @Before("webLog()")
    public void doBefore(JoinPoint joinPoint) {
        // 接收到请求，记录请求内容
        ServletRequestAttributes attributes =
                (ServletRequestAttributes) org.springframework.web.context.request.RequestContextHolder.getRequestAttributes();
        HttpServletRequest request = attributes.getRequest();
        // 记录下请求内容
        logger.info("URL: " + request.getRequestURL().toString());
        logger.info("IP: " + request.getRemoteAddr());
    }
    @AfterReturning(returning = "ret", pointcut = "webLog()")
    public void doAfterReturning(Object ret) {
        // 处理完请求，返回内容
        logger.info("RESPONSE: " + ret);
}	}
```

```运行日志
2023-07-12T22:50:39.230+08:00  INFO 14460 --- [nio-8080-exec-1] cn.cxc.framework.aop.RequestLogAspect    : URL: http://localhost:8080/hello
2023-07-12T22:50:39.230+08:00  INFO 14460 --- [nio-8080-exec-1] cn.cxc.framework.aop.RequestLogAspect    : IP: 0:0:0:0:0:0:0:1
Hello Spring Boot!
2023-07-12T22:50:39.230+08:00  INFO 14460 --- [nio-8080-exec-1] cn.cxc.framework.aop.RequestLogAspect    : RESPONSE: Hello Spring Boot!
```



## 总结

看45就好。

`ProxyFactoryBean`只能针对一个目标类，`AutoProxyCreator`是 `BeanPostProcessor` 能代理某些类。所以最佳实践是使用`AdvisorAutoProxyCreator` 代理某些类的某些方法或使用基于`@Aspect`注解的方式。

AOP 主要配置三项：切面、目标对象、通知(方法拦截器是通知的具体实现)



## 注解

| 注解 | 元注解Target | 释义 | 成员变量 |
| ---- | ------------ | ---- | -------- |
|      |              |      |          |
|      |              |      |          |
|      |              |      |          |







# AspectJ框架

Spring AOP 提供了对 AspectJ 注解的支持 (底层实现还是不同的)

AspectJ 是一个基于字节码的 Java AOP 框架



将切面类(@Aspect)中的通知(@Before @AfterReturning)织入切入点(@Pointcut)，生成代理类



## @AliasFor 注解

注解的属性别名和属性重写

### 属性别名

* 显式别名：单个注解中，一对属性上声明@AliasFor

```java
public @interface One{
    @AliasFor("B")  // 显式别名
    String A();
    @AliasFor("A")	// 显式别名
    String B();
}
```

* 隐式别名

```java
public @interface One{
    String A();
}
```

```java
@One
public @interface Two{
    @AliasFor(annotation = One.class, attribute = "A")        
    String B();  // 隐式别名
    @AliasFor(annotation = One.class, attribute = "A")   
    String C();  // 隐式别名
}
```



### 属性重写

属性重写/属性覆盖 override

* 隐式重写

```java
public @interface One{
    String A();
}
```

```java
@One
public @interface Two{
    String A();  // 隐式重写
}
```

* 显示重写

```java
public @interface One{
    String B();
}
```

```java
@One
public @interface Two{
    @AliasFor(annotation = One.class, attribute = "B")
    String A();  // 实现重写
}
```





# 未整理

## 注解开发

开启自动代理

### 切入点表达式

![1600571893159](https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting2/img/2023_09_21_02_32.png)

```java
@Before(value = "execution(* com.imooc.aspectJ.demo1.ProductDao.save(..))")
// 返回值 类.方法(参数)
```

### 通知类型+切点命名

![1600571567325](https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting2/img/2023_09_21_02_31.png)

```java
package com.imooc.aspectJ.demo1;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;

/**
 * 切面类
 */
@Aspect
public class MyAspectAnno {

    @Before(value = "myPointcut1()")
    public void before(JoinPoint joinPoint) {
        System.out.println("前置通知" + joinPoint);
    }

    @AfterReturning(value = "myPointcut1()||execution(* com.imooc.aspectJ.demo1.ProductDao.update(..))", returning = "result")
    public void afterReturning(Object result) {
        System.out.println("后置通知" + result);
    }

    // around方法返回值就是目标代理方法执行返回值
    // 参数ProceedingJoinPoint可以调用拦截目标方法执行
    @Around(value = "execution(* com.imooc.aspectJ.demo1.ProductDao.delete(..))")
    public Object around(ProceedingJoinPoint joinPoint) throws Throwable {
        System.out.println("环绕前通知");
        Object obj = joinPoint.proceed();
        System.out.println("环绕后通知");
        return obj;
    }

    // 通过设置throwing属性，可以设置发生异常对象参数
    @AfterThrowing(value = "execution(* com.imooc.aspectJ.demo1.ProductDao.findOne(..))", throwing = "e")
    public void afterThrowing(Throwable e) {
        System.out.println("异常抛出通知" + e.getMessage());
    }

    // 无论是否有异常，最终通知都会通知
    @After(value = "execution(* com.imooc.aspectJ.demo1.ProductDao.findAll(..))")
    public void after() {
        System.out.println("最终通知");
    }

    @Pointcut(value = "execution(* com.imooc.aspectJ.demo1.ProductDao.save(..))")
    public void myPointcut1() {}
}
```

