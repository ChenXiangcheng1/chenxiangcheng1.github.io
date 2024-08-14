---
title: Springboot集成springdoc库
date: 2023-08-04
update: 2023-08-04

tags:
categories:
  - Spring
  - Spring Boot
  - 集成库
keywords: RESTful,OpenAPI,Swagger,SpringDoc
description: 主要关于OpenAPI，Swagger，SpringDoc
top_img:
cover:
---



# SpringDoc

## OpenAPI规范

[OAS(OpenAPI Specification) 官网](https://www.openapis.org/)	[OAS(OpenAPI Specification) github](https://github.com/OAI/OpenAPI-Specification)

OpenAPI 规范以前称为 Swagger 规范，定义了 RESTful API 的标准，后被 Linux 列为 API 标准，从而成为了一种行业标准。

目前(2023.8)最新的版本为 OpenApi3.1.0



### Swagger Core

Swagger 是一个实现了OpenAPI 规范的工具集。其中核心是`io.swagger.core.v3`(Swagger Core)用于生成和解析 OpenAPI3.x.x 规范的 API 文档

OpenAPI 文档本身是一个 JSON 对象，可以用 JSON 或 YAML 格式表示。



**UI 库**

当然你也可以直接使用

SpringFox Swagger UI 基于 Swagger2，最新的基于 OpenAPI3 来兼容 OpenAPI2 的注解，逐渐弃用

SpringDoc OpenAPI UI 基于 OpenAPI3

还有其他个人开发的 UI...



### springdoc-openapi

[homepage](https://springdoc.org/)	[github](https://github.com/springdoc/springdoc-openapi)

是 Spring 官方推出的 OpenAPI 规范的实现之一，能更方便地与Spring Boot集成自动配置，通过Swagger-Core生成 OpenAPI 文档，通过 Swagger-UI 来 Json 可视化并生成 Web 页面。

与 Spring REST DOC 生成静态文档不同，springdoc-openapi 生成动态文档



| swagger2                                                     | OpenAPI3                                                     | 注解位置                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ | -------------------------------------------- |
| @EnableSwagger2Doc<br />@EnableOpenApi  # Swagger3           |                                                              | 对SpringBootApplication使用                  |
| @Api(tags = "用户管理")                                      | @Tag(name = “接口类描述”)                                    | Controller 类上                              |
| @ApiOperation(value = "创建用户(方法作用)", notes = "根据User对象创建用户(方法备注说明)") | @Operation(summary =“接口方法描述”)                          | Controller 方法上                            |
| @ApiImplicitParams                                           | @Parameters                                                  | Controller 方法上                            |
| @ApiImplicitParam(paramType = "path", dataType = "Long", name = "id", value = "用户编号", required = true, example = "1") | @Parameter(description=“参数描述”)                           | Controller 方法上 @Parameters 里             |
| @ApiParam                                                    | @Parameter(description=“参数描述”)                           | Controller 方法的参数上                      |
| @ApiIgnore                                                   | @Parameter(hidden = true) 或@Operation(hidden = true) 或@Hidden |                                              |
| @ApiModel(description="用户实体")                            | @Schema                                                      | DTO(数据传输对象)类上，对bean使用            |
| @ApiModelProperty("用户编号")                                | @Schema                                                      | DTO(数据传输对象)属性上，对bean.property使用 |
|                                                              |                                                              |                                              |

```java
@ApiImplicitParams：用在请求的方法上，表示一组参数说明
    @ApiImplicitParam：用在@ApiImplicitParams注解中，指定一个请求参数的各个方面
        name：参数名
        value：参数的汉字说明、解释
        required：参数是否必须传
        paramType：参数放在哪个地方
            · header --> 请求参数的获取：@RequestHeader
            · query --> 请求参数的获取：@RequestParam
            · path（用于restful接口）--> 请求参数的获取：@PathVariable
            · div（不常用）
            · form（不常用）    
        dataType：参数类型，默认String，其它值dataType="Integer"       
        defaultValue：参数的默认值
        
@ApiResponses：用在请求的方法上，表示一组响应
    @ApiResponse：用在@ApiResponses中，一般用于表达一个错误的响应信息
        code：数字，例如400
        message：信息，例如"请求参数没填好"
        response：抛出异常的类
```



### swagger-spring-boot-starter

[swagger-spring-boot-starter 个人做的](https://github.com/SpringForAll/spring-boot-starter-swagger)

使用方式：1：添加依赖，2：添加基本注解，3：springboot配置(yml)相关内容

```xml
<!--添加swagger依赖-->
<dependency>
    <groupId>com.spring4all</groupId>
    <artifactId>swagger-spring-boot-starter</artifactId>
    <version>1.9.0.RELEASE</version>
</dependency>

<!--版本不一样Swagger3依赖-->
<dependency>
     <groupId>io.springfox</groupId>
      <artifactId>springfox-boot-starter</artifactId>
      <version>3.0.0</version>
</dependency>
```

```yml
# swagger2配置
swagger:
  title: spring-boot-starter-swagger  # 标题
  description: Starter for swagger 2.x  # 描述
  version: 1.4.0.RELEASE  # 版本
  license: Apache License, Version 2.0  # 许可证
  licenseUrl: https://www.apache.org/licenses/LICENSE-2.0.html  # 许可证URL
  termsOfServiceUrl: https://github.com/dyc87112/spring-boot-starter-swagger  # 服务条款URL
  contact.name: cxc  # 维护人
  contact.url: http://blog.didispace.com  # 维护人URL
  contact.email: 1093553592@qq.com  # 维护人email
  base-package: com.example  # swagger扫描的基础包，默认：全扫描
  base-path: /**  # 需要处理的基础URL规则，默认：/**
```

```java
// swagger3配置
@Configuration
public class Swagger3Config {
    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.OAS_30)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.withMethodAnnotation(ApiOperation.class))
                .paths(PathSelectors.any())
                .build();
    }
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("Swagger3接口文档")
                .description("更多请咨询服务开发者Ray。")
                .contact(new Contact("Ray。", "http://www.ruiyeclub.cn", "ruiyeclub@foxmail.com"))
                .version("1.0")
                .build();
}	}
```

Swagger2 文档 URL：http://localhost:8080/swagger-ui.html

Swagger3 文档 URL：http://localhost:8080/swagger-ui/

