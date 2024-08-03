---
title: Spring Boot
date: 2023-08-24
update: 2023-08-24

tags:
categories:
  - Spring
  - Spring Boot
keywords: Spring Boot
description: 主要关于Spring Boot的启动流程和自动配置原理
top_img:
cover:
---



# Spring Boot

[Reference文档](https://docs.spring.io/spring-boot/docs/current/reference/html/)	|	[参考文档(中文)](https://springdoc.cn/spring-boot/index.html)	|	[API 文档][https://docs.spring.io/spring-boot/docs/current/api/]	|	[源码](https://github.com/spring-projects/spring-boot)

```bash
Modules
├──spring-boot							# 提供了核心特性的支持
├──spring-boot-autoconfigure
|		└──@EnableAutoConfiguration  	# 注释会Spring上下文的自动装配
├──spring-boot-starters  				# 依赖描述符
├──spring-boot-actuator
├──spring-boot-actuator-autoconfigure
├──spring-boot-test
├──spring-boot-test-autoconfigure
├──spring-boot-loader
└──spring-boot-devtools  				# 提供了额外的开发时功能
```



## 版本

Spring Boot 3.0，Spring Boot 2.7~2.4 和 2.4 以下版本之间变化较大

[github spring boot wiki#release-notes](https://github.com/spring-projects/spring-boot/wiki#release-notes)



## Spring Boot 核心特性

* 可独立运行的应用程序
  uber jar(超级jar包)：是将应用程序的所有依赖项打包到一个独立的 JAR 文件中，使得应用程序可以独立运行，而无需依赖外部的类路径或其他依赖项管理机制，自动部署。优点：分布式部署方便
  War包：传统 Web 项目需要手动部署，使用 war 包在 Web 容器中启动。
  通过 SpringApplicationBuilder 或 SpringApplication 静态方法可以直接创建 SpringApplication 实例来直接启动应用程序。
* 嵌入式 Web 容器
* 约定大于配置
  简化了设置组件参数
* 自动配置(装配) Spring 和第三方库(starter)
  在 Spring Boot 中，采用约定优于配置的方式，根据类路径、包名、注解等自动装载 Bean。自动装载主应用程序类所在包、配置类所在包、@Component、第三方库中的类，无需显式配置 `@ComponentScan`。
* 运行时应用监控：服务器压力、内存占用、数据库负载



## SpringApplication

SpringApplication 类用于从 Java Main方法引导和运行 Spring 应用。

引导步骤：

1. **创建一个 ApplicationContext 实例**(依赖你的类路径)
   [Spring IOC笔记#ApplicationContext](../spring-core/核心/spring_ioc.md#ApplicationContext)

* 注册一个 CommandLinePropertySource 以暴露命令行参数作为 Spring properties，用于将命令行参数作为值注入到 Spring bean 中
  需要将 CommandLinePropertySource 添加到 ApplicationContext 的 Environment

* **刷新 ApplicationContext**，加载所有单例 Bean

- 触发所有实现了 CommandLineRunner 函数式接口的 Bean，用于在应用程序启动后，接收流量之前执行一些初始化任务

```java
@SpringBootApplication
public class ProjectnameApplication {
    public static void main(String[] args) {
        ConfigurableApplicationContext ctx = SpringApplication.run(ProjectnameApplication.class, args);
}	}
```

方法：

`SpringApplication.run(Class<?>, String...):ConfigurableApplicationContext`



### run() - Spring Boot 启动流程

1. **创建 SpringApplication 实例，再执行 run 方法**
2. **创建 Spring 应用运行监听器** SpringApplicationRunListeners
   自动配置 spring.factories 中的 SpringApplicationRunListeners 注册到 IOC 容器。
   Spring 应用进行到某一阶段时会广播通知所有的监听器，监听器的方法被回调执行
3. **创建和配置环境** prepareEnvironment
   **加载系统环境变量、配置文件application.properties**
4. **创建应用上下文** createApplicationContext
   None、Servlet、Reactive
5. prepareContext
   **解析 Bean 配置信息为 Bean Definition**
6. refreshContext
   **实例化 Bean，并注册到 IOC 容器**
   自动配置/装配、加载组件(组件扫描+DI，加载`@Component`、`@Entity`)、应用初始化(启动Web容器、初始化日志组件、数据源、连接池、数据池)
7. afterRefresh
   **在上下文刷新后调用afterRefresh，空方法留给开发者自定义子类去实现**



```java
// ProjectNameApplication.java
@SpringBootApplication
public class ProjectNameApplication {
    public static void main(String[] args) {  // SpringBoot项目入口
        ApplicationContext ctx = SpringApplication.run(ProjectNameApplication.class, args);        
}	}
```

```java
// SpringApplication.java
public static ConfigurableApplicationContext run(Class<?>[] primarySources, String[] args) {
    return (new SpringApplication(primarySources)).run(args);
}
```

```java
// SpringApplication.java
public SpringApplication(Class<?>... primarySources) {
    this(null, primarySources);
}
```

```java
// SpringApplication.java
public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {  // 创建 SpringApplication 实例，应用程序上下文从主要来源primarySources(当前启动类包)加载 Bean
    this.resourceLoader = resourceLoader;
    Assert.notNull(primarySources, "PrimarySources must not be null");
    this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
    this.webApplicationType = WebApplicationType.deduceFromClasspath();
    this.bootstrapRegistryInitializers = new ArrayList<>(
            getSpringFactoriesInstances(BootstrapRegistryInitializer.class));
    setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class));  // 获取所有的配置在spring.factores中的ApplicationContextInitializer
    setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));  // 获取所有的配置在spring.factores中的ApplicationContextInitializer
    this.mainApplicationClass = deduceMainApplicationClass();  // 推断启动类，得到FrameworkApplication.class
}
```

```java
// SpringApplication.java
public ConfigurableApplicationContext run(String... args) {  // 运行Spring应用程序，创建并刷新一个新的ApplicationContext
    long startTime = System.nanoTime();
    DefaultBootstrapContext bootstrapContext = createBootstrapContext();
    ConfigurableApplicationContext context = null;
    configureHeadlessProperty();
    SpringApplicationRunListeners listeners = getRunListeners(args);  // 1 实例化spring.factories中配置的所有SpringApplicationRunListener并返回到 listeners 中。SpringApplicationRunListeners内部是一个List<SpringApplicationRunListener>。spring应用进行到某一阶段时会广播通知所有的监听器, 监听器的方法就会被回调执行
    listeners.starting(bootstrapContext, this.mainApplicationClass); //启动应用监听器，回调监听器的starting方法。
    try {
        ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);//包装命令行启动参数
        ConfigurableEnvironment environment = prepareEnvironment(listeners, bootstrapContext, applicationArguments);  // 2 准备应用环境, 包括读取系统环境变量、yml/properties等配置文件。同时回调listeners的environmentPrepared方法
        Banner printedBanner = printBanner(environment);  // 打印Banner(Spring logo)，可以自定义
        context = createApplicationContext();  // 3 创建一个新的ApplicationContext
        context.setApplicationStartup(this.applicationStartup);
        prepareContext(bootstrapContext, context, environment, listeners, applicationArguments, printedBanner);  // 4 上下文准备，会广播通知 listeners 执行 contextPrepared 回调方法
        refreshContext(context);  // 5 刷新上下文
        afterRefresh(context, applicationArguments);  // 6 收尾工作
        Duration timeTakenToStartup = Duration.ofNanos(System.nanoTime() - startTime);
        if (this.logStartupInfo) {
            new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), timeTakenToStartup);
        }
        listeners.started(context, timeTakenToStartup);
        callRunners(context, applicationArguments);
    }
    catch (Throwable ex) { ...        
    }
    try {
        if (context.isRunning()) {
            Duration timeTakenToReady = Duration.ofNanos(System.nanoTime() - startTime);
            listeners.ready(context, timeTakenToReady);
        }
    }
    catch (Throwable ex) { ...       
    }
    return context;
}
```



#### createApplicationContext()

创建默认 IOC 容器

```java
// // SpringApplication.java
protected ConfigurableApplicationContext createApplicationContext() {
    return this.applicationContextFactory.create(this.webApplicationType);
}
```

```java
// DefaultApplicationContextFactory.java
public ConfigurableApplicationContext create(WebApplicationType webApplicationType) {
    try {
        return getFromSpringFactories(webApplicationType, ApplicationContextFactory::create,
                this::createDefaultApplicationContext);  // 
    }
    catch (Exception ex) { ...
}	}
```

```java
// DefaultApplicationContextFactory.java
private <T> T getFromSpringFactories(WebApplicationType webApplicationType,
        BiFunction<ApplicationContextFactory, WebApplicationType, T> action, Supplier<T> defaultResult) {
    for (ApplicationContextFactory candidate : SpringFactoriesLoader.loadFactories(ApplicationContextFactory.class,
            getClass().getClassLoader())) {
        // (ServletWebServerApplicationContextFactory,"SERVLET")
        T result = action.apply(candidate, webApplicationType);          
        if (result != null) {
            return result;
    }	}
    return (defaultResult != null) ? defaultResult.get() : null;
}
```

```java
// ServletWebServerApplicationContextFactory.java
private ConfigurableApplicationContext createContext() {
    if (!AotDetector.useGeneratedArtifacts()) {
        return new AnnotationConfigServletWebServerApplicationContext();  // 默认的IOC容器
    }
    return new ServletWebServerApplicationContext();
}
```



#### refreshContext()

`refresh():void` 初始化/刷新底层 ApplicationContext。从给定的 XML 文件中加载所有 BeanDefinition 并实例化初始化 Bean

```java
// SpringApplication.java
private void refreshContext(ConfigurableApplicationContext context) {
    if (this.registerShutdownHook) {
        shutdownHook.registerApplicationContext(context);
    }
    refresh(context);
}
```

```java
// SpringApplication.java
protected void refresh(ConfigurableApplicationContext applicationContext) {
    applicationContext.refresh();
}
```

```java
// ServletWebServerApplicationContext.java
public final void refresh() throws BeansException, IllegalStateException {
    try {
        super.refresh();
    }
    catch (RuntimeException ex) {
        WebServer webServer = this.webServer;
        if (webServer != null) {
            webServer.stop();
        }
        throw ex;
}	}
```

| refresh()方法过程                            | 具体工作                                                     |
| -------------------------------------------- | ------------------------------------------------------------ |
| prepareRefresh()                             | 标准初始化工作                                               |
| obtainFreshBeanFactory()                     | 获取ConfigurableListableBeanFactory，调用 `loadBeanDefinitions(DefaultListableBeanFactory):void` 将 BeanDefinition 加载到给定的 BeanFactory，委托给 BeanDefinitionReaders加载并解析 XML 文件为 BeanDefinition |
| prepareBeanFactory(beanFactory)              |                                                              |
| postProcessBeanFactory(beanFactory)          | 模板方法具体功能由子类实现                                   |
| invokeBeanFactoryPostProcessors(beanFactory) | 初始化Bean工厂后置处理器(JDK动态代理增强)                    |
| registerBeanPostProcessors(beanFactory)      | 注册Bean后置处理器(JDK动态代理增强)                          |
| initMessageSource()                          | 国际化处理                                                   |
| initApplicationEventMulticaster()            | 初始化应用消息广播器                                         |
| onRefresh()                                  | 模板方法具体功能由子类实现                                   |
| registerListeners()                          | 注册监听器                                                   |
| finishBeanFactoryInitialization(beanFactory) | 实例化Bean<br />填充属性<br />调用初始化方法(afterPropertiesSet(), init())<br />调用BeanPostProcessor(生成Bean代理) |
| finishRefresh()                              | 完成上下文的刷新                                             |

```java
// AbstractApplicationContext.java
public void refresh() throws BeansException, IllegalStateException {
	synchronized (this.startupShutdownMonitor) {
        StartupStep contextRefresh = this.applicationStartup.start("spring.context.refresh");

        // 为刷新准备Context
        prepareRefresh();

        // 告诉子类刷新内部BeanFactory
        ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

        // 为使用Context准备BeanFactory
        prepareBeanFactory(beanFactory);

        try {
            // 允许在Context子类中进行BeanFactory的后置处理PostProcess
            postProcessBeanFactory(beanFactory);  // 自定义扩展点

            StartupStep beanPostProcess = this.applicationStartup.start("spring.context.beans.post-process");
            // 激活BanFactory的PostProcessor作为Bean到Context
            invokeBeanFactoryPostProcessors(beanFactory);

            // 注册拦截创建Bean的PostProcessors
            registerBeanPostProcessors(beanFactory);
            beanPostProcess.end();

            // 为了Context初始化MessageSource
            initMessageSource();

            // 为了Context初始化EventMulticaster
            initApplicationEventMulticaster();

            // 初始化在特定Context子类中特定的Bean
            onRefresh();  // 自定义扩展点

            // 查找并注册listener
            registerListeners();

            // 实例化所有剩余的非懒初始化的单例Bean
            finishBeanFactoryInitialization(beanFactory);

            // 最后一步: 发布相应的事件
            finishRefresh();
        }

        catch (BeansException ex) {
            if (logger.isWarnEnabled()) {
                logger.warn("Exception encountered during context initialization - " +
                        "cancelling refresh attempt: " + ex);
            }

            // Destroy already created singletons to avoid dangling resources(未被正确释放的资源).
            destroyBeans();

            // Reset 'active' flag.
            cancelRefresh(ex);

            // Propagate exception to caller.
            throw ex;
        }

        finally {
            // Reset common introspection caches in Spring's core, since we
            // might not ever need metadata for singleton beans anymore...
            resetCommonCaches();
            contextRefresh.end();
        }
    }
}
```



### CommandLineRunner 

可以在同一Application 的上下文中定义多个 CommandLineRunner 或 ApplicationRunner，并且能够使用 `@Order`  注解进行按特定顺序调用的排序

示例：

```java
@Component
public class AppRunner implements CommandLineRunner {
	private static final Logger logger = LoggerFactory.getLogger(AppRunner.class);
    private final BookRepository bookRepository;
	public AppRunner(BookRepository bookRepository) {
		this.bookRepository = bookRepository;
	}
	@Override
	public void run(String... args) throws Exception {  // 用于运行Bean的回调函数
		logger.info(".... Fetching books");
		logger.info("isbn-1234 -->" + bookRepository.getByIsbn("isbn-1234"));
		....
}	}
```





## SpringApplicationBuilder

构建 SpringApplication 和 ApplicationContext 实例，还可以设置活动配置文件，默认属性。

方法：

`banner(Banner)`

`bannerMode(Banner.Mode)`

`context()`

`build(String...)`

`web(WebApplicationType)`

`listeners(ApplicationListener)` 监听 SpringApplication 事件，推荐 SpringApplicationRunListener 粒度更细

`run(String...)`

`parent(Class<?>)`

`parent(ConfigurableApplicationContext)`

`profiles(String...)`

`properties(String...)`



示例：

```java
ConfigurableApplicationContext context = new SpringApplicationBuilder(Application.class)
    	.profiles("server")
    	.properties("transport=local").run(args);
```



## SpringApplicationRunListener

侦听 SpringApplication 运行方法。SpringApplicationRunListeners 是通过 SpringFactoriesLoader 加载的，每次运行都会创建一个新的SpringApplicationRunListener实例。
应该声明一个接受 SpringApplication 实例和String[]参数的公共构造函数。

示例：

```java
public class CustomSpringApplicationRunListener implements SpringApplicationRunListener {
    private SpringApplication application;
    private String[] args;
    public MyRunListener(SpringApplication application, String[] args) {
        this.application = application;
        this.args = args;
    }
    @Override
    public void starting(ConfigurableBootstrapContext bootstrapContext) {
        // 可以输出SpringApplication和args信息。
        System.out.println("CustomSpringApplicationRunListener: Application starting...");
        SpringApplicationRunListener.super.starting(bootstrapContext);
    }
    @Override
    public void environmentPrepared(ConfigurableBootstrapContext bootstrapContext, ConfigurableEnvironment environment) {
        System.out.println("CustomSpringApplicationRunListener: Environment prepared.");
        SpringApplicationRunListener.super.environmentPrepared(bootstrapContext, environment);
    }
    @Override
    public void contextPrepared(ConfigurableApplicationContext context) {
        System.out.println("CustomSpringApplicationRunListener: Context prepared.");
        SpringApplicationRunListener.super.contextPrepared(context);
    }
    @Override
    public void contextLoaded(ConfigurableApplicationContext context) {
        System.out.println("CustomSpringApplicationRunListener: Context loaded.");
        SpringApplicationRunListener.super.contextLoaded(context);
    }
    @Override
    public void started(ConfigurableApplicationContext context, Duration timeTaken) {
        System.out.println("CustomSpringApplicationRunListener: Application started.");
        SpringApplicationRunListener.super.started(context, timeTaken);
    }
    @Override
    public void ready(ConfigurableApplicationContext context, Duration timeTaken) {
        System.out.println("CustomSpringApplicationRunListener: Application ready.");
        SpringApplicationRunListener.super.ready(context, timeTaken);
    }
    @Override
    public void failed(ConfigurableApplicationContext context, Throwable exception) {
        System.out.println("CustomSpringApplicationRunListener: Application failed to start.");
        SpringApplicationRunListener.super.failed(context, exception);
}	}
```

```META-INF/spring.factories
org.springframework.boot.SpringApplicationRunListener = \ cn.cxc.framework.listener.CustomSpringApplicationRunListener
```



## 1加依赖

### SPI机制

Service Provider Interface 服务提供接口 = 基于Interface + 策略模式 + 配置文件(将变化隔离)，配置文件适合整体解决方案的变化
@Primary @条件注解，适合具体类/对象的变化  [Spring IOC笔记#条件注解](../spring-core/核心/spring_ioc.md#配置类)

```
					 方案A
调用方 - 标准服务接口 - 方案B
					 方案C
```



Java SPI 机制：在 `src/main/resources/META-INF/services` 目录， 新增一个以接口命名的文件，使用 `java.util.ServiceLoader` 来加载配置文件中指定的实现。例如`mysql-connector-java-8.0.18.jar` 采用原生 SPI 机制，就有一个 `/META-INF/services/java.sql.Driver` 

```/META-INF/services/java.sql.Driver
com.mysql.cj.jdbc.Driver
```



Spring Boot 是 Java SPI 机制的变种，



### Starter

Starter 包是特殊的 JAR 包，将 jar 添加到类路径，包含一组预配置和依赖项。
有的提供 `@EnableXXX` 注解和配置类

[Spring Boot 管理的依赖](https://docs.spring.io/spring-boot/docs/current/reference/html/dependency-versions.html#appendix.dependency-versions)



| ArtifactId 构建ID            | 描述                                                         |
| ---------------------------- | ------------------------------------------------------------ |
| spring-boot-starter-web      | 整合 SpringMVC，提供 Web 支持                                |
| spring-boot-starter-data-jpa | 对 JPA 支持，集成 Hibernate                                  |
| spring-boot-starter-logging  | 增加logback日志的支持                                        |
| spring-boot-starter-test     | 整合 Unit 单元测试                                           |
| spring-boot-starter-parent   | 提供默认 maven，提供依赖管理以省略 version 标签              |
| spring-boot-starter-actuator | 监控和管理应用程序，提供导航端点<br />http://localhost:8080/actuator |
| **第三方**                   |                                                              |
| mybatis-spring-boot-starter  | 整合 Mybatis                                                 |



### 实现

**自己实现 @EnalbeXX 注解和 ImportSelector 接口 **

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Import(StarterSelector.class)
public @interface EnableStarterConfigration {
}
```

```java
public class StarterSelector implements ImportSelector {
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        return new String[]{StarterConfigration.class.getName()};
}	}
```

```java
@Configuration
public class StarterConfigration {
    @Bean
    public MyDo myDo() {
        return new MyDo();
}	}
```

```java
@EnableStarterConfigration
public class TryStarterApplication {
    public static void main(String[] args) {
        ConfigurableApplicationContext context =
                new SpringApplicationBuilder(TryStarterApplication.class)
                        .web(WebApplicationType.NONE)  // 不启动web容器
                        .run(args);
        IDo myDo = (IDo) context.getBean("myDo");
        myDo.doSomething();  // 成功自动装配
}	}
```



**利用@EnableAutoConfiguration和\META-INF\spring.factories**

为了让 Spring Boot 自动发现您的自动配置类，您可能需要将其所在的包或模块添加到 `spring.factories` 文件(用于声明自动配置类)中

这样，当其他项目引入您的自动配置依赖项时，Spring Boot 将自动加载并应用您的自动配置类。

```\META-INF\spring.factories
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  cn.cxc.demo.StarterConfigration
```



### 自动配置

Spring Boot 的自动装配机制会试图根据你所添加的依赖来自动配置你的Spring应用程序。 例如，如果你添加了 `HSQLDB` 依赖，而且你没有手动配置任何 DataSource Bean，那么 Spring Boot 就会自动配置内存数据库。

你需要将 `@EnableAutoConfiguration` 或 `@SpringBootApplication` 注解添加一个主要的 `@Configuration` 类上，从而开启自动配置功能。

具体过程：当应用程序引入特定的 Starter 包后，Spring Boot 会自动扫描该 Starter 包中的配置类，并根据条件判断(条件注解)是否应用这些自动配置。这样应用程序就可以自动获取所需的功能和设置，而无需显式进行繁琐的配置。**本质就是 `@Import` 注解导入第三方配置类和Bean到 IOC 容器中。**

`@Component` 或 @`Configration`注解，只能装载自己项目中的源码库



#### 原理

自动配置原理就是，有一个 `@EnableXXX` 注解装载第三方模块，其中包含 `@Import` 注解导入 `Selector` 导包选择器类实例到 IOC 容器中，通过 `selector.selectImports()` 读取 `META-INF/spring/full-qualified-annotation-name.imports` 文件中配置类的**全类名**，来注册配置类和其中的 Bean 到 IOC 容器，最终装载了第三方模块。

Spring2.7之前读取 `META-INF/spring.factories` 文件

```java
@SpringBootConfiguration  // 标识该类为配置类，配置类用于定义Bean配置信息
@EnableAutoConfiguration  // 启用 Spring Application Context 的自动配置功能，装配自动配置模块到classpath上
@ComponentScan  // 用于扫描同级目录以下的 bean 加入到 IOC 容器中
public @interface SpringBootApplication {
    // ......
}
```

```java
@Configuration
public @interface SpringBootConfiguration {  
}
```

spring-boot-autoconfigration.jar 是 SpringBoot 内置配置类

```java
@AutoConfigurationPackage  // 将@SpringBootApplication主配置类所在的包下面所有的组件都扫描注冊到 spring 容器中
@Import(AutoConfigurationImportSelector.class)  // 导入 AutoConfigurationImportSelector 类实例到 IOC 容器中
public @interface EnableAutoConfiguration {  
}
```

```java
// AutoConfigurationImportSelector.java  // 导包选择器
// 获取要导入的自动配置类的全类名。即扫描spring.factories文件并返回其中Configuration类的全类名
public String[] selectImports(AnnotationMetadata annotationMetadata) {
    if (!isEnabled(annotationMetadata)) {
        return NO_IMPORTS;
    }
    AutoConfigurationEntry autoConfigurationEntry = getAutoConfigurationEntry(annotationMetadata);  // 
    return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
}
```

```java
// AutoConfigurationImportSelector.java
// 根据导入的@Configuration类的AnnotationMetadata返回AutoConfigurationImportSelector.AutoConfigurationEntry
protected AutoConfigurationEntry getAutoConfigurationEntry(AnnotationMetadata annotationMetadata) {
    if (!isEnabled(annotationMetadata)) {
        return EMPTY_ENTRY;
    }
    AnnotationAttributes attributes = getAttributes(annotationMetadata);
    List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);
    configurations = removeDuplicates(configurations);
    Set<String> exclusions = getExclusions(annotationMetadata, attributes);
    checkExcludedClasses(configurations, exclusions);
    configurations.removeAll(exclusions);
    configurations = getConfigurationClassFilter().filter(configurations);
    fireAutoConfigurationImportEvents(configurations, exclusions);
    return new AutoConfigurationEntry(configurations, exclusions);
}
```

```java
// AutoConfigurationImportSelector.java
// 获取候选配置类的全类名类List
protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
    List<String> configurations = ImportCandidates.load(AutoConfiguration.class, getBeanClassLoader())
        .getCandidates();  // 
    Assert.notEmpty(configurations,
            "No auto configuration classes found in "
                    + "META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports. If you "
                    + "are using a custom packaging, make sure that file is correct.");
    return configurations;
}
```

```java
// ImportCandidates.java
// 从 classpath 中加载候选导入文件的名称。候选导入类的名称存储在 classpath 上名为META-INF/spring/full-qualified-annotation-name.imports的文件中
public static ImportCandidates load(Class<?> annotation, ClassLoader classLoader) {
    Assert.notNull(annotation, "'annotation' must not be null");
    ClassLoader classLoaderToUse = decideClassloader(classLoader);
    String location = String.format(LOCATION, annotation.getName());
    Enumeration<URL> urls = findUrlsInClasspath(classLoaderToUse, location);
    List<String> importCandidates = new ArrayList<>();
    while (urls.hasMoreElements()) {
        URL url = urls.nextElement();
        importCandidates.addAll(readCandidateConfigurations(url));
    }
    return new ImportCandidates(importCandidates);
}
```

```java
// AutoConfigurationImportSelector.java
protected List<AutoConfigurationImportFilter> getAutoConfigurationImportFilters() {
    return SpringFactoriesLoader.loadFactories(AutoConfigurationImportFilter.class, this.beanClassLoader);
}
```

TODO: 有空看看这个类 SpringFactoriesLoader.java，了解自动装配

```java
// SpringFactoriesLoader
// 创建SpringFactoriesLoader实例，使用指定的类加载器从指定位置META-INF/spring.factories加载并实例化工厂实现
public static SpringFactoriesLoader forDefaultResourceLocation(@Nullable ClassLoader classLoader) {
    return forResourceLocation(FACTORIES_RESOURCE_LOCATION, classLoader);
}
```

spring.factories 文件用于指定当前 SDK 需要装载的配置类。

```spring.factor
# C:\Users\qq109\.m2\repository\org\springframework\boot\spring-boot-autoconfigure\2.2.2.RELEASE\spring-boot-autoconfigure-2.2.2.RELEASE.jar!\META-INF\spring.factories
# Initializers
org.springframework.context.ApplicationContextInitializer=\
org.springframework.boot.autoconfigure.SharedMetadataReaderFactoryContextInitializer,\
org.springframework.boot.autoconfigure.logging.ConditionEvaluationReportLoggingListener

# Application Listeners
org.springframework.context.ApplicationListener=\
org.springframework.boot.autoconfigure.BackgroundPreinitializer

# Auto Configuration Import Listeners
org.springframework.boot.autoconfigure.AutoConfigurationImportListener=\
org.springframework.boot.autoconfigure.condition.ConditionEvaluationReportAutoConfigurationImportListener

# Auto Configuration Import Filters
org.springframework.boot.autoconfigure.AutoConfigurationImportFilter=\
org.springframework.boot.autoconfigure.condition.OnBeanCondition,\
org.springframework.boot.autoconfigure.condition.OnClassCondition,\
org.springframework.boot.autoconfigure.condition.OnWebApplicationCondition

# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\
....
```



## 3写配置

属性配置比配置类优先级更高，配置类存在父子上下文重复扫描问题

常用的配置管理方式

1 与 jar 包同级的外部配置文件
2 自动加载 resource 目录下的配置文件 `application-{profile}.properties` 或 `application-{profile}.yml`
3 配置项为 ${SOME_ENV} 使用环境变量或命令行参数`java -jar xxx.jar --key1.key11=value`

application-{profile}.yml
yaml 可以保证配置顺序(properties无序)，"*"一定要加引号(properties不需要)

```yaml
key1:
  key11: value1, value2
  key22:
    - value3  # - 表示列表
```

[常用 Spring Boot 属性列表](https://docs.spring.io/spring-boot/docs/current/reference/html/application-properties.html#appendix.application-properties)

```yaml
debug: true  # 调试模式，debug模式下会打印自动装配项目以及触发自动装配的条件

logging:
    level:
      org.springframework.web: DEBUG
      org.hibernate: DEBUG
    file:
      path: trymylog  # 日志文件输出路径

server:
  port: 8080
  servlet.context-path: / # 设置应用上下文

spring:
  mvc:
    log-request-details: true
  main:
    banner-mode: "off"
  jackson:
    time-zone: GMT+8  # 必需  # 2023-09-30T00:37:18.000+08:00 时区+8
    date-format: yyyy-MM-dd HH:mm:ss
    property-naming- strategy: SNAKE_CASE
    serialization:
      WRITE_DATES_AS_TIMESTAMPS: true
  datasource:
    url: jdbc:mysql://localhost:3306/user_center
    username: root
    password: 
    driver-class-name: com.mysql.cj.jdbc.Driver
    hikari:  # 数据库连接池
      maximum-pool-size: 10    
  config:
    import:  # 导入外部配置文件
      - classpath:application-nacos.yml
      - classpath:application-actuator.yml
      - optional:nacos:dev.yml  # 监听nacos配置中心的DEFAULT_GROUP:dev.yml
      - optional:nacos:aaa.yml?group=group_01&refreshEnabled=false # 不开启动态刷新
  profiles:
    active: dev  # 指定环境配置文件application-{env}.yml。profile: dev、prd
    group:  # 定义配置组的逻辑名称，为了关联配置  # 配合 spring.config.activate.on-profile使用
      dev: dev-actuator, dev-nacos
      prod: prod-actuator, prod-nacos
```



### actuator

```yaml
--- # actuator
spring:
  config:
    activate:
      on-profile: dev-actuator  # profile表达式  # 配合spring.profiles.group使用
info:
  app:
    name: try-spring-boot
    version: 0.1-SNAPSHOT
    description: Demo project for Spring Boot
  author: "Nemesis"
  email: "chenxiangcheng1@gmail.com"

management:
  endpoint:
    health:
      show-details: always  # 健康端点显示完整的健康信息
  endpoints:
    web:
      exposure:
        include: health,info  # 暴露健康端点和 info 端点，"*"暴露所有端点
  info:
    env:
      enabled: true  # 暴露 Environment 中名称以 info. 开头的任何属性
```



### nacos

```yaml
--- # nacos
spring:
  config:
    activate:
      on-profile: dev-nacos  # profile表达式  # 配合spring.profiles.group使用
spring:
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
        namespace: 450cb8f7-64d4-4be0-8c5d-49b7450c2d6c  # Nacos命名空间ID
        cluster-name: HZ  # Nacos集群名称
        metadata:
          instance: content-center-instanceName
          # 自己这个实例的版本
          version: v1
          # 允许调用的提供者版本
          target-version: v1
      config:
        server-addr: 127.0.0.1:8848
  config:
    import:
      - optional:nacos:dev.yml  # 监听nacos配置中心的DEFAULT_GROUP:dev.yml
      - optional:nacos:aaa.yml?group=group_01&refreshEnabled=false # 不开启动态刷新
    activate:
      on-profile: nacos
  application:
    name: user-center  # 服务名称尽量用-, 不要用_  # 作为Nacos服务名称
```



### ribbon

```yaml
--- # ribbon
spring:
  config:
    activate:
      on-profile: dev-ribbon  # profile表达式  # 配合spring.profiles.group使用
<clientName>:  # 局部配置 - 当Ribbon调用xxx-center/service<clientName>时，使用RandomRule...配置的负载均衡器
  ribbon:
	NFLoadBalancerclassName: ILoadBalancer实现类全路径
	NFLoadBalancerRuleclassName: IRule实现类全路径
	NFLoadBalancerPingclassName: IPing实现类全路径
	NIWSServerListclassName: ServerList实现类全路径
	NIWSServerListFilterclassName: ServerListFilter实现类全路径
ribbon:
  eager-load:  # 创建和初始化负载均衡器LoadBalancer的时机，eager-load enabled饥饿加载、懒加载
    enabled: true
    clients: user-center,xxx-service
```



## 2Spring Boot 注解

| 注解                       | 元注解Target                      | 释义                                                         | 成员变量                                         |
| -------------------------- | --------------------------------- | ------------------------------------------------------------ | ------------------------------------------------ |
| `@SpringBootApplication`   | TYPE                              | =@SpringBootConfiguration+<br />@EnableAutoConfiguration+<br />@ComponentScan。标识该类为配置类，装配自动配置模块，启动组件扫描 |                                                  |
| `@EnableXXXX`              |                                   | =@Import。启动XXX功能，装配模块。可以去看自动配置原理        |                                                  |
| `@Import`                  | TYPE                              | 导入该类实例到 IOC 容器中                                    | `value():Class<?>[]`                             |
| `@AutoConfiguration`       | TYPE                              | 标识该类提供了能被Spring Boot自动应用的配置，在指定的自动配置类之间应用，只影响定义 Bean 的顺序，不影响创建 Bean 的顺序<br />=@Configuration+@AutoConfigureBefore+@AutoConfigureAfter | `before():Class<?>[]`<br />`afters():Class<?>[]` |
| `@ConfigurationProperties` | TYPE、@Configuration类中@Bean方法 | 外部配置多个属性注入，类型安全(编译时反馈)，需要创造一个 POJO | 可绑定到该对象的有效属性的前缀                   |











# 未整理

## 自动发现/统一注册

springboot中 主要是发现 (优点，代码量少，缺点，资源分散，联系不明显)，而不是注册 (统一注册)

使类被 Spring 发现，需要使用特定的注解或者实现特定的接口



注册controller，发现controller



## 访问托管的静态资源

模板引擎渲染：在服务端把html页面渲染好再返回到前端
		静态资源一般存贮在 resources/static/ 下。
			如图片存在 resources/static/imgs/ 下，可以直接通过 localhost:8081/imgs/数字0.png 访问静态资源图片Content-Type会直接设置为对应的静态资源类型，如 image/png

前后端分离：服务端返回 Json数据，很少返回 html
		静态资源托管在 OSS 七牛 码云 单独构建一个Nginx服务 另外构建一个Springboot项目仅保存图片 当前的项目工程static中



## ValidationMessages.properties

文件名不能变：约定大于配置

```properties
id.positive = id必须是正整数，可以带入参数{min}{max}
token.password = password不符合规范：当前值是${validatedValue}，最大值应该是{max}，最小值应该是{min}

${validatedValue}：表示当前的需要校验的值
```

JSR303校验的message模板配置：消除Controller参数校验注解的message**硬编码**：@Positive(message = "错误信息")，该文件用于配置消息错误信息的配置

应用：@Positive(message = "{id.positive}")，会抛出ConstraintViolationException异常，要在全局异常处理中处理（Constraint：约束，Violation：违反）

## SpringBoot 框架二次封装

### 全局异常处理

异常的分类
	Error
	Exeption：
		CheckedException：编译时异常
		RuntimeException：运行时异常

#### controller

```java
package com.zjbti.missyou.api.v1;

import com.zjbti.missyou.dto.PersonDTO;
import com.zjbti.missyou.service.BannerService;
import org.hibernate.validator.constraints.Length;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.*;

import javax.servlet.http.HttpServletResponse;
import javax.validation.constraints.Max;

/**
 * @date 2021/1/25 @Created by cxc
 */
@Controller
@RequestMapping("/banner")
@Validated
public class BannerController {

    private BannerService bannerService;

    @Autowired
    public BannerController(BannerService bannerService) {
        this.bannerService = bannerService;
    }

    //    @GetMapping("/test")
    @RequestMapping(value = {"/test/{id1}", "/test1/{id1}"}, method = {RequestMethod.GET, RequestMethod.POST})
    @ResponseBody
    public PersonDTO test(@PathVariable(name = "id1") @Max(value = 10, message = "不可以超过10哦QAQ") Integer id,
                       @RequestParam(value = "name") @Length(min = 2, max = 3) String name,
//                       @RequestBody Map<String,Object> person,
                       @RequestBody @Validated PersonDTO person,
                       HttpServletResponse response) {
//        response.getWriter().write("hello你好");
//        throw new RuntimeException("123123");
//        PersonDTO personDTO = new PersonDTO(null,11);
//        PersonDTO personDTO = new PersonDTO();
//        personDTO.setName("c");
//        personDTO.setAge(15);
        PersonDTO personDTO = PersonDTO.builder().name("陈祥城").age(18).build();
//        dto.setAge(11);
        return personDTO;
//        throw new NotFoundException(10000);
//        return "Hello你好呀";
    }
}
```



#### HttpException

```java
package com.zjbti.missyou.exception.http;

/**
 * @date 2021/1/28 @Created by cxc
 */
public class HttpException extends RuntimeException {

    protected Integer code;
    protected Integer httpStatusCode = 500;

    public Integer getCode() {
        return code;
    }

    public Integer getHttpStatusCode() {
        return httpStatusCode;
    }
}

```



#### NotFoundException

```java
package com.zjbti.missyou.exception.http;

/**
 * @date 2021/1/28 @Created by cxc
 */
public class NotFoundException extends HttpException {

    public NotFoundException(int code) {
        this.code = code;
        this.httpStatusCode = 404;
    }
}

```



#### GlobalExceptionAdvice 全局异常通知

```java
package com.zjbti.missyou.core;

import com.zjbti.missyou.core.configration.ExceptionCodeConfigration;
import com.zjbti.missyou.exception.http.HttpException;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpStatus;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.ResponseStatus;

import javax.servlet.http.HttpServletRequest;

/**
 * 统一处理所有异常，并且返回到前端
 * @date 2021/1/27 @Created by cxc
 */
@ControllerAdvice
public class GlobalExceptionAdvice {

    @Autowired
    private ExceptionCodeConfigration codeConfigration;

    @ExceptionHandler(value = Exception.class)
    @ResponseBody
    @ResponseStatus(code = HttpStatus.INTERNAL_SERVER_ERROR)
    public UnifyResponse handleException(HttpServletRequest req, Exception e) {
        String requestUrl = req.getRequestURI();
        String method = req.getMethod();
        System.out.println(e);
        
        UnifyResponse message = new UnifyResponse(9999, "服务器内部错误", method + " " + requestUrl);
        return message;
    }

    // 以这种方式处理异常
    @ExceptionHandler(value = HttpException.class)
    public ResponseEntity<UnifyResponse> handleHttpException(HttpServletRequest req, HttpException e) {
        String requestUrl = req.getRequestURI();
        String method = req.getMethod();
        UnifyResponse message = new UnifyResponse(e.getCode(), codeConfigration.getMessage(e.getCode()), method+ " " + requestUrl);

        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_JSON);

        HttpStatus httpStatus = HttpStatus.resolve(e.getHttpStatusCode());

        ResponseEntity<UnifyResponse> r = new ResponseEntity<>(message, headers, httpStatus);
        return r;
    }

}

```



#### 注解

| 注解                                                         | 释义                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **GlobalExceptionAdvice**                                    | 全局异常通知                                                 |
| @ControllerAdvice                                            | Controller 通知/增强，用于声明全局型的东西                   |
| @ExceptionHandler(value = Exception.class)                   | 异常处理，注解标注的方法，用于**捕获Controller中抛出的不同类型的异常**，并进行处理返回，从而达到异常全局处理的目的 |
|                                                              |                                                              |
| @ResponseStatus(code = HttpStatus.INTERNAL_SERVER_ERROR)     | 修改响应状态码，也可以通过ResposneEntity(message, headers, httpStatus) 的构造参数 httpStatus 改 |
|                                                              |                                                              |
| **ExceptionCodeConfigration**                                | 异常码配置                                                   |
| @ConfigurationProperties(prefix="lin")                       | 配置属性的前缀                                               |
| @PropertySource(value = "classpath:config/exception-code.properties") | 配置属性数据源，会自动注入对应配置文件名的属性               |
|                                                              |                                                              |



#### 配置异常码

```properties
lin.codes[10000] = 通用异常
lin.codes[10001] = 通用参数错误
```



#### ExceptionCodeConfigration

```java
package com.zjbti.missyou.core.configration;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.PropertySource;
import org.springframework.stereotype.Component;

import java.util.HashMap;
import java.util.Map;

/**
 * 避免根据异常码配置文件一个个注入，直接封装一个类，负责管理一个配置文件
 * @date 2021/1/28 @Created by cxc
 */
@ConfigurationProperties(prefix="lin")
@PropertySource(value = "classpath:config/exception-code.properties")
@Component
public class ExceptionCodeConfigration {

    private Map<Integer, String> codes = new HashMap<>();

    public Map<Integer, String> getCodes() {
        return this.codes;
    }

    public void setCodes(Map<Integer, String> codes) {
        this.codes = codes;
    }

    public String getMessage(int code) {
        return codes.get(code);
    }

}

```

#### UnifyResponse

```java
package com.zjbti.missyou.core;

/**
 * 序列化需要getter访问器，不然Spring序列化读取不到，会报错
 * @date 2021/1/28 @Created by cxc
 */
public class UnifyResponse {

    private Integer code;
    private String message;
    private String url;

    public UnifyResponse(Integer code, String message, String url) {
        this.code = code;
        this.message = message;
        this.url = url;
    }

    public Integer getCode() {
        return code;
    }

    public String getMessage() {
        return message;
    }

    public String getUrl() {
        return url;
    }
}

```



### 根据目录自动路由前缀映射

#### AutoPrefixConfigration 通过config注入

```java
package com.zjbti.missyou.core.configration;

import com.zjbti.missyou.core.hack.AutoPrefixUrlMapping;
import org.springframework.boot.autoconfigure.web.servlet.WebMvcRegistrations;
import org.springframework.stereotype.Component;
import org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping;

/**
 * @date 2021/1/31 @Created by cxc
 */
@Component
public class AutoPrefixConfigration implements WebMvcRegistrations {

    @Override
    public RequestMappingHandlerMapping getRequestMappingHandlerMapping() {
        return new AutoPrefixUrlMapping();
    }
}

```

#### AutoPrefixUrlMapping 处理请求映射的类

```java
package com.zjbti.missyou.core.hack;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;
import org.springframework.web.servlet.mvc.method.RequestMappingInfo;
import org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping;

import java.lang.reflect.Method;

/**
 * @date 2021/1/31 @Created by cxc
 */
@Component
public class AutoPrefixUrlMapping extends RequestMappingHandlerMapping {

    @Value("${missyou.api-package}")
    private String apiPackagePath;

    @Override
    protected RequestMappingInfo getMappingForMethod(Method method, Class<?> handlerType) {
        RequestMappingInfo mappingInfo = super.getMappingForMethod(method, handlerType);
        if (null != mappingInfo) {
            String prefix = this.getPrefix(handlerType);
            RequestMappingInfo newMappingInfo = RequestMappingInfo.paths(prefix).build().combine(mappingInfo);
            return newMappingInfo;
        }
        return mappingInfo;
    }

    private String getPrefix(Class<?> handlerType) {
        String packageName = handlerType.getPackage().getName();
        String dotPath = packageName.replaceAll(this.apiPackagePath, "");
        return dotPath.replace(".", "/");
    }
}

```

### Java Post参数校验 + Spring全局异常处理

#### controller

```java
package com.zjbti.missyou.api.v1;

import com.zjbti.missyou.dto.PersonDTO;
import com.zjbti.missyou.service.BannerService;
import org.hibernate.validator.constraints.Length;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.*;

import javax.servlet.http.HttpServletResponse;
import javax.validation.constraints.Max;

/**
 * @date 2021/1/25 @Created by cxc
 */
@Controller
@RequestMapping("/banner")
@Validated
public class BannerController {

    private BannerService bannerService;

    @Autowired
    public BannerController(BannerService bannerService) {
        this.bannerService = bannerService;
    }

    //    @GetMapping("/test")
    @RequestMapping(value = {"/test/{id1}", "/test1/{id1}"}, method = {RequestMethod.GET, RequestMethod.POST})
    @ResponseBody
    public PersonDTO test(@PathVariable(name = "id1") @Max(value = 10, message = "不可以超过10哦QAQ") Integer id,
                       @RequestParam(value = "name") @Length(min = 2, max = 3) String name,
//                       @RequestBody Map<String,Object> person,
                       @RequestBody @Validated PersonDTO person,
                       HttpServletResponse response) {
//        response.getWriter().write("hello你好");
//        throw new RuntimeException("123123");
//        PersonDTO personDTO = new PersonDTO(null,11);
//        PersonDTO personDTO = new PersonDTO();
//        personDTO.setName("c");
//        personDTO.setAge(15);
        PersonDTO personDTO = PersonDTO.builder().name("陈祥城").age(18).build();
//        dto.setAge(11);
        return personDTO;
//        throw new NotFoundException(10000);
//        return "Hello你好呀";
    }
}

```

#### DTO 使用自定义注解

DTO：data transfer object数据传输对象，定义的是服务端接受到的数据对象

```java
package com.zjbti.missyou.dto;


import com.zjbti.missyou.validators.PasswordEqual;
import lombok.Builder;
import lombok.Getter;
import org.hibernate.validator.constraints.Length;

/**
 * @date 2021/2/1 @Created by cxc
 */
@Getter
//@Setter
//@AllArgsConstructor
//@RequiredArgsConstructor
//@NoArgsConstructor
@Builder
//@NoArgsConstructor
@PasswordEqual(min = 1,message = "两次密码不相同")
public class PersonDTO {

//    @NonNull
    @Length(min=2, max = 10, message = "name长度错误")
    private String name;
    private Integer age;

    private String password1;
    private String password2;

//    public String getName() {
//        return name;
//    }
//
//    public void setName(String name) {
//        this.name = name;
//    }
//
//    public Integer getAge() {
//        return age;
//    }
//
//    public void setAge(Integer age) {
//        this.age = age;
//    }
}

```

#### 自定义注解

```java
package com.zjbti.missyou.validators;

import javax.validation.Constraint;
import javax.validation.Payload;
import java.lang.annotation.*;

/**
 * @date 2021/2/2 @Created by cxc
 */
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE, ElementType.FIELD})
// 约束
@Constraint(validatedBy = PasswordValidator.class)
public @interface PasswordEqual {
    int min() default 4;
    int max() default 6;
    String message() default "passwords are not equal";
    // 自定义校验注解的两个规范方法
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}

