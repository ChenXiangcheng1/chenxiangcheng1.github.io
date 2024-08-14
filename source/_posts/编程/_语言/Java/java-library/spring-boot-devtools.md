# spring-boot-devtools

[官方文档](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#using.devtools)	|	[热替换](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#using.running-your-application.hot-swapping)

提供自动重启、热替换(重新加载类无需重启整个应用程序)、静态资源LiveReload

使用 Spring-boot-devtools 的应用程序会在类路径上的文件更改时或 mvn compile 或者手动 build 构建项目后自动restart。
静态资产和视图模板不会restart，会live reload

restart 原理：两个类加载器 base classloader、restart classloader，重启只需要重新加载 restart classloader



## 配置

application-{profile}.properties

```properties
spring.devtools.restart.additional-paths=a.txt
```

