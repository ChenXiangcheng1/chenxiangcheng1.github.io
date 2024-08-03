# Spring MVC

Spring MVC 是基于 servlet 的 web 框架。
底层还是 DispatcherServlet 接收客户端请求并根据 RequestMapping 请求映射委托给相应的 Controller 处理。



## 项目目录结构

Spring MVC 项目的目录结构

```bash
${basedir}  		# 项目根目录，basedir为项目名
├── src
|	├── main
|	│ 	├── java
|	│ 	├── resources
|	│ 	|   	├── application.properties
|	│ 	|   	├── static  // 静态资源
|	│ 	|   	└── templates  // 动态模板
|	|	└── webapp  # 区别所在
|	└── test
├── target
|	└──classes
└── pom.xml
```

Spring Boot 项目的目录结构

```bash
${basedir}  		# 项目根目录，basedir为项目名
├── src
|	├── main
|	│ 	├── java
|	│ 	├── resources
|	│ 	|   	├── application.properties
|	│ 	|   	├── static  
|	│ 	|   	└── templates  
|	└── test
├── target
|	└──classes
└── pom.xml
```



## 返回

### 返回 ModelAndView

ModelAndView 是 Web MVC 框架中 Model 和 View 的持有对象。使只能单值返回的 Controller 可以返回 Model 和 View。
View 可以采用字符串名称的形式，需要由 ViewResolver 模板引擎解析并渲染，或者直接指定 View 对象。
Model 是一个 Map 可以键入多个对象。



`ModelAndView()` 无参构造，之后还需要填充 Bean 属性

`ModelAndView(String)` 当没有 Model 数据需要公开时，参数 viewName
`viewName = "/view.jsp"` 请求转发，共享 Request
`viewName = "redirect:/view.jsp"` 请求重定向

`ModelAndView(String,String,Object)` 单个 Model 对象的构造函数，参数分别为 viewName、modelName、modelObject

`ModelAndView(String,Map<String,?>,HttpStatusCode)`  

`addObject(String,Object):ModelAndView` 添加属性到 Model

`addAllObjects(Map<String,?>):ModelAndView` 添加 map 中所有属性到 Model

`setViewName(String):void`

`setView(View):void`

`setStatus(HttpStatusCode):void` 设置响应的 HTTP 状态



#### 动态模板引擎

Spring MVC 框架中 ViewResolver 默认使用是 JSP 作为动态模板引擎



##### JSP

