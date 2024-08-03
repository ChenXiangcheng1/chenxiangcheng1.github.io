

# spring-boot-starter-actuator

[中文文档#生产就绪功能](https://springdoc.cn/spring-boot/actuator.html#actuator.endpoints.enabling)



## 配置

application-{profile}.yml

```yml
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
        include: "*"
  info:
    env:
      enabled: true  # 暴露 Environment 中名称以 info. 开头的任何属性
```



## 端点

http://localhost:8080/actuator