```

#### 注解处理器 实现 ConstraintValidator 接口

```java
package com.zjbti.missyou.validators;

import com.zjbti.missyou.dto.PersonDTO;

import javax.validation.ConstraintValidator;
import javax.validation.ConstraintValidatorContext;

/**
 * 实现 ConstraintValidator接口，指定泛型第一个是自定义注解，第二个是自定义注解修饰的目标的类型
 * @date 2021/2/2 @Created by cxc
 */
public class PasswordValidator implements ConstraintValidator<PasswordEqual, PersonDTO> {

    private int min;
    private int max;

    @Override
    public void initialize(PasswordEqual constraintAnnotation) {
        this.min = constraintAnnotation.min();
        this.max = constraintAnnotation.max();
    }

    @Override
    public boolean isValid(PersonDTO personDTO, ConstraintValidatorContext constraintValidatorContext) {
        String password1 = personDTO.getPassword1();
        String password2 = personDTO.getPassword2();
        boolean match = password1.equals(password2);
        return match;
    }
}

```

#### 全局异常处理

```java
package com.zjbti.missyou.core;

import com.zjbti.missyou.core.configration.ExceptionCodeConfigration;
import com.zjbti.missyou.exception.http.HttpException;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpStatus;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.validation.ObjectError;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.ResponseStatus;