[JavaEE笔记#JSP](../../../Java/JDK21.md#组件)

###### 请求转发和请求重定向的区别

请求转发

* 共享 Request

`viewName = "/view.jsp"` 

请求重定向

* 当数据处理逻辑和页面无关时使用

`viewName = "redirect:/view.jsp"` 请求重定向



##### Freemarker

[官网](https://freemarker.apache.org/)	|	[github](https://github.com/apache/freemarker)

```xml
<!-- applicationContext.xml -->
<bean id="ViewResolver" class="org.springframework.web.servlet.view.freemarker.FreeMarkerViewResolver">
    <property name="contentType" value="text/html;charset=utf-8"/>
    <property name="suffix" value=".ftl"/>
</bean>
<bean id="freemarkerConfig" class="org.springframework.web.servlet.view.freemarker.FreeMarkerConfigurer">
    <property name="templateLoaderPath" value="/WEB-INF/ftl"/>
    <property name="freemarkerSettings">
        <props>
            <prop key="defaultEncoding">UTF-8</prop>
        </props>
    </property>
</bean>
```

```txt
webapp
├─um
├─WEB-INF
│  ├─ftl
│  │ └─view.ftl  # Freemaker
│  └─web.xml
├─index.html
└─view.jsp  # JSP
```



### 返回序列化对象

spring-mvc 内置 jackson-databind 序列化库

[Jackson笔记](../../../java-library/jackson-databind.md)



### 返回ResponseEntity\<T>

用于添加 HttpStatusCode 状态代码到**响应实体**

`ResponseEntity(T,MultiValueMap<String,String>,HttpStatusCode)` 参数分别为 body、headers、status

`getHeaders():HttpHeaders`

`getBody():T`

`getStatusCode():HttpStatus`

`ResponseEntity.status(HttpStatusCode)` 设置 status

`accepted():BodyBuilder` 202

`badRequest():BodyBuilder` 400

`created(URI):BodyBuilder` 201

`internalServerError():BodyBuilder` 500

`noContent():BodyBuilder<?>` 204

`notFound():BodyBuilder<?>` 404

`ok():BodyBuilder` 200

`ok(T):ResposneEntity<T>` 200

`unprocessableEntity():BodyBuilder` 422



**ResponseEntity.BodyBuilder**

用于添加 Body 到响应实体，继承自 HeadersBuilder 也能添加 header

`body(T):ResponseEntity<T>` 设置响应的 Response body

`contentLength(long):ResponseEntity.BodyBuilder` 设置响应的 Content-Length

`contentType(MediaType):ResponseEntity.BodyBuilder` 设置响应的 Content-Type



**HeadersBuilder<B extends HeadersBuilder<B>>**

用于添加 header 到响应实体

`header(String,String...):B` 在响应头的给定名称下添加 header 值

`headers(HttpHeaders):B` 



示例1：

```java
return new ResponseEntity<String>("Hello World", responseHeaders, HttpStatus.CREATED);
```

示例2：

```java
return ResponseEntity.status(HttpStatusCode.OK).header(HttpHeaders.ACCEPT).body(user);
```

示例3：

```java
return ResponseEntity.created(location).header("MyResponseHeader", "MyValue").body(null);
```





## 同源策略

同源策略是浏览器(chrome、firefox...)的策略
同源：协议、域名、端口一致。http://localhost 与 http://127.0.0.1 不同源
同源策略只允许网页脚本从同一源获取数据
	网页脚步不能访问其他源的 DOM(HTML、XML、XHTML)
	XMLHttpRequest、Fetch API 不允许跨域请求
	网页脚步的网络通信不允许获取其他源的响应



HTML中能够跨域访问的标签：<script>、<img>、<link> 
<a></a> 超链接不属于跨域请求



### 启用跨域资源共享

服务端启用跨域资源共享 CORS (Cross-Origin Resource Sharing) 是浏览器跨域解决方案之一



**简单请求** - 即使跨域也不会触发预请求的请求

* 限制请求方法为 GET、POST、HEAD
* 无自定义标头

浏览器在发起**跨域请求**前先发送一个 **OPTIONS 预检请求**，服务器通过设置 CORS 标头来说明是否允许跨域请求以及支持的请求源、方法、标头等信息。如果服务器通过了预检请求，则浏览器之后的跨域请求不需要预检请求了。

预检请求响应的标头
access-control-request-method: 允许的请求方法
access-control-request-headers: 允许的请求头
access-control-request-origin: 允许的请求源，还可以设置为 *(不推荐



配置1 - JavaConfig

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**") // 匹配所有请求路径
                .allowedOrigins("*") // 允许所有来源
                .allowedMethods("GET", "POST", "PUT", "DELETE") // 允许的请求方法
                .allowedHeaders("*"); // 允许的请求头
    }
    
    @Bean
    public WebConfig webConfig() {
        return new WebConfig();
    }
}
```



配置2 - 注解

@CrossOrigin(origin = {"http://localhost:8080", "xx"})



配置3 - XML applicationContext.xml

不推荐 XML(很烦的，需要声明命名空间)

```xml
<mvc:cors>
    <mvc:mapping path="/restful/**"
                 allowed-origins="http://localhost:8080,http://www.imooc.com"
                 max-age="3600"/>
</mvc:cors>
```



## 发送 HTTP 请求

### HttpURLConnection

用于发送单个请求



### RestTemplate

同步阻塞地发送 HTTP 请求，调用接口。 将被弃用。

`getForObject(String,Class<T>,Object...):T` get请求返回T

`postForObject(String,Object,Class<T>,Object...):T` post请求返回T

`getForEntity(String,Class<T>,Object...):ResponseEntity<T>`  get请求返回ResponseEntity<T>

示例：

```java
RestTemplate rest = new RestTemplate();
String sessionText = rest.getForObject(url, String.class);  // 参数1：url，参数2：要接收的类型，返回：JSON字符串
try {
    Map<String, Object> session = mapper.readValue(sessionText, Map.class);  // JSON反序列化为对象
} catch (JsonProcessingException e) {
    e.printStackTrace();
}
```



### WebClient

org.springframework.web.reactive.client.WebClient



## 拦截功能

filter
filter基于 servlet 规范	 [JavaEE笔记#Servlet](.../../../Java/JDK21.md#十三种规范)

Spring AOP
Spring AOP 是针对方法进行拦截



### HandlerInterceptor

拦截器针对所有 HTTP 请求进行预处理和后处理，可以实现**身份验证**、**日志记录**、**权限校验**
HandlerInterceptor 底层通过 Spring AOP 实现

```java
public class MyInterceptor implements HandlerInterceptor {

    private static final Logger logger = LoggerFactory.getLogger(AccessHistoryInterceptor.class);
    
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        // 在请求处理之前执行的操作
        // Object handler表示要访问方法
        System.out.println("MyInterceptor: " + request.getRequestURL()+" - 准备执行");
        StringBuilder log = new StringBuilder();
        log.append(request.getRemoteAddr());
        log.append("|");
        log.append(request.getRequestURL());
        log.append("|");
        log.append(request.getHeader("user-agent"));        
        logger.info(log.toString());  // logger.info 对日志进行输出（要和logback.xml中logger配置的等级一样）
        return true; // 返回true表示继续处理请求，返回false表示中断请求
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        // 在请求处理之后、视图渲染之前执行的操作
        System.out.println("MyInterceptor: " + request.getRequestURL()+" - 目标处理成功");
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        // 在请求完成并且视图渲染完成后执行的操作
        System.out.println("MyInterceptor: " + request.getRequestURL()+" - 响应内容已产生");
    }
}
```

```java
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new cn.cxc.framework.interceptor.MyInterceptor()).addPathPatterns("/**");
        WebMvcConfigurer.super.addInterceptors(registry);
    }
    @Bean
    public WebConfig webConfig() {
        return new WebConfig();
    }
}
```



## Spring MVC 框架处理流程

![1610506643787](https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting2/img/2023_09_23_03_21.png)

Interceptor - 拦截器，可以在请求处理的不同阶段插入自定义的逻辑处理

DispatcherServlet - 是 Spring MVC 的前端控制器，负责接收所有的 HTTP 请求并将请求分发到相应的处理器

Handler Mapping - 处理器映射，将请求的 URL 路径和其他条件，将请求映射到相应的控处理器。

Handler Adapter - 处理器适配器，协调控制器的执行，适配器遇到不同的处理器(拦截器、Controller) 会执行不同的方法，进行参数适配数据绑定(将HTTP请求参数绑定到模型对象)验证

View Resolver - 视图解析器，根据视图的逻辑名称解析视图对象，并将模型数据渲染到视图上。



## 问题

### 父子上下文重叠问题

父子上下文重叠问题：配置类重复扫描
解决：重叠对象放在启动类以外的包目录下

父子上下文重叠问题：事务管理的 Service 被Spring、SpringMVC 重复扫描()
解决：扫描排除



**总结**

有的组件具有子上下文，需要避免被扫描



**父子上下文**

父 context 容器中保存数据源、服务层、DAO层、事务的Bean。
Web容器(Tomcat)上下文：ServletContext  
根上下文：WebApplicationContext

```xml
<!-- application.xml父上下文容器 -->
<!-- 启动组件扫描，排除@Controller组件，该组件由SpringMVC配置文件扫描 -->
<context:component-scan base-package="com.will">
	<context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/
