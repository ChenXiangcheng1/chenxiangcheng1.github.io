# Jackson

同类产品比较：Jackson 比 Gson 好，Fastjson 漏洞多

进行序列化的对象必须有 Getter 方法



## 配置

application-{profile}.yml

```yml
spring:
  jackson:
    time-zone: GMT+8
    date-format: yyyy-MM-dd HH:mm:ss
    property-naming-strategy: LOWER_CAMEL_CASE  # Java对象属性名转Json字段名的命名策略
    # LOWER_CAMEL_CASE 小驼峰 myVarName (默认)
    # UPPER_CAMEL_CASE 大驼峰 MyVarName
    # SNAKE_CASE 蛇形 my_var_name
    # UPPER_SNAKE_CASE 大蛇形 MY_VAR_NAME
    # LOWER_CASE 小写 myvarname
    # KEBAB_CASE 烤肉 my-var-name
    # LOWER_DOT_CASE 小写&点 my.var.name
    serialization:
      WRITE_DATES_AS_TIMESTAMPS: true  # flase(默认) 返回UTC格林尼治时间，true 返回时间戳(ms毫秒 13位)
```



## 注解

| 注解          | 元注解Target                                                 | 释义                                                 | 成员变量                                                     |
| ------------- | ------------------------------------------------------------ | ---------------------------------------------------- | ------------------------------------------------------------ |
| `@JsonFormat` | ANNOTATION_TYPE<br />FIELD，例如 Bean 的 Date 类型成员变量<br />METHOD<br />PARAMETER<br />TYPE | 通用注释(没有特定解释)，用于表明如何序列化的配置信息 | pattern: 细格式化，例如"yyyy-MM-dd HH:mm:ss"<br />timezone: 序列化器的时区，例如"GMT+8" |





# 未整理

 JPA 笔记中 Jackson 序列化为 List/Map