import javax.servlet.http.HttpServletRequest;
import javax.validation.ConstraintViolationException;
import java.util.List;

/**
 * 统一处理所有异常，并且返回到前端
 * @date 2021/1/27 @Created by cxc
 */
@ControllerAdvice
public class GlobalExceptionAdvice {

    @Autowired
    private ExceptionCodeConfigration codeConfigration;

    @ExceptionHandler(value = Exception.class)
    @ResponseBody
    @ResponseStatus(code = HttpStatus.INTERNAL_SERVER_ERROR)
    public UnifyResponse handleException(HttpServletRequest req, Exception e) {
        String requestUrl = req.getRequestURI();
        String method = req.getMethod();
        // 打印未知异常，可高效查看异常和开发
        System.out.println(e);

        UnifyResponse message = new UnifyResponse(9999, "服务器内部错误(未知异常)", method + " " + requestUrl);
        return message;
    }

    // HttpException异常发生，http状态码不相同，需要动态设置
    @ExceptionHandler(value = HttpException.class)
    public ResponseEntity<UnifyResponse> handleHttpException(HttpServletRequest req, HttpException e) {
        String requestUrl = req.getRequestURI();
        String method = req.getMethod();
        UnifyResponse message = new UnifyResponse(e.getCode(), codeConfigration.getMessage(e.getCode()), method+ " " + requestUrl);

        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_JSON);