</context:component-scan>
```

子 context 容器中保存 MVC 相关的 Controller 的 Bean
DispatcherServlet，前端控制器

```xml
<!-- springmvc.xml子上下文容器 -->
<!-- spring mvc 的配置文件里只扫描controller -->
<context:component-scan base-package="com.will.controller"/>
```



## 身份认证

session+cookie

缺点：
紧耦合，前端不能独立测试开发部署
扩展性差，分布式需要共享会话
有状态



### JWT

JSON Web Token，配合拦截器使用



### OAuth2

可以实现 SSO(Single Sign-On) 单点登入

https://www.zhihu.com/question/392620649





## 配置

### JavaConfig

#### WebMvcConfigurer

```java
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new cn.cxc.framework.interceptor.MyInterceptor()).addPathPatterns("/**");
        WebMvcConfigurer.super.addInterceptors(registry);
    }

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**") // 匹配所有请求路径
                .allowedOrigins("*") // 允许所有来源
                .allowedMethods("GET", "POST", "PUT", "DELETE") // 允许的请求方法
                .allowedHeaders("*"); // 允许的请求头
    }

    @Bean
    public WebConfig webConfig() {
        return new WebConfig();
    }
}
```



#### WebMvcRegistrations

通过实现`WebMvcRegistrations` 和 `HandlerExceptionResolver`接口，

全局异常处理：可以返回自定义的`ExceptionHandlerExceptionResolver`实例来处理全局异常。可以将异常统一处理，并生成适当的错误响应，以便前端能够正确处理异常情况。

请求参数解析器和响应结果封装：可以返回自定义的`RequestMappingHandlerAdapter`实例，其中可以注册自定义的请求参数解析器和响应结果封装器。这样可以实现对请求参数的自动解析和响应结果的统一封装，减少前后端交互的工作量。

```java
@Configuration
public class CustomWebMvcRegistrations implements WebMvcRegistrations {

