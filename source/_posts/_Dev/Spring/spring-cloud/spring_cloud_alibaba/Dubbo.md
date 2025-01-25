# Dubbo

Dubbo :是一个RPC框架+自动注册与发现，`SOA框架 (Service-Oriented Architecture 面向服务架构)`

作为RPC：使用 Dubbo 协议进行`节点间通信`，Dubbo 协议默认使用 Netty 作为基础通信组件，长连接进行传输且支持各种传输协议，用于实现`各进程节点之间的内部通信`。

作为SOA：
具有服务治理功能，提供`服务的注册和发现`！`用zookeeper实现注册中心`！启动时候服务端会把所有接口注册到注册中心，
并且订阅configurators,服务消费端订阅provide，
configurators,routers,订阅变更时，zk会推送providers,configuators，
routers,启动时注册长连接，进行通讯！
consumer和provider启动后，后台启动定时器，发送统计数据到monitor（监控中心）！提供各种容错机制和负载均衡策略！！

![](https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting2/img/2023_09_25_02_00.jpeg)

## Dubbo 使用

[文档](http://dubbo.apache.org/zh/docs/)

[使用手册](https://dubbo.gitbooks.io/dubbo-user-book/content/)



### dubbo-admin.war 与 Tomcat

dubbo-admin.war 需要解压到 tomcat 的 webapps/ROOT 目录下，可以通过 http://192.168.231.133:8080/ 直接访问

webapps\ROOT\WEB-INF 下的 dubbo.properties 文件

```properties
dubbo.registry.address=zookeeper://127.0.0.1:2181
dubbo.admin.root.password=root
dubbo.admin.guest.password=guest
```



### 服务提供端

需要注册和提供的服务

```java
/**
 * xml方式服务提供者接口
 */
public interface ProviderService {

    String SayHello(String word);
}

public class ProviderServiceImpl implements ProviderService{

    public String SayHello(String word) {
        return word;
    }
}
```

#### 添加依赖

```xml
        <dependency>
            <groupId>com.alibaba.boot</groupId>
            <artifactId>dubbo-spring-boot-starter</artifactId>
            <version>0.2.0</version>
        </dependency>
```

#### 暴露接口(XML配置方法)

provider.xml

```xml 
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans        http://www.springframework.org/schema/beans/spring-beans.xsd        http://code.alibabatech.com/schema/dubbo        http://code.alibabatech.com/schema/dubbo/dubbo.xsd">

    
    <!--当前项目在整个分布式架构里面的唯一名称，计算依赖关系的标签-->
    <dubbo:application name="provider" owner="sihai">
        <!-- 下面的参数是可以不配置的，这里配置是因为出现了端口的冲突，所以配置。qos：Quality of Service 一种安全机制，当网络发生拥塞的时候，所有的数据流都有可能被丢弃；为满足用户对不同应用不同服务质量的要求，就需要网络能根据用户的要求分配和调度资源，对不同的数据流提供不同的服务质量：对实时性强且重要的数据报文优先处理；对于实时性不强的普通数据报文，提供较低的处理优先级，网络拥塞时甚至丢弃。 -->
        <dubbo:parameter key="qos.enable" value="true"/>
        <dubbo:parameter key="qos.accept.foreign.ip" value="false"/>
        <dubbo:parameter key="qos.port" value="55555"/>
    </dubbo:application>

    <!-- 监控中心配置， 用于配置连接监控中心相关信息，可以不配置，不是必须的参数。 -->
    <dubbo:monitor protocol="registry"/>

    <!--dubbo这个服务所要暴露的服务地址所对应的注册中心，address 是注册中心的地址，这里我们配置的是 N/A 表示由 dubbo 自动分配地址。或者说是一种直连的方式，不通过注册中心。-->
    <!--<dubbo:registry address="N/A"/>-->
    <dubbo:registry address="N/A" />
    <!-- <dubbo:registry address="zookeeper://localhost:2181" check="false"/> -->
    <!-- 如果zookeeper是集群的话 -->
    <!-- <dubbo:registry protocol="zookeeper" address="192.168.11.129:2181,192.168.11.137:2181,192.168.11.138:2181"/> -->

    <!--当前服务发布所依赖的协议；dubbo、webserovice、Thrift、Hessain、http-->
    <dubbo:protocol name="dubbo" port="20880"/>

    <!--服务发布的配置，需要暴露的服务接口 interface 是接口的包路径，ref 是配置的接口的 bean。-->
    <dubbo:service
            interface="com.sihai.dubbo.provider.service.ProviderService"
            ref="providerService"/>

    <!--接口的Bean bean定义-->
    <bean id="providerService" class="com.sihai.dubbo.provider.service.ProviderServiceImpl"/>

</beans>
```

类似 spring 的配置文件，而且，dubbo 底层就是 spring，服务交给 IOC 容器管理

#### 暴露接口(注解配置方法+yml)

```yml
dubbo:
  application:
    name: "chinasofti-manager-service"
  # 指定注册中心协议和地址
  registry:
    protocol: "zookeeper"
    address: "192.168.231.133:2181"
# 指定通信协议和接口
  protocol:
    name: "dubbo"
    port: 20880
```

```java
import com.alibaba.dubbo.config.annotation.Service;

@Service
@Component
public class UserServiceImpl implements UserService {
    
    // 需要dubbo和Spring的注解
```

#### 发布接口

```java
package com.sihai.dubbo.provider;

import com.alibaba.dubbo.config.ApplicationConfig;
import com.alibaba.dubbo.config.ProtocolConfig;
import com.alibaba.dubbo.config.RegistryConfig;
import com.alibaba.dubbo.config.ServiceConfig;
import com.alibaba.dubbo.container.Main;
import com.sihai.dubbo.provider.service.ProviderService;
import com.sihai.dubbo.provider.service.ProviderServiceImpl;
import org.springframework.context.support.ClassPathXmlApplicationContext;

import java.io.IOException;

/**
 * xml方式启动
 *
 */
public class App 
{
    public static void main( String[] args ) throws IOException {
        //加载xml配置文件启动
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("META-INF/spring/provider.xml");
        context.start();
        System.in.read(); // 按任意键退出
    }
}
```

Dobbo 暴露的 url

```url
dubbo://192.168.234.1:20880/com.sihai.dubbo.provider.service.ProviderService?anyhost=true&application=provider&bean.name=com.sihai.dubbo.provider.service.ProviderService&bind.ip=192.168.234.1&bind.port=20880&dubbo=2.0.2&generic=false&interface=com.sihai.dubbo.provider.service.ProviderService&methods=SayHello&owner=sihai&pid=8412&qos.accept.foreign.ip=false&qos.enable=true&qos.port=55555&side=provider&timestamp=1562077289380
```

### 服务消费端

消费 url

#### XML配置方法

consumer.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans        http://www.springframework.org/schema/beans/spring-beans.xsd        http://code.alibabatech.com/schema/dubbo        http://code.alibabatech.com/schema/dubbo/dubbo.xsd">

    <dubbo:application name="consumer" owner="sihai"/>

    <!--点对点的方式-->
    <dubbo:registry address="N/A" />
    <!--<dubbo:registry address="zookeeper://localhost:2181" check="false"/>-->

    <!--生成一个远程服务的调用代理-->
    <!--这里采用点对点的方式，所以，需要配置在服务端暴露的 url 。-->
    <dubbo:reference id="providerService"
                     interface="com.sihai.dubbo.provider.service.ProviderService"
                     url="dubbo://192.168.234.1:20880/com.sihai.dubbo.provider.service.ProviderService"/>
	<!-- 由于我们这里使用 zookeeper 作为注册中心，所以，跟点对点的方式是不一样的，这里不再需要 dubbo 服务端提供的 url 了，只需要直接引用服务端提供的接口即可。 -->
    <!--<dubbo:reference id="providerService"
                     interface="com.sihai.dubbo.provider.service.ProviderService"/>-->

</beans>
```

#### 注解配置方法（注解+yml）

```yml
dubbo:
  application:
    name: "chinasofti-manager-web"

  # 指定注册中心协议和地址
  registry:
    protocol: "zookeeper"
    address: "192.168.231.133:2181"
```

```
@Reference
private CategoryService categoryService;
```

#### 调用服务

```java
import com.alibaba.dubbo.config.ApplicationConfig;
import com.alibaba.dubbo.config.ReferenceConfig;
import com.alibaba.dubbo.config.RegistryConfig;
import com.sihai.dubbo.provider.service.ProviderService;
import org.springframework.context.support.ClassPathXmlApplicationContext;

import java.io.IOException;

/**
 * xml的方式调用
 *
 */
public class App 
{
    public static void main( String[] args ) throws IOException {

        ClassPathXmlApplicationContext context=new ClassPathXmlApplicationContext("consumer.xml");
        context.start();
        ProviderService providerService = (ProviderService) context.getBean("providerService");
        String str = providerService.SayHello("hello");
        System.out.println(str);
        System.in.read();

    }
}
```