        HttpStatus httpStatus = HttpStatus.resolve(e.getHttpStatusCode());

        ResponseEntity<UnifyResponse> r = new ResponseEntity<>(message, headers, httpStatus);
        return r;
    }

    // 参数异常，http状态码相同，不用动态设置，使用注解
    @ExceptionHandler(value = MethodArgumentNotValidException.class)
    @ResponseBody
    @ResponseStatus(code = HttpStatus.BAD_REQUEST)
    public UnifyResponse handleBeanValidation(HttpServletRequest req, MethodArgumentNotValidException e) {
        String requestUrl = req.getRequestURI();
        String method = req.getMethod();

        List<ObjectError> errors = e.getBindingResult().getAllErrors();
        String message = this.formatAllErrorMessages(errors);

        return new UnifyResponse(10001, message, method + " " + requestUrl);
    }

//	Constraint：约束	Violation：违反
    @ExceptionHandler(value = ConstraintViolationException.class)
    @ResponseBody
    @ResponseStatus(code = HttpStatus.BAD_REQUEST)
    public UnifyResponse handleConstraintViolationException(HttpServletRequest req, ConstraintViolationException e) {
        // 使用遍历自获取message，定义性更强
//        StringBuffer errorMsg = new StringBuffer();
//        for (ConstraintViolation error : e.getConstraintViolations()) {
//            String msg = e.getMessage();
//            String m = error.getPropertyPath().toString();
//            String name = m.split("[.]")[1];
//            errorMsg.append(name).append(" ").append(msg).append(";");
//        }
//        return new UnifyResponse(10001, errorMsg.toString(), requestUrl);
        String requestUrl = req.getRequestURI();
        String method = req.getMethod();
        String message = e.getMessage();
        return new UnifyResponse(10001, message, method + " " + requestUrl);
    }

    private String formatAllErrorMessages(List<ObjectError> errors) {
        StringBuffer errorMsg = new StringBuffer();
        errors.forEach(error ->
                errorMsg.append(error.getDefaultMessage()).append(';')
        );
        return errorMsg.toString();
    }
}

```

要知道要处理的异常类型，可以通过直接测试查看

MethodArgumentNotValidException：是当参数异常时抛出，配合@max、@min等注解使用

ConstraintViolationException：是配合Java原生的@Constraint注解 + Java原生的ConstraintValidator注解处理器 + Spring全局异常处理 使用

[https://docs.spring.io/spring-boot/docs/current/reference/html/]: 