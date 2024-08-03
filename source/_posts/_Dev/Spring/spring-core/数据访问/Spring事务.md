---
title: Spring事务
date: 2023-07-13
update: 2023-08-17

tags:
categories:
  - Spring
  - Spring Core
  - 数据访问
keywords: Spring,Spring事务
description: 主要是关于Spring事务
top_img:
cover:
---

# Spring 事务处理机制

## 事务的细粒度

一般封装在业务层的每一个方法上，每个业务方法是原子性的，在持久层dao太细，在表现层太粗



## ACID

[MySQL8笔记#3.2事务的四大特性 ACID](..\..\MySQL\MySQL8.md)



## 编程式事务处理

基于底层 API 的编程式事务管理

* PlatformTransactionManager(核心)
* Transaction Definition
* TransactionStatus

基于 TransactionTemplate 的编程式事务管理

* TransactionTemplate



### 1基于底层 API (基础)

![image-20230816215403191](https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_08_16_21_54.png)

事务管理器(核心)有三种实现：声明式事务管理、编程式事务管理、注解驱动事务管理



#### PlatformTransactionManager(核心)

不管使用哪种事务管理的方法，都需要声明事务管理器。

方法：
`getTransaction(TransactionDefinition):TransactionStatus` 开启一个事务，获取当前活动事务

`commit(TransactionStatus):void` 提交事务

`roolback(TransactionStatus):void` 回滚事务



#### TransationDefinition接口

用于配置事务 

##### 事务隔离级别(隔离性的具体体现)

需要注意是 Spring 是事务隔离级别是建立在连接的数据库支持事务的基础上的，如果 Spring 项目连接的数据库不支持事务(或对应的事务隔离级别)，那么即使在 Spring 中设置了事务隔离级别，也是无效的设置。

[MySQL8笔记#事务隔离级别](..\..\MySQL\MySQL8.md)

| Spring 中事务隔离级别          | 释义                                                         | 并发问题                                                     |
| ------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| ISOLATION_DEFAULT (Spring默认) | 使用连接数据库的默认事务隔离级别。                           |                                                              |
| ISOLATION_READ_UNCOMMITTED     | 读未提交，可以读取到其他事务中未提交的数据。存在脏读。       | 该隔离级别因为可以读取到其他事务中未提交的数据，而未提交的数据可能会发生回滚，因此我们把该级别读取到的数据称之为脏数据，把这个问题称之为脏读 |
| ISOLATION_READ_COMMITTED       | 读已提交，只能读取到已经提交事务的数据。解决了脏读，存在不可重复读。 | 不会有脏读问题。但由于在事务的执行中可以读取到其他事务提交的结果，所以在不同时间的相同 SQL 查询中，可能会得到不同的结果，这种现象叫做不可重复读 |
| ISOLATION_REPEATABLE_READ      | 可重复读，解决了不可重复读，但存在幻读。                     | 它能确保同一事务多次查询的结果一致。但也会有新的问题，比如此级别的事务正在执行时，另一个事务成功的插入了某条数据，但因为它每次查询的结果都是一样的，所以会导致查询不到这条数据，自己重复插入时又失败（因为唯一约束的原因）。明明在事务中查询不到这条信息，但自己就是插入不进去，这就叫幻读 （Phantom Read）； |
| ISOLATION_SERIALIZABLE         | 串行化，解决所有并发问题，但性能太低。                       | 最高的事务隔离级别，它会强制事务排序，使之不会发生冲突，从而解决了脏读、不可重复读和幻读问题，但因为执行效率低，所以真正使用的场景并不多。 |



##### 事务默认超时

TIMEOUT_DEFAULT，默认30秒



##### 事务传播行为

https://zhuanlan.zhihu.com/p/148504094

事务传播行为：解决方法调用时，被调用的方法其事务如何处理的问题。

```java
public void a() {
    // begin
    // do something...
    b();
    // commit
}
public void b() {
    // begin
    // do something...
    // commit
}
```

| Spring 事务传播行为                             | 翻译                | 释义                                                         |
| ----------------------------------------------- | ------------------- | ------------------------------------------------------------ |
| PROPAGATION_REQUIRED (Spring默认，也是最常用的) | 方法b必须使用事务   | 支持当前事务；如果当前没有事务，就**新建一个事务**。         |
| PROPAGATION_SUPPORTS                            | 看a                 | 支持当前事务；如果当前没有事务，就以非事务方式执行。         |
| PROPAGATION_MANDATORY                           | 强制b使用事务       | 支持当前事务，如果当前没有事务，就抛出异常。                 |
| PROPAGATION_REQUIRES_NEW                        | 需要                | 新建事务，如果当前存在事务，把当前事务挂起。                 |
| PROPAGATION_NOT_SUPPORTED                       |                     | 以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。   |
| PROPAGATION_NEVER                               | 方法b一定不能用事务 | 以非事务方式执行，如果当前存在事务，则抛出异常。             |
| PROPAGATION_NESTED                              | 嵌套，可以部分回滚  | 如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，就新建一个事务。 |

支持当前事务：指a有事务，则b加入事务a，并在方法a的事务提交时一起提交。



#### 例子

##### 装载依赖到IOC容器

也可以在方法中`new DataSourceTransactionManager()` 和 `new DefaultTransactionDefinition`，但是依据 Spring 规范最好还是将依赖配置出去装载到 IOC 容器，再在 service 层中再注入。

```xml
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"/>  <!-- DataSourceTransactionManager基于JDBC的dataSource -->
</bean>
<bean id="transactionDefinition" class="org.springframework.transaction.support.DefaultTransactionDefinition">
    <!-- 可以查看class的父类TransactionDefinition来选择需要该的属性property -->
    <property name="propagationBehaviorName" value="PROPAGATION_REQUIRED"/>  <!-- 配置事务传播行为 -->
</bean>
```



##### service层

```java
package com.imooc.os.service.impl;
@Service
public class OrderServiceImpl implements OrderService {
    @Autowired private OrderDao orderDao;
    @Autowired private ProductDao productDao;
    @Autowired private PlatformTransactionManager transactionManager;
    @Autowired private TransactionDefinition transactionDefinition;
    
    public void addOrder(Order order) {  // 增加订单
        order.setCreateTime(new Date());
        order.setStatus("待付款");
        TransactionProxyFactoryBean transactionProxyFactoryBean;
        TransactionStatus transactionStatus = transactionManager.getTransaction(transactionDefinition);  // 开启一个事务
        try {
            orderDao.insert(order);  // 增加订单
            Product product = productDao.select(order.getProductsId());
            product.setStock(product.getStock() - order.getNumber());
            productDao.update(product);  // 修改库存
            transactionManager.commit(transactionStatus);
        }catch (Exception e){
            e.printStackTrace();
            transactionManager.rollback(transactionStatus);
}	}	}
```



### 2基于 TranssactionTemplate

#### TransactionTemplate接口

封装了 DefaultTransactionDefinition，不需要使用底层 API 手动开启和提交事务

方法：

`execute(TransactionCallback<T>):<T>` TransactionTemplate 使用 注入的PlatformTransactionManager 自动定义了事务并提交，通过 TransactionCallback 回调函数来执行业务操作操作



#### 例子

##### 装载依赖到IOC容器

```xml
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"/>  <!-- DataSourceTransactionManager基于JDBC的dataSource -->
</bean>
<bean id="transactionTemplate" class="org.springframework.transaction.support.TransactionTemplate">
    <property name="transactionManager" ref="transactionManager"/>  <!-- 指定事务管理器 -->
</bean>
```

##### service层

```java
package com.imooc.os.service.impl2;
@Service
public class OrderServiceImpl implements OrderService {
    @Autowired private OrderDao orderDao;
    @Autowired private ProductDao productDao;
    @Autowired private TransactionTemplate transactionTemplate;

    public void addOrder(final Order order) {
        order.setCreateTime(new Date());
        order.setStatus("待付款");
        transactionTemplate.execute(new TransactionCallback() {  // 执行事务
            public Object doInTransaction(TransactionStatus transactionStatus) {
                try {
                    orderDao.insert(order);
                    Product product = productDao.select(order.getProductsId());
                    product.setStock(product.getStock() - order.getNumber());
                    productDao.update(product);
                }catch (Exception e){
                    e.printStackTrace();
                    transactionStatus.setRollbackOnly();
                }
                return null;
            }
        });
}	}
```



## 声明式事务处理

* Spring的声明式事务处理是建立在 AOP 的基础之上的。其本质是对方法前后进行拦截，然后在目标方法开始之前创建或者加入一个事务，在执行完目标方法之后根据执行情况提交或者回滚事务。
* 建议在开发中使用声明式事务，是因为这样可以使得业务代码纯粹干净(对业务层代码侵入性小)，方便后期的代码维护



### 3基于TransactionInterceptor通知

通过 Spring AOP 的代理机制将事务管理逻辑织入到目标方法中。

#### TransactionInterceptor接口

`TransactionInterceptor` 拦截器是 Spring AOP 的通知(Advice)的实现，用于在方法执行过程中执行事务管理。



#### 例子

##### 装载依赖到IOC容器

```xml
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"/>  <!-- DataSourceTransactionManager基于JDBC的dataSource -->
</bean>
<bean id="orderServiceTarget" class="com.imooc.os.service.impl.OrderServiceImpl"/>
<bean id="transactionInterceptor" class="org.springframework.transaction.interceptor.TransactionInterceptor">
    <property name="transactionManager" ref="transactionManager"/>  <!-- 配置事务管理器 -->
    <property name="transactionAttributes">  <!-- 配置事务属性 -->
        <props>
            <!-- 事务属性：key是目标对象的方法名。配置传播行为，配置readOnly属性(只读属性在锁等方面得到优化) -->
            <prop key="get*">PROPAGATION_REQUIRED, readOnly</prop>
            <prop key="find*">PROPAGATION_REQUIRED, readOnly</prop>
            <prop key="search*">PROPAGATION_REQUIRED, readOnly</prop>
            <prop key="*">PROPAGATION_REQUIRED</prop>
        </props>
    </property>
</bean>
<!-- Spring AOP 配置切面 -->
<bean id="orderService" class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="target" ref="orderServiceTarget"/>  <!-- 配置目标 -->
    <property name="interceptorNames">  <!-- 配置通知 -->
        <list>  <!-- interceptorNames属性是List类型，所以需要<list>标签 -->
            <idref bean="transactionInterceptor"/>
        </list>
    </property>
</bean>
```

##### service层

```java
package com.imooc.os.service.impl;
@Service
public class OrderServiceImpl implements OrderService {
    @Autowired private OrderDao orderDao;
    @Autowired private ProductDao productDao;
    public void addOrder(Order order) {  // 添加订单
        order.setCreateTime(new Date());
        order.setStatus("待付款");
        orderDao.insert(order);  // 增加订单
        Product product = productDao.select(order.getProductsId());
        product.setStock(product.getStock() - order.getNumber());
        productDao.update(product);  // 修改库存
}	}
```



### 4基于 TransactionProxyFactoryBean切面

通过 Spring AOP 的代理机制将事务管理逻辑织入到目标方法中。

#### TransactionProxyFactoryBean接口

`TransactionProxyFactoryBean` 是 Spring AOP 的切面的实现，其中 transactionAttributes 属性等效于事务拦截器TransactionInterceptor，简化了 XML 配置



#### 例子

##### 装载依赖到IOC容器

```xml
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"/>
</bean>
<bean id="orderServiceTarget" class="com.imooc.os.service.impl.OrderServiceImpl"/>
<!-- Spring AOP 配置切面 -->
<bean id="orderService" class="org.springframework.transaction.interceptor.TransactionProxyFactoryBean">
    <property name="transactionManager" ref="transactionManager"/>
    <property name="target" ref="orderServiceTarget"/>  <!-- 配置目标 -->
    <property name="transactionAttributes">  <!-- 配置通知 -->
        <props>
            <!-- key是目标对象的方法名，readOnly配置只读属性-->
            <prop key="get*">PROPAGATION_REQUIRED, readOnly</prop>
            <prop key="find*">PROPAGATION_REQUIRED, readOnly</prop>
            <prop key="search*">PROPAGATION_REQUIRED, readOnly</prop>
            <prop key="*">PROPAGATION_REQUIRED</prop>
        </props>
    </property>
</bean>
```

##### service层

```java
package com.imooc.os.service.impl;
@Service
public class OrderServiceImpl implements OrderService {
    @Autowired private OrderDao orderDao;
    @Autowired private ProductDao productDao;
    public void addOrder(Order order) {  // 添加订单
        order.setCreateTime(new Date());
        order.setStatus("待付款");
        orderDao.insert(order);  // 增加订单
        Product product = productDao.select(order.getProductsId());
        product.setStock(product.getStock() - order.getNumber());
        productDao.update(product);  // 修改库存
}	}
```



### 5基于\<tx\>命名空间(最好的方法)

基于TransactionInterceptor或者TransactionProxyFactory中，每一个service实现都需要配置AOP。
基于<tx>：支持通配符，只需要配置一项，大大简化了配置。

#### 例子

##### 装载依赖到IOC容器

```xml
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"/>
</bean>
<!-- 配置通知 -->
<tx:advice id="txAdvice" transaction-manager="transactionManager">
    <tx:attributes>
        <tx:method name="get*" propagation="REQUIRED" read-only="true"/>
        <tx:method name="find*" propagation="REQUIRED" read-only="true"/>
        <tx:method name="search*" propagation="REQUIRED" read-only="true"/>
        <tx:method name="*" propagation="REQUIRED"/>
    </tx:attributes>
</tx:advice>
<!-- 配置切面 -->
<aop:config>
	<!-- 配置切入点(方法通配符)   -->
    <aop:pointcut id="pointcut" expression="execution(* com.imooc.os.service.impl.*.*(..))"/>
    <aop:advisor advice-ref="txAdvice" pointcut-ref="pointcut"/>
</aop:config>
```

##### service层

```java
package com.imooc.os.service.impl;
@Service
public class OrderServiceImpl implements OrderService {
    @Autowired private OrderDao orderDao;
    @Autowired private ProductDao productDao;
    public void addOrder(Order order) {  // 添加订单
        order.setCreateTime(new Date());
        order.setStatus("待付款");
        orderDao.insert(order);  // 增加订单
        Product product = productDao.select(order.getProductsId());
        product.setStock(product.getStock() - order.getNumber());
        productDao.update(product);  // 修改库存
}	}
```



### 6基于@Transaction注解(常用)

当一个方法操作好几个表的数据，并且都是修改新增操作时使用，事务是为了防止错误发生

#### 例子

##### 装载依赖到IOC容器

装载事务管理器到IOC容器，作为注解驱动

```xml
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"/>
</bean>
<tx:annotation-driven transaction-manager="transactionManager"/>
```

##### service层

```java
package com.imooc.os.service.impl6;
@Service
public class OrderServiceImpl implements OrderService {
    @Autowired private OrderDao orderDao;
    @Autowired private ProductDao productDao;

    @Transactional(propagation = Propagation.REQUIRED)
    public void addOrder(Order order) {
        order.setCreateTime(new Date());
        order.setStatus("待付款");
        orderDao.insert(order);
        Product product = productDao.select(order.getProductsId());
        product.setStock(product.getStock() - order.getNumber());
        productDao.update(product);
}	}
```



#### 注释

| 注解                  | 释义 |
| --------------------- | ---- |
| @TransactionalService |      |
| @Transactional        |      |
| @Rollback             |      |



## 总结

看156就好。
声明式事务处理对业务代码侵入性更小，事务拦截器只能针对一个目标类，<tx>模糊匹配会误伤，所以最佳实践是使用基于 `@Transaction` 注解的方式。