    @Override
    public RequestMappingHandlerMapping getRequestMappingHandlerMapping() {
        return WebMvcRegistrations.super.getRequestMappingHandlerMapping();
    }
    
    @Override
    public RequestMappingHandlerAdapter getRequestMappingHandlerAdapter() {      
        return WebMvcRegistrations.super.getRequestMappingHandlerAdapter();
    }
    
    @Override
    public ExceptionHandlerExceptionResolver getExceptionHandlerExceptionResolver() {
        // return WebMvcRegistrations.super.getExceptionHandlerExceptionResolver();
        return new CustomExceptionHandlerExceptionResolver();
    }

    private static class CustomExceptionHandlerExceptionResolver extends ResponseEntityExceptionHandler {

        @Override
        protected ResponseEntity<Object> handleExceptionInternal(Exception ex, Object body, HttpHeaders headers, HttpStatus status, WebRequest request) {
            // 在这里编写自定义的异常处理逻辑
            // 可以根据异常类型、请求信息等进行特定的处理

            // 例如，生成自定义的错误响应
            ErrorResponse errorResponse = new ErrorResponse();
            errorResponse.setMessage("An error occurred");
            errorResponse.setStatus(status.value());

            // 返回自定义的错误响应
            return new ResponseEntity<>(errorResponse, headers, status);
        }
    }

    private static class ErrorResponse {
        private String message;
        private int status;
        // Getters and setters
    }

