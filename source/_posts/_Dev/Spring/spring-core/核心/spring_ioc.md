---
title: Spring IOC
date: 2023-07-13
update: 2023-08-17

tags:
categories:
  - Spring
  - Spring Core
  - 核心
keywords: Spring,SpringCore,IOC
description: 关于SpringCore中IOC的底层实现
top_img:
cover: https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_07_12_18_18.png" alt="image-20230712181851670
---



# Spring IOC

## IOC

IOC有什么优势？需要扫描所有依赖查是否有对应注解，很蠢



[好文章，分析了各种方式的利弊](https://martinfowler.com/articles/injection.html)

传统的编程模型中，对象通过直接实例化其他对象来满足其依赖关系。
控制反转是指将对象创建和依赖管理的控制权从程序转移到外部容器中，由容器负责对象创建和依赖管理，对象需要遵循一些约定。
只需要关注依赖注入方式和依赖项

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_09_18_00_23.png" alt="image-20230918002300866" style="zoom:67%;" />

<center>在lister类中使用简单创建的依赖关系</center>

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_09_18_00_24.png" alt="image-20230918002405627" style="zoom:50%;" />

<center>依赖注入后的依赖关系(Assembler组装)</center>



## DI

依赖注入是指将依赖项注入到对象中，依赖注入是控制反转的实现。



### 构造方法注入(最佳实践)

实现特定参数的构造方法，通过配置将依赖项注入到对象的构造方法参数

示例：PicoContainer

```java
class MovieLister {
	public MovieLister(MovieFinder finder) {
		this.finder = finder;       
}	}
```

```java
class ColonMovieFinder {
	public ColonMovieFinder(String filename) {
		this.filename = filename;
}	}
```

```java
private MutablePicoContainer configureContainer() {
	MutablePicoContainer pico = new DefaultPicoContainer();
    Parameter[] finderParams =  {new ConstantParameter("movies1.txt")};
    pico.registerComponentImplementation(MovieFinder.class, ColonMovieFinder.class, finderParams);
    pico.registerComponentImplementation(MovieLister.class);
    return pico;
}
```



### Setter 方法注入

实现特定参数的 Setter 方法，通过配置将依赖项注入到对象的 Setter 方法参数

```java
class MovieLister {
	private MovieFinder finder;
	public void setFinder(MovieFinder finder) {
  		this.finder = finder;       
}	}
```

```java
class ColonMovieFinder {
	public void setFilename(String filename) {
		this.filename = filename;
}	}
```

```xml
<beans>
    <bean id="MovieLister" class="spring.MovieLister">
        <property name="finder">
            <ref local="MovieFinder"/>
        </property>
    </bean>
    <bean id="MovieFinder" class="spring.ColonMovieFinder">
        <property name="filename">
            <value>movies1.txt</value>
        </property>
    </bean>
</beans>
```

```java
public void testWithSpring() throws Exception {
    ApplicationContext ctx = new FileSystemXmlApplicationContext("spring.xml");
    MovieLister lister = (MovieLister) ctx.getBean("MovieLister");
    Movie[] movies = lister.moviesDirectedBy("Sergio Leone");
    assertEquals("Once Upon a Time in the West", movies[0].getTitle());
}
```



### 接口注入



## BeanDefinition

BeanDefinition类 - 描述一个 bean 实例，它具有属性值、构造函数参数值以及具体实现的更多信息

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_07_12_18_18.png" alt="image-20230712181851670" style="zoom:67%;" />

SpringApplication.refresh() 调用 AbstractApplicationContext.refresh() 调用 AbstractApplicationContext.obtainFreshBeanFactory() 调用 AbstractXMLApplicationContext.refreshBeanFactory() 调用 BeanDefinitionReader.**loadBeanDefinitions() 加载并解析 Bean配置信息(XML、Configuration类、注解) 为 BeanDefinition**

DefaultListableBeanFactory.**registerBeanDefinition() 注册 BeanDefinition 到 BeanFactory** 

AbstractBeanFactory.getBean() 调用 **AbstractAutowireCapableBeanFactory.createBean() 通过 Java 反射机制创建 Bean 实例**



## BeanDefinitionRegistry

BeanDefinitionRegistry接口 - 用于注册持有的 Bean Definition 实例。Spring core 中实现类为 DefaultListableBeanFactory 

`registerBeanDefinition(String, BeanDefinition): void` 注册一个新的 BeanDefinition 到注册表。DefaultListableBeanFactory  实现了该接口 将 BeanDefinition 存储到 beanDefinitionMap<String, BeanDefinition> 中。



## BeanFactory

<center><img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_06_20_16_11.png" alt="image-20230620161151340" style="zoom: 35%;" /><img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_07_12_14_50.png" alt="Snipaste_2023-07-12_14-48-47 (1)" style="zoom:46%;" /></center>

![BeanFactory](https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting2/img/2023_09_19_12_58.png)

BeanFactory接口 - 用于访问 Spring bean 容器的根接口，是应用组件的中心注册表。
通过 BeanFactory 接口和其子类实现了 Spring 的 DI 功能。
BeanFactory **加载**存储在配置源的 **Bean Definition**，再使用 org.springframework.beans 来**配置 Bean**，**初始化 Bean**
如果在一个工厂实例中找不到 Bean，会再询问父工厂

HierarchicalBeanFactory接口 - 管理bean工厂层次结构(在未知bean的情况下委托给父级)



### AbstractBeanFactory

AbstractBeanFactory 类 - 提供 ConfigurableBeanFactory SPI 的全部功能。其实现有 DefaultListableBeanFactory、AbstractAutowireCapableBeanFactory



#### `getBean(String):Object`

`getBean(String):Object` 获取 Bean 实例

```java
// AbstractBeanFactory.java
public Object getBean(String name) throws BeansException {
    return doGetBean(name, null, null, false);
}
```

```java
// AbstractBeanFactory.java
protected <T> T doGetBean(
        String name, @Nullable Class<T> requiredType, @Nullable Object[] args, boolean typeCheckOnly)
        throws BeansException {

    String beanName = transformedBeanName(name);  // 1去掉&Beanname的工厂引用前缀&
    Object beanInstance;

    // 急检查单例缓存
    Object sharedInstance = getSingleton(beanName); // 2从单例缓存ConcurrentHashMap<String,Object>获取Bean实例，若存在则返回
    if (sharedInstance != null && args == null) {
        ...		
        beanInstance = getObjectForBeanInstance(sharedInstance, name, beanName, null);  // &FactoryBean创建其他Bean
    }

    else {
        // 如果已经创建该实例而失败，我们大概再循环引用
        if (isPrototypeCurrentlyInCreation(beanName)) {
            throw new BeanCurrentlyInCreationException(beanName);
        }

        // 3检查 Bean Definition 是否存在在父工厂中(委托给父工厂)，若存在则返回
        BeanFactory parentBeanFactory = getParentBeanFactory();
        if (parentBeanFactory != null && !containsBeanDefinition(beanName)) {
            // Not found -> check parent.
            String nameToLookup = originalBeanName(name);
            if (parentBeanFactory instanceof AbstractBeanFactory abf) {
                return abf.doGetBean(nameToLookup, requiredType, args, typeCheckOnly);
            }
            ...
        }

        if (!typeCheckOnly) {
            markBeanAsCreated(beanName);
        }

        StartupStep beanCreation = this.applicationStartup.start("spring.beans.instantiate")
                .tag("beanName", name);
        try {
            if (requiredType != null) {
                beanCreation.tag("beanType", requiredType::toString);
            }
            RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);
            checkMergedBeanDefinition(mbd, beanName, args);

            // 4保证当前bean所依赖的bean的初始化
            String[] dependsOn = mbd.getDependsOn();
            if (dependsOn != null) {
                for (String dep : dependsOn) {
                    if (isDependent(beanName, dep)) {
                        throw new BeanCreationException(mbd.getResourceDescription(), beanName,
                                "Circular depends-on relationship between '" + beanName + "' and '" + dep + "'");
                    }
                    registerDependentBean(dep, beanName);
                    try {
                        getBean(dep);
                    }
                    catch (NoSuchBeanDefinitionException ex) {
                        throw new BeanCreationException(mbd.getResourceDescription(), beanName,
                                "'" + beanName + "' depends on missing bean '" + dep + "'", ex);
            }	}	}

            // 5创建Bean实例
            if (mbd.isSingleton()) {  // 5.1若Bean的作用域为Singleton单例，查看先前是否创建过。若创建过则直接返回，若没创建过则创建一个
                sharedInstance = getSingleton(beanName, () -> {
                    try {
                        return createBean(beanName, mbd, args);
                    }
                    catch (BeansException ex) {                        
                        destroySingleton(beanName);  // 显示删除实例从单例缓存，创建过程会急放实例到单例缓存，以允许循环引用的解析。还要移除任何收到的对Bean的临时引用。
                        throw ex;
                    }
                });
                beanInstance = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
            }

            else if (mbd.isPrototype()) {  // 5.2若Bean的作用域为原型，则创建一个新实例
                // It's a prototype -> create a new instance.
                Object prototypeInstance = null;
                try {
                    beforePrototypeCreation(beanName);
                    prototypeInstance = createBean(beanName, mbd, args);
                }
                finally {
                    afterPrototypeCreation(beanName);
                }
                beanInstance = getObjectForBeanInstance(prototypeInstance, name, beanName, mbd);
            }

            else {  // 5.3若是其他作用域(web容器还有session request作用域)
                String scopeName = mbd.getScope();
                if (!StringUtils.hasLength(scopeName)) {
                    throw new IllegalStateException("No scope name defined for bean '" + beanName + "'");
                }
                Scope scope = this.scopes.get(scopeName);
                if (scope == null) {
                    throw new IllegalStateException("No Scope registered for scope name '" + scopeName + "'");
                }
                try {
                    Object scopedInstance = scope.get(beanName, () -> {
                        beforePrototypeCreation(beanName);
                        try {
                            return createBean(beanName, mbd, args);
                        }
                        finally {
                            afterPrototypeCreation(beanName);
                        }
                    });
                    beanInstance = getObjectForBeanInstance(scopedInstance, name, beanName, mbd);
                }
                catch (IllegalStateException ex) {
                    throw new ScopeNotActiveException(beanName, scopeName, ex);
        }	}	}
        catch (BeansException ex) {
            beanCreation.tag("exception", ex.getClass().toString());
            beanCreation.tag("message", String.valueOf(ex.getMessage()));
            cleanupAfterBeanCreationFailure(beanName);
            throw ex;
        }
        finally {
            beanCreation.end();
        }
    }

    return adaptBeanInstance(name, beanInstance, requiredType);  // 6对Bean实例检查类型是否匹配
}
```

```java
// AbstractBeanFactory.java
protected abstract Object createBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args)
			throws BeanCreationException;
```

```java
// AbstractAutowireCapableBeanFactory.java
protected Object createBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args)
        throws BeanCreationException {

    if (logger.isTraceEnabled()) {
        logger.trace("Creating instance of bean '" + beanName + "'");
    }
    RootBeanDefinition mbdToUse = mbd;

    // Make sure bean class is actually resolved at this point, and
    // clone the bean definition in case of a dynamically resolved Class
    // which cannot be stored in the shared merged bean definition.
    Class<?> resolvedClass = resolveBeanClass(mbd, beanName);
    if (resolvedClass != null && !mbd.hasBeanClass() && mbd.getBeanClassName() != null) {
        mbdToUse = new RootBeanDefinition(mbd);
        mbdToUse.setBeanClass(resolvedClass);
    }

    // Prepare method overrides.
    try {
        mbdToUse.prepareMethodOverrides();
    }
    catch (BeanDefinitionValidationException ex) {
        throw new BeanDefinitionStoreException(mbdToUse.getResourceDescription(),
                beanName, "Validation of method overrides failed", ex);
    }

    try {
        // Give BeanPostProcessors a chance to return a proxy instead of the target bean instance.
        Object bean = resolveBeforeInstantiation(beanName, mbdToUse);
        if (bean != null) {
            return bean;
        }
    }
    catch (Throwable ex) {
        throw new BeanCreationException(mbdToUse.getResourceDescription(), beanName,
                "BeanPostProcessor before instantiation of bean failed", ex);
    }

    try {
        Object beanInstance = doCreateBean(beanName, mbdToUse, args);
        if (logger.isTraceEnabled()) {
            logger.trace("Finished creating instance of bean '" + beanName + "'");
        }
        return beanInstance;
    }
    catch (BeanCreationException | ImplicitlyAppearedSingletonException ex) {
        // A previously detected exception with proper bean creation context already,
        // or illegal singleton state to be communicated up to DefaultSingletonBeanRegistry.
        throw ex;
    }
    catch (Throwable ex) {
        throw new BeanCreationException(
                mbdToUse.getResourceDescription(), beanName, "Unexpected exception during bean creation", ex);
    }
}
```

createBean() 调用 doCreateBean() 调用 initializeBean()

```java
// AbstractAutowireCapableBeanFactory
protected Object initializeBean(String beanName, Object bean, @Nullable RootBeanDefinition mbd) {  // 初始化给定的 bean 实例
    invokeAwareMethods(beanName, bean);  // 2. 调用Aware接口的方法(注入BeanName、BeanFactory)，令Bean能感知(意识到)到Spring容器，从而能对容器进行操作

    Object wrappedBean = bean;
    if (mbd == null || !mbd.isSynthetic()) {
        wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);  // 3. 在Bean初始化之前，对Bean添加自定义的处理逻辑
    }

    try {
        invokeInitMethods(beanName, wrappedBean, mbd);  // 4. Bean如果实现了InitializingBean接口，则调用响应的回调方法。再执行Bean中定义的init()方法。
    }
    catch (Throwable ex) {
        throw new BeanCreationException(
                (mbd != null ? mbd.getResourceDescription() : null), beanName, ex.getMessage(), ex);
    }
    if (mbd == null || !mbd.isSynthetic()) {
        wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);  // 5. 在Bean初始化之后，对Bean添加自定义的处理逻辑
    }

    return wrappedBean;
}
```



#### AbstractAutowireCapableBeanFactory

DefaultListableBeanFactory类是 BeanFactory 接口的默认实现类。提供 Bean 创建(带有构造函数解析)、属性填充、装配(包括自动装配)和初始化。支持自动装配构造函数、按名称的属性、按类型的属性。

`createBean(Class<?>, int, boolean):Object` 见上面

##### 初始化Bean

`initializeBean(String,Object,RootBeanDefinition):Object` 初始化 Bean

Bean 初始化方法的标准顺序

**创建 Bean 实例，并依赖注入属性**

**调用 Aware 接口的方法** 
注入BeanName、BeanFactory
通过Aware接口Bean能感知(意识到)到Spring容器，从而能对容器进行操作

1. BeanNameAware's `setBeanName`
   BeanClassLoaderAware's `setBeanClassLoader`
   BeanFactoryAware's `setBeanFactory`
   EnvironmentAware's `setEnvironment`
   EmbeddedValueResolverAware's `setEmbeddedValueResolver`
   ResourceLoaderAware's `setResourceLoader` (only applicable when running in an application context)
   ApplicationEventPublisherAware's `setApplicationEventPublisher` (only applicable when running in an application context)
   MessageSourceAware's `setMessageSource` (only applicable when running in an application context)
   ApplicationContextAware's `setApplicationContext` (only applicable when running in an application context)
   ServletContextAware's `setServletContext` (only applicable when running in a web application context)

**`BeanPostProcessor(s).postProcessBeforelnitialization()`** 
在Bean初始化之前，对Bean添加自定义的处理逻辑。

2. `postProcessBeforeInitialization` methods of BeanPostProcessors

**初始化 Bean**

`InitializingBean(s).afterPropertiesSet()` 
此时 Bean 所有属性都已设置，Bean 如果实现了 **InitializingBean 接口**，则调用响应的回调方法

3. InitializingBean's `afterPropertiesSet`

执行 Bean 中定义的 **`init()` 方法**

4. a custom `init-method` definition

**`BeanPostProcessor(s).postPcocessAfterInitialization()`** 
在Bean初始化之后，对Bean添加自定义的处理逻辑 (为给定的 Bean 创建 AOP 代理)
AbstractAutoProxyCreator 实现了BeanPostProcessor

5. `postProcessAfterInitialization` methods of BeanPostProcessors

**Bean 初始化完毕**

```java
@Override public Object postProcessAfterInitialization(final Object bean, String beanName) throws BeansException {
    System.out.println("执行BeanPostProcessor的postProcessAfterInitialization方法，返回：" + bean + "beanName:" + beanName);
    if ("userDao".equals(beanName)) {
        // 动态代理
        Object proxy = Proxy.newProxyInstance(bean.getClass().getClassLoader(), bean.getClass().getInterfaces(), new InvocationHandler() {
            // 这里是重写代理的方法，如果方法名为xxx，那么xxx，再返回代理后的方法
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                if ("save".equals(method.getName())) {
                    System.out.println("代理Bean的save方法");
                    return method.invoke(bean, args);
                }
                return method.invoke(bean, args);
            }
        });
        System.out.println("返回代理");
        return proxy;
    } else {
        System.out.println("返回Bean");
        return bean;
}	}
```



##### 销毁Bean

`destoryBean(Object):void`

* 若实现了 DisposableBean 接口，则会调用 destroy() 方法
* 若 Bean 配置了 destry-method 属性，则会调用其配置的销毁方法



##### Bean的生命周期

1. instantiate bean对象实例化constructor
2. populate properties 封装属性 
3. 如果Bean实现BeanNameAware 执行 setBeanName
4. 如果Bean实现BeanFactoryAware 或者ApplicationContextAware 设置工厂 setBeanFactory 或者上下文对象 setApplicationContext
5. (关键)如果存在类实现 BeanPostProcessor，则执行postProcessBeforeInitialization，该方法返回Object 即bean
6. 如果Bean实现InitializingBean 执行afterPropertiesSet方法
7. 调用<bean init-method="init"> 指定初始化方法init
8. (关键)如果存在类实现 BeanPostProcessor（后处理Bean），执行postProcessAfterInitialization，该方法返回Object 即bean

重点  这里是在所有Bean之前都注入完后才执行第一个run方法的哦(主按xml顺序次按ref依赖顺序注入)

9. 执行业务处理run方法
10. 如果Bean实现DisposableBean(一次性Bean) 执行destroy方法
11. 调用<bean destroy-method="customerDestory"> 指定指定的销毁方法





##### DefaultListableBeanFactory

DefaultListableBeanFactory类是 BeanFactory 接口的默认实现类。能以 List 的形式返回 BeanDefinition 信息

```java
// DefaultListableBeanFactory.java
private final Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap<>(256);  //key为BeanName
private volatile List<String> beanDefinitionNames = new ArrayList<>(256);  // 按注册顺序
```

`getBeanDefinition(String):BeanDefinition`

`getBean(Class<T>):T` 底层都是调用 `AbstractBeanFactory.getBean(String)` 来实例化 Bean 的



## ApplicationContext

ApplicationContext接口 - 是为应用程序提供配置的中央接口

继承了 BeanFactory 接口：能够管理、装配Bean
继承了 ResourcePatternResolver：能够加载资源文件
继承了 MessageSource：能够实现国际化等功能
继承了 Application EventPublisher：能够注册监听器，能够 实现监听机制

[SpringBoot笔记#SpringApplication](../../spring-boot/spring_boot3.2.0.md#SpringApplication)



### ClassPathXmlApplicationContext

加载类路径下的 XML 配置文件

```java
ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");
UserService userService = (UserService) applicationContext.getBean("userService");
```



### FileSystemXmlApplicationContext

加载文件系统下的 XML 配置文件



### AnnotationConfigApplicationContext

加载注解配置文件，是默认的 IOC 容器

#### 示例

```java
ApplicationContext ctx = new AnnotationConfigApplicationContext(TestConfiguration.class);
// 常用方法
ctx.register(AppConfig.class);
ctx.refresh();
MyBean myBean = ctx.getBean(MyBean.class);
```





## 注解

[API 文档](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/package-summary.html)

### 注册

#### 组件扫描+@Component

| 注解                                                         | 元注解Target | 声明组件的职责，只起到标识作用                               | 成员变量                                                     |
| ------------------------------------------------------------ | ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `@ComponentScan(value)`                                      | TYPE         | 扫描指定包中的注解组件<br />如果未指定特定包，则将从声明此注释的类的包中进行扫描 | 要扫描的含注解组件的包                                       |
| `@Component(value)`                                          | TYPE         | 标识该类为组件，在使用基于注释的配置和类路径扫描时，这些类实例对象将被视为组件扫描的候选对象，需要被扫描装配到 IOC 容器中<br />**默认使用无参构造函数** | **指定 Bean 的 name，默认 name 为首字母小写的类名**<br />推荐当存在候选Bean时指定名称为 clsname#itfn |
| `@Repository(value)`                                         | TYPE         | DAO层实现类...                                               |                                                              |
| `@Service(value)`                                            | TYPE         | Service层实现类...                                           |                                                              |
| `@Controller(value)`                                         | TYPE         | Controller层实现类...                                        |                                                              |
| `@Configuration(value)`                                      |              | JavaConfig类...                                              |                                                              |
| @ContextConfiguration("c<br />lasspath:applicationContext3.xml") |              |                                                              |                                                              |



#### 配置类+@Bean

当使用第三方库中的某个方法，需要把该方法的对象注册到 Spring 容器中。此时只能使用 `JavaConfig 类+@Bean`
`组件扫描+@Component`

| **Configration类**          | 元注解Target                | 声明Bean配置信息，触发自动配置和组件扫描                     | 成员变量                                                     |
| --------------------------- | --------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `@Bean(value)`              | METHOD<br />ANNOTATION_TYPE | 表明一个方法生成一个由 Spring 容器管理的 Bean                | **指定 Bean 的  name，默认 name 为@Bean方法名称**<br />name 接受一组字符串指定多个别名<br />destroyMethod 在关闭 ApplicationContext 期间调用的方法名，例如 close() |
| **条件注解**                | **元注解Target**            | **释义**                                                     | **成员变量**                                                 |
| `@Conditional`              | TYPE、@Bean方法             | <br />自定义条件注解，表示该组件在所有 Condition classes 匹配后才有资格注册到 IOC 容器 | `value():Class<? extends Condition>[]`，xxCondition.class 实现 Condition 接口，检查是否符合条件 |
| `@ConditionalOnProperty`    | TYPE、@Bean方法             | 成品条件注解，用于检查指定 property 是否具有特定值           | `name():String[]`，要检查的 property 名称<br />`havingValue():String` 预期值<br />`matchIfMiss():boolean` 如果属性未设置，条件是否匹配 |
| `@ConditionalOnBean`        | TYPE、@Bean方法             | 当 BeanFactory 中包含符合要求的 Bean 时匹配。强烈建议仅在自动配置类中使用该注解 | `value():Class<?>[]`，Bean的类型名<br />`name():String[]`，Bean的名称 |
| `@ConditionalOnMissingBean` | TYPE、@Bean方法             | 当 BeanFactory 中不包含符合要求的 Bean 时匹配。强烈建议仅在自动配置类中使用该注解 | `value():Class<?>[]`，Bean的类型名<br />`name():String[]`，Bean的名称 |



##### Bean 属性

| 注解            | 元注解Target                                  | 释义                                                         | 成员变量                                                  |
| --------------- | --------------------------------------------- | ------------------------------------------------------------ | --------------------------------------------------------- |
| `@Scope(value)` | TYPE、METHOD                                  | 表明使用 Bean 实例的范围                                     | 作用域的名字                                              |
| `@Lazy`         | 直接或间接的 `@Component` 类或用 `@Bean` 方法 | 表明 Bean 是否要延迟实例化<br />单例实例是在创建 applicationContext 时创建的是急初始化<br />@Lazy注解只在Bean的单例作用域下有效 | boolean，默认为默true懒加载                               |
| `@Primary`      | 直接或间接的 `@Component` 类或 `@Bean` 方法   | 表明一个 Bean 应该优先，当多个候选者都有资格自动装配一个单值依赖时若存在一个 primary Bean则该Bean为自动装配的值 |                                                           |
| `@Order`        | TYPE、METHOD、FIELD                           | 定义被 @Order 注解的同一类型或接口的排序顺序，使数组或列表中元素按特定顺序排序<br />默认为 BeanDefinition 注册顺序 | 顺序值，值越低优先级越高，默认为Ordered.LOWEST_PRECEDENCE |
| `@Profile()`    | TYPE、METHOD                                  | 当指定的配置文件处于活动状态，则有资格注册该组件。激活配置文件 application-{String}.properties：`SpringApplicationBuilder.profiles(String)` | 配置文件名                                                |
| @DependsOn      |                                               | 强制在创建此Bean之前创建特定的其他Bean                       |                                                           |

| Bean 作用域                |                                                              |
| -------------------------- | ------------------------------------------------------------ |
| singleton单例              | 默认，适合**无状态 Bean**<br />在创建 applicationContext 时创建单例实例，在项目中只会创建一次 |
| prototype原型              | 适合有状态 Bean<br />在每次原型注册表 beanFactory.getBean() 时都会生成一个拷贝 Bean 实例<br />[设计模式笔记#原型模式](../../../design-patterns/创建型/原型模式.md) |
| **WebApplicationContext:** |                                                              |
| request                    | HTTP请求                                                     |
| session                    | HTTP会话                                                     |
| application                |                                                              |
| **Spring Cloud:**          |                                                              |
| refresh                    | 允许 Bean 在运行时被动态刷新                                 |

###### 单例Bean在多线程环境下的线程安全问题

单例Bean应该尽量设计为无状态的，避免在单例Bean中引入实例变量以存储状态。
如果有必要存储状态，确保使用线程安全的数据结构或同步机制来保证多线程访问的安全性。



无状态的

```java
@Component
public class SingletonBean {
    public static void addData(List<String> data, String item) {  // 局部变量不存在线程安全问题
        data.add(item);
    }
    public static List<String> getData(List<String> data) {
        return new ArrayList<>(data);
}	}
```

有状态的

```java
@Component
public class SingletonBean {
    private List<String> data = new ArrayList<>();  // 实例变量存在线程安全问题
    public void addData(String item) {
        data.add(item);
    }
    public List<String> getData() {
        return data;
}	}
```

解决1：[Java线程笔记#ThreadLocal](../../../Java/类库APIs/Java线程.md#ThreadLocal)
解决2：使用线程安全的数据结构、使用同步机制-锁



### 依赖注入

原理：@Autowired、@Inject、@Value 和 @Resource 注释通过 `BeanPostProcessor ` 处理，这意味着 BeanPostProcessor 无法自动装配。

| 注解                   | 元注解Target                                                 | 释义                                                         | 成员变量                                                     |
| ---------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `@Value`               | FIELD<br />setter方法、PARAMETER                             | 外部配置(properties/yml)单个属性注入，该注解实际由BeanPostProcessor处理执行 | `value():String`，<br />可以直接写字符串，还支持SpEL #{}或属性占位符${my.app.myProp} |
| `@PropertySource`      | @Configration类                                              |                                                              | `value():String[]`，指明要加载的属性文件的资源位置，例如"classpath:/com/myco/app.properties" |
| `@Autowired(required)` | CONSTRUCTOR<br />METHOD<br />PARAMETER<br />FIELD<br />ANNOTATION_TYPE | **机制：**配合required成员变量、@Qualifier@Primary约束进行DI，先byType策略若无法确定唯一依赖关系再byName策略<br /><br />**相似注解：**JSR-@Resource、JSR-@Inject<br /><br />**使用方式：**自动装配[构造函数(最佳实践)\|字段(最方便，通过反射实现)\|方法(setter方法、config方法)\|参数(只在 spring-test 中支持)]。**使用时需要确保能找到唯一依赖项**<br /><br />自动装配Person[]、List<Person>、Map<String,Person>对象时，会byType注入所有Person对象，其中Map的键为注入对象的Name<br /><br />自动装配的**内置Bean**：`BeanFactory` 、 `ApplicationContext` 、 `Environment` 、 `ResourceLoader` 和 `MessageSource`<br /><br />**@Autowired与@Resouce的区别？**<br />1来源不同(@Resouce字段方式自动装配可以不耦合Spring) 2策略不同 3@Autowired支持构造方法 4成员变量不同(@Autowired支持required(@Resouce找不到匹配Bean会返回Null)、@Resouce支持name和type)<br /><br />**为什么不推荐FIELD方式注入？**<br />因为1无法在容器外部new实例化FIELD方式自动装配的类(因为容器外部属性缺失) 2无法注入静态、final变量 | required，默认true，表示该依赖是否必须，编译时找不到依赖项会报错<br />构造函数自动装配：required=true只能注解单个构造函数，required=false多个构造函数声明该注解时将选择具有最大依赖关系的构造函数 |
| `@Qualifier()`         | FIELD<br />METHOD<br />PARAMETER<br />TYPE<br />ANNOTATION_TYPE | 多目标的 Bean 通过类名和限定符匹配<br />1自动装配时，此注释可以用在字段或参数上，作为候选 Bean 的限定符<br />2可以作为自定义注释的元注解，使用自定义注释作为限定符 | Bean ID，例如xxxid#myBean                                    |
| `@Configurable`        | TYPE                                                         | 标记该类具有Spring 驱动配置资格<br />一般配合 AspectJ 使用   | autowire = Autowire.BY_TYPE，默认No                          |
| **Jakarta EE**         |                                                              | **不推荐使用**                                               |                                                              |
| @PostConstruct         | METHOD                                                       | 相当于xml中的init-method，当Bean初始化时执行。               |                                                              |
| @PreDestroy            | METHOD                                                       | 相当于xml中的destory-method，当Bean销毁时执行。              |                                                              |
| @Resource              | TYPE<br />FIELD<br /> METHOD                                 | JSR-250<br /><br />**机制：**配合@Qualifier约束进行DI，先byName策略若无法确定唯一依赖关系再byType策略 | name，默认是[FIELD字段名称\|METHOD方法对应的JavaBean属性名称]<br />type，默认是[FIELD字段类型\|METHOD方法对应的JavaBean属性类型] |
| @Inject                |                                                              | JSR-330没有使用的必要                                        |                                                              |



#### 字段注入和构造方法注入

[Intellij IDEA中Mybatis Mapper自动注入警告的6种解决方案](https://www.imooc.com/article/287865)

[Spring开发者Oliver Drotbohm博客#Why field injection is evil2014](https://odrotbohm.de/2013/11/why-field-injection-is-evil/)
字段注入缺点：不安全(构造方法中能添加断言更安全)、测试复杂(在容器之外需要反射注入)
构造方法注入缺点：样板构造方法多，有循环依赖问题

[Spring博客#setter注入比较构造方法注入2007](https://spring.io/blog/2007/07/11/setter-injection-versus-constructor-injection-and-the-use-of-required)
Setter注入用于非必须依赖，构造方法注入用于必需依赖



最佳实践：final构造方法注入，使用Lombok还能减少样板代码，非必须依赖Setter注入

```java
@RequiredArgsConstructor(onConstructor = @__(@Autowired))
```



#### 自动装配顺序

@Autowired自动装配问题 - 存在多个匹配的候选 Bean
解决1：@Autowired + @Qualifier
解决2：@Autowired + @Primary
解决3：@Resource

@Autowired自动装配顺序：根据类型 byType 匹配目标类型及其子类
@Autowired+@Qualifier自动装配顺序：根据限定符 byName 匹配 Bean name
@Resource自动装配顺序：根据限定符 byName 匹配 Bean name，在byType



### 简单示例

```java
@Component("person")
public class Person {
    @Value("1") private Long id;  // 字段注入
	@Value("Jack") private String name;  // 字段注入
    private Pet pet;
    @Autowired  // 构造方法注入
    public Person(Pet pet) {
        this.pet = pet;
}	}
```

```java
@Primary
@Component
public class Dog implements Pet {
    public void move() {
        System.out.println("running");
}	}

@Component
public class Bird implements Pet {
    public void move() {
        System.out.println("flying");
}	}
```

```java
// @ComponentScan("other-package")
@SpringBootApplication
public class FrameworkApplication {
    public static void main(String[] args) {
        ApplicationContext ctx = SpringApplication.run(FrameworkApplication.class, args);
        Person person = ctx.getBean(Person.class);        
        person.move();
}	}
```

```运行
running
```



## XML

springconfig.xml

```xml
<bean id="userService" class="com.imooc.ioc.demo1.UserServiceImpl" init-method="" destory-method="">
    <constructor-arg name="id" value=1/>  // 构造方法注入
	<property name="name" value="李四"/>
</bean>
```

**通过解析xml文件，获取到id属性和class属性里面的内容，利用反射原理获取到配置里面类的实例对象，存入到Spring的bean容器中。**



## XML和注解注入的优劣势

XML优点：集中
XML缺点：XML配置麻烦

注解优势：便捷
注解缺点：分散，不知道哪些Bean交给Spring管理了

最佳实践：使用@ComponentScan开启组件扫描，使用@Autowired、@Value负责依赖注入

