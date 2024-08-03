---
# https://hexo.io/zh-cn/docs/front-matter
layout: post
title: API设计  # 必需
urltitle: API-design # 注意
date: 2024-06-17 02:37:00  # 2024-06-02 20:32:22 # 必需
updated: 2024-06-17 # 注意
comments: true  # 注意
tags: [API设计] # 注意
# categories: [] # 注意
permalink: null  # 覆盖永久链接
disableNunjucks: false  # 禁用Nunjucks插件标签
lang:  
published: true  # 是否发布 # 注意

# https://butterfly.js.org/posts/dc584b87/#Post-Front-matter
keywords: Web开发, API设计, RESTful API, GraphQL, gRPC # 3-5个 可中文 # 注意
description: RESTful API的最佳实践 # 注意
top_img:  # 注意
cover:  # 封面  # 注意
toc: true
toc_number: true # 是否显示额外数字标题  # 注意
toc_style_simple: true  # 文章页侧边栏只显示TOC
copyright: true # 显示版权(转载文章false)
copyright_author:
copyright_author_href:
copyright_url:
copyright_info:
mathjax: false # 是否加载 mathjax.js
katex: false # 是否加载 katex.min.css  # 注意
aplayer:  # aplayer音乐插件
highlight_shrink: false # 代码框是否缩略
aside: true  # 是否显示侧边栏
abcjs:  # abcjs乐谱渲染
series:  # 系列文章 # https://butterfly.js.org/posts/4aa8abbe/#series-%E7%B3%BB%E5%88%97%E6%96%87%E7%AB%A0

sticky:  # 置顶
---



# API设计

## HTTP/1

### REST原则

REST(representational state transfer 具象状态传输)

| 两个核心 | 释义                                                         |
| -------- | ------------------------------------------------------------ |
| RE       | 表层，也叫做资源表现层、资源的表现形式、具象，例如 JSON、XML 都属于表现层 |
| ST       | 状态存于客户端需要传递到服务端                               |

| 6个约束       | 释义                                                         |      |
| ------------- | ------------------------------------------------------------ | ---- |
| 使用 C/S 架构 | 资源通过URI(Uniform Resource Identifier)统一资源标识符表示，URL(Uniform Resource Locator)统一资源定位符)定位符是URI的子集<br />例如，前后端分离，页面和服务不在同一服务器上运行 |      |
| 统一接口      | 例如，GET、POST、PUT、DELETE                                 |      |
| 无状态        | 服务端并不会保存有关客户的任何状态(session)，也就是说要服务后端服务，就要带 token 过去<br />客户端通过JWT、OAuth维护状态 |      |
| 缓存          | GET一般缓存、POST不缓存<br />例如：服务端通过 token 缓存已登录过的用户信息，客户端请求会带一个 token 过来，后台服务通过带过来的 token 在缓存中取出用户信息 |      |
| 分层系统      | 例如， 一个父系统下有多个子模块，每个模块都是独立的服务      |      |
| 按需代码      | 例如，服务端返回JS拓展客户端功能                             |      |



#### RESTful API(C/S架构 CRUD)

RESTful API(符合REST原则的API)提供了一组设计原则和约束条件(不是标准)

应用场景：C/S架构
优点：URL简洁有层次，更易于实现缓存等机制
缺点：有些操作(sign_in)不好表示为资源、粒度不好把握(所有、单个、部分信息、整合信息)



实践：

`SpringMVC` 原生态的支持了 REST 风格的架构设计
所涉及到的注解:`@RequestMapping` `@PathVariable` `@ResponseBody`



##### 请求

请求协议：使用 HTTP 协议，并利用 HTTP 方法(GET、POST、PUT、DELETE)来对资源执行不同的操作
接口风格：使用基于资源的接口风格
响应数据格式：常用 JSON 等轻量级数据格式(XML重)

|                        | 非 Restful API URL                      | RESTful API URI                |
| ---------------------- | --------------------------------------- | ------------------------------ |
| 根据用户id查询用户数据 | [GET] http://127.0.0.1/user/query/1     | [GET] http://127.0.0.1/user/1  |
| 新增用户               | [POST] http://127.0.0.1/user/save       | [POST] http://127.0.0.1/user   |
| 修改用户信息           | [POST] http://127.0.0.1/user/update     | [PUT] http://127.0.0.1/user    |
| 删除用户信息           | [GET/POST] http://127.0.0.1/user/delete | [DELETE] http://127.0.0.1/user |

总结：操作与资源定位分离



##### HTTP API 成功和错误响应格式

方案一：业务状态用 http code 表示(不建议)
	1使用业务code在业务错误下是很有必要的，业务code表示业务状态、http code表示网络状态。即请求成功但是业务错误返回 {200, errcode, errmsg}
	2客户端部分请求库将非200定义为异常需要try catch

方案二：同时使用 http code 和业务code(成功返回data，失败返回4/5XX)(推荐)

方案三：使用200 + 业务code(目前240616国内很多)



RESTful API：用 HTTP 功能

[aws What is RESTful API?](https://aws.amazon.com/what-is/restful-api/#:~:text=RESTful%20API%20is%20an%20interface,applications%20to%20perform%20various%20tasks.)

[微软Azure API设计最佳实践](https://learn.microsoft.com/zh-cn/azure/architecture/best-practices/api-design)
[Azure REST API](https://github.com/microsoft/api-guidelines/blob/vNext/azure/Guidelines.md)

[Amazon SP-API](https://developer-docs.amazon.com/sp-api/docs/response-format#success-response)

[阮一峰 RESTful API 设计指南](https://www.ruanyifeng.com/blog/2014/05/restful_api.html)

[Blog#restful api最佳实现](https://www.vinaysahni.com/best-practices-for-a-pragmatic-restful-api)

[微信支付v3 API](https://pay.weixin.qq.com/wiki/doc/apiv3_partner/apis/chapter4_3_2.shtml)

[Google Cloud API#errors](https://cloud.google.com/apis/design/errors)



RPC：把 HTTP 当成纯传输协议用

[JSON RPC 2.0](https://www.jsonrpc.org/specification)

[拼多多](https://open.pinduoduo.com/application/document/api?id=pdd.order.information.get)

[抖音开发者平台](https://developer.open-douyin.com/docs/resource/zh-CN/mini-app/develop/server/trade-system/general/order/query_order#%E5%93%8D%E5%BA%94%E5%8F%82%E6%95%B0)



##### 接口设计(不属于REST原则)

1. 幂等性：是指对**同一个操作**的多次执行所产生的效果是相同的，不会因为**重复提交**而导致数据状态的改变。它确保了在进行数据操作时不会产生意外的副作用，从而保证了系统的稳定性和一致性。
   分布式环境下客户端实现重试或消息补偿机制重复发送同一个请求，这时应使用乐观锁

2. 统一响应(不推荐)

| 业务code | msg            |
| -------- | -------------- |
| 0        | 服务器处理成功 |
| 1XXX     | 参数异常       |
| 2XXX     | 数据库处理异常 |

3. 无状态
   不用session
   客户端存
   后端统一存储



### GraphQL(复杂数据类型)

应用场景：复杂类型数据(需要对后端数据整合，不然会获取不足或过度获取)



## RPC

### gRPC(微服务)

基于 HTTP2

优点：
Protocol Buffers 比 JSON 高效
支持双向流式传输

应用场景：分布式系统的微服务
浏览器没有暴露HTTP/2接口所以应用程序不能直接使用 gRPC