    @Bean
    public CustomWebMvcRegistrations customWebMvcRegistrations() {
        return new CustomWebMvcRegistrations();
    }
}
```



### XML

部署描述符 web.xml 

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">
    
    <!--1. DispatchServlet初始化Spring上下文容器-->
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <!--
            DispatcherServlet是Spring MVC最核心的对象
			初始化上下文容器
            DispatcherServlet用于拦截Http请求,
            并根据请求的URL调用与之对应的Controller方法,来完成Http请求的处理
        -->
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!--applicationContext.xml 指向上下文容器配置文件路径-->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:applicationContext.xml</param-value>
        </init-param>
        <!--
            在Web应用启动时自动创建Spring IOC容器,
            并初始化DispatcherServlet
			不写会在第一次访问 url 时创建IoC容器
        -->
        <load-on-startup>0</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <!--"/" 代表拦截所有请求-->
        <url-pattern>/</url-pattern>
    </servlet-mapping>

    <!-- 2.解决Post请求乱码问题 -->
    <filter>
        <filter-name>characterFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>characterFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
    
    <!-- 默认SpringMVC无法获得PUT DELETE请求中的表单数据，可通过Jackson获得其中的JSON数据，可处理POST请求中的表单数据 -->
    <!-- SpringBoot中可以获得PUT、DELETE请求中的表单数据 -->	
    <!-- 3. 配置表单过滤器，使得能够获得Put、Delete请求中的表单数据 -->
	<filter>
        <filter-name>formContentFilter</filter-name>
        <filter-class>org.springframework.web.filter.FormContentFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>formContentFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>   
</web-app>
```

applicationContext.xml 用于定义和配置 Spring 应用程序的组件、依赖关系、拦截器、视图解析器等。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:mv="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/context
            http://www.springframework.org/schema/context/spring-context.xsd
            http://www.springframework.org/schema/mvc
            http://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <!-- 0. 导入别的配置 -->
    <import resource="spring-dao.xml"/>
    
    <!-- 1. 包扫描 -->
    <context:component-scan base-package="com.imooc.springmvc"></context:component-scan>

    <!-- 2.将图片/JS/CSS等静态资源排除在DispatcherServlet之外，可提高执行效率 (静态资源没必要给DispatcherServlet拦截)，其他AOP拦截器还是会拦截静态资源 -->
    <mvc:default-servlet-handler/>
    
    <!--3. 启用Spring MVC的注解开发模式 和 4. 日期转换-->
    <mvc:annotation-driven conversion-service="conversionService">
        <!-- 5. 解决乱码问题，配置StringHttpMessageConverter，改Response中的编码 -->
        <mvc:message-converters>
            <bean class="org.springframework.http.converter.StringHttpMessageConverter">
                <property name="supportedMediaTypes">
                    <list>
                        <!-- response.setContentType("text/html;charset=utf-8") -->
                        <value>text/plain;charset=utf-8</value>
                        <!-- 遇到text/html;改为utf-8 -->
                        <value>text/html;charset=utf-8</value>
                    </list>
                </property>
            </bean>
        </mvc:message-converters>
    </mvc:annotation-driven>

    <!-- 4. 日期转换 -->
    <bean id="conversionService" class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
        <property name="converters">
            <set>
                <bean class="com.imooc.springmvc.converter.MyDateConverter"/>
            </set>
        </property>
    </bean>

    <!-- 使用jsp视图解析器
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="viewClass" value="org.springframework.web.servlet.view.JstlView"/>
        <property name="prefix" value="/WEB-INF/pages/"/>
        <property name="suffix" value=".jsp"/>
    </bean>  -->
    
    <!-- 6. 配置视图解析器，告诉SpringMVC使用Freemarker模板引擎 -->
	<bean id="ViewResolver" class="org.springframework.web.servlet.view.freemarker.FreeMarkerViewResolver">
        <!-- 设置响应输出（渲染完成时，向客户端返回 Response 中所使用的字符集编码），解决中文乱码 -->
        <property name="contentType" value="text/html;charset=utf-8"/>
        <!-- 指定 FreeMarker 模板文件扩展名，等效于在mvc:annotation-driven配置StringHttpMessageConverter -->
        <property name="suffix" value=".ftl"/>
    </bean>

    <bean id="freemarkerConfig" class="org.springframework.web.servlet.view.freemarker.FreeMarkerConfigurer">
        <!-- 设置模板保存的目录 -->
        <property name="templateLoaderPath" value="/WEB-INF/ftl"/>
        <!-- 其他模板引擎设置 -->
        <property name="freemarkerSettings">
            <props>
                <!-- 设置Freemarker脚本与数据渲染时使用的字符集 -->
                <prop key="defaultEncoding">UTF-8</prop>
            </props>
        </property>
    </bean>
    
    <!-- 7. 配置全局同源策略 -->
    <mvc:cors>
        <mvc:mapping path="/restful/**"
                     allowed-origins="http://localhost:8080,http://www.imooc.com"
                     max-age="3600"/>
    </mvc:cors>
    
    <!-- 配置拦截器 -->
   	<mvc:interceptors>
		<mvc:interceptor>
            <!-- 对/restful下所有请求拦截 -->
            <mvc:mapping path="/restful/**"/>
            <mvc:mapping path="/webapi/**"/>
            <!-- 拦截器Aop排除静态资源，拦截器不处理 -->
            <mvc:exclude-mapping path="/**.ico"/>
            <mvc:exclude-mapping path="/**.jpg"/>
            <mvc:exclude-mapping path="/**.gif"/>
            <mvc:exclude-mapping path="/**.js"/>
            <mvc:exclude-mapping path="/**.css"/>
            <mvc:exclude-mapping path="/resources/**"/>
            <!-- 指明哪个类对请求进行拦截处理 -->
            <bean class="com.imooc.restful.interceptor.MyInterceptor"/>
		</mvc:interceptor>
	</mvc:interceptors>      
</beans>
```



## 注解

### 处理请求与响应

#### Controller

| 注解                                                         | 元注解Target | 释义                                                         | 成员变量                                                     |
| ------------------------------------------------------------ | ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| @RestController                                              |              | 相当于@ResponseBody ＋ @Controller，默认返回json格式         |                                                              |
| `@ResponseBody`                                              | TYPE,METHOD  | **将方法的返回值绑定到 HTTP 响应的 response body 中**，常将对象通过 HttpMessageConverter 转为 JSON 返回<br />**默认返回视图名称，走视图处理器** |                                                              |
| `@RequestMapping(value = {"/category"}, method = {RequestMethod.GET})` | TYPE,METHOD  | 将web请求映射到请求处理类中的方法，通过将 URI 与方法绑定，Spring MVC 便可通过 Tomcat对外暴露服务<br />子注解：`Get`、`Post`、`Delete`、`Put`、`Patch`... | path(value)：路径映射的URI，例如 "/test"<br />params: 参数限制<br />handers: 请求头 |
| `@GetMapping`                                                |              |                                                              |                                                              |
| `@PostMapping`                                               |              |                                                              |                                                              |
| @CrossOrigin(origin = {"http://localhost:8080", "xx"}, maxAge = 3600) |              | 允许哪些域名，向你的端口访问<br />maxAge 缓存预检请求        |                                                              |
| `@ResponseStatus()`                                          | TYPE,METHOD  | 用HTTP状态码和异常原因标记一个方法或异常类，该状态码不会覆盖 ResponseEntity 的异常状态码 | code : HTTP 状态码，默认HttpStatus.INTERNAL_SERVER_ERROR<br />reason: 异常原因字符串 |
|                                                              |              |                                                              |                                                              |
| @ControllerAdvice                                            | TYPE         | 用于注册@ExceptionHandler、@ModelAttribute、@InitBinder Bean到Spring容器 |                                                              |
| @ExceptionHandler                                            | METHOD       | 用于全局异常处理                                             | 异常类型(value)                                              |
| @ModelAttribute                                              |              | 用于全局数据绑定                                             |                                                              |
| @InitBinder                                                  |              | 用于全局数据预处理                                           |                                                              |



### 处理请求

#### 绑定方法参数

**处理Get请求**

请求：http://example.com/path/test1/test2
@GetMapping("/path/{param1}/{param2}")  // 路径变量分别为param1=test1、param2=test2
使用: **@PathVariable** (最佳)

请求：http://example.com/path?param1=value1&param2=value2
请求参数分别为param1=value1、param2=value2
使用：**@RequestParam**

**处理Post请求**

请求：表单提交，Content-Type 为 application/x-www-form-urlencoded
使用：**@ModelAttribute**

请求：JSON提交，Content-Type 为 application/json 或 application/xml
使用：**@RequestBody** (最佳)

还可以：HttpServletRequest 

| 注解                | 元注解Target                                          | 释义                                                         | 成员变量                                                     |
| ------------------- | ----------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `@PathVariable`     | PARAMETER                                             | **绑定方法参数到 URI 模板变量**<br />若方法参数类型为 Map 将注入<variable,Value> | **name: 路径变量名称**<br />required：是否必须               |
| `@RequestParam`     | PARAMETER                                             | **绑定方法参数到 web 请求参数**<br />若方法参数类型为 Map 将注入<请求参数,Value> | **name：请求参数名称**<br />required：是否必须<br />defaultValue: 方法参数的默认值 |
| `@RequestBody`      | PARAMETER                                             | **绑定方法参数到 Web 请求正文**body<br />使用 HttpMessageConverter 解析 JSON 然后封装 | required：是否必须                                           |
| @ModelAttribute     | PARAMETER、METHOD                                     | **绑定方法参数或方法返回到 Model 属性**                      |                                                              |
| `@DateTimeFormat()` | METHOD<br />FIELD<br />PARAMETER<br />ANNOTATION_TYPE | 声明字段或方法参数应格式化为日期或时间<br />String 转 Date、Calendar、Long | pattern：自定义模式，例如 "yyyy-MM-dd"                       |



#### 验证

##### JSR-303 Bean Validation

Java Specification-Requests Java请求规范

| 注解                                        | 元注解Target | 释义                                                         | 成员变量 |
| ------------------------------------------- | ------------ | ------------------------------------------------------------ | -------- |
| @NotNull                                    |              | 只是一个标记，不会进行校验<br />区别：<br />JSR的只是一个标记，不会进行校验<br />Lombok的是编译时校验<br />Spring的是运行时校验 |          |
| @Validated                                  |              | 修饰类，修饰参数，验证开关，配合其他参数校验注解使用，会抛出ConstraintViolationException异常，可配合ValidationMessages.properties文件输出异常信息 |          |
| @Max(value="10", message="不可以超过10")    |              | 修饰参数，最大不超过10，会抛出MethodArgumentNotValidException异常 |          |
| @Min                                        |              |                                                              |          |
| @Range(min=1, max=10, message= "123321")    |              |                                                              |          |
| @Length(min=2, max = 10, message = "xxxxx") |              | 对DTO成员变量使用，修饰参数                                  |          |
| @Valid                                      |              | 修饰成员变量对象，实现级联校验                               |          |
| @NotBlank                                   |              | 修饰参数，不为空                                             |          |
| @Positive(message = "错误信息")             |              | 正                                                           |          |
| @Size                                       |              |                                                              |          |
| @Digits                                     |              |                                                              |          |



##### Spring 注解

| 注解        | 元注解Target             | 释义                                                         | 成员变量 |
| ----------- | ------------------------ | ------------------------------------------------------------ | -------- |
| `@NonNull`  | METHOD、PARAMETER、FIELD | 运行时进行参数校验，以确保传入的参数不为null。<br />区别：<br />JSR的只是一个标记，不会进行校验<br />Lombok的是编译时校验<br />Spring的是运行时校验 |          |
| `@Nullable` | METHOD、PARAMETER、FIELD | 能为空<br />常用于参数校验                                   |          |



## Spring 工具类

### BeanUtils



### StringUtils



