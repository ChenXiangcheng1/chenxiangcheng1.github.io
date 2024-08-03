---
# https://hexo.io/zh-cn/docs/front-matter
layout: post
title: HTTP/3  # 必需
urltitle: HTTP_3 # 注意
date: 2023-05-01 # 2024-06-02 20:32:22 # 必需
updated: 2023-07-19 # 注意
comments: true  # 注意
tags: [计算机网络, HTTP] # 注意
# categories: [] # 注意
permalink: null  # 覆盖永久链接
disableNunjucks: false  # 禁用Nunjucks插件标签
lang:  
published: true  # 是否发布 # 注意

# https://butterfly.js.org/posts/dc584b87/#Post-Front-matter
keywords: 计算机网络, HTTP/3, 请求和响应报文, HTTPS, TLS机制 # 3-5个 可中文 # 注意
description: 关于HTTP协议非常基础的内容 # 注意
top_img: https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_05_03_23_15.png # 注意
cover: https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_05_03_23_15.png # 封面  # 注意
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

# HTTP

HyperText Transfer Protocol 超文本传输协议
是所有www文件(html、css、js、png、mp4、text、pdf...)请求和应答的标准
以 ASCII 码传输



## HTTP/2

多路复用，浏览器不暴露HTTP/2接口

应用场景：gRPC就是基于HTTP/2的



## HTTP/3 

基于UDP
使用 QUIC 协议保证可靠传输
测试：https://quic.nginx.org/quic.html

TODO：有空深入了解 QUIC 协议



## 请求与响应过程

1. **DNS解析**：浏览器缓存、系统缓存、路由器缓存、IPS服务器缓存(通过DHCP得到)、根域名服务器缓存、顶级域名服务器缓存
2. **TCP连接建立会话**：三次握手
3. **发送HTTP请求报文**
4. **服务器处理请求并返回HTTP响应报文**
5. **浏览器解析渲染页面**
6. **关闭连接**：四次挥手

ARP广播得到目标MAC地址



## 请求报文

HTTP请求报文 = 请求行 + 请求头 + 请求正文

```HTTP请求
GET请求

GET / HTTP/1.1		（这是请求行，请求方法：GET，请求地址URI：/，HTTP协议版本：HTTP/1.1）
Host: www.imooc.com
Connection: keep-alive
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/80.0.3987.106 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9
Content-Length: 612						(这两项可以关注一下)
Content-Type: application/csp-report	(这两项可以关注一下)
Cookie: zg_did=%7B%22did%22%3A%20%2217009bab896318-048a9423947a59-b383f66-144000-17009bab897b54%22%7D; imooc_uuid=f4c4e666-7e46-450e-9ccf-83971de1f578; imooc_isnew_ct=1580711197; imooc_isnew=2; UM_distinctid=17014351aac33e-058c4c3fa0427d-b383f66-144000-17014351aad849; loginstate=1; apsid=Q4NzY1MTJkYzY3NWFjMDM1OWJkMWEzNzJjOTliNWYAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAODIwMTcyMgAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABzb2Z0cHl0aG9uQUlAMTYzLmNvbQAAAAAAAAAAAAAAADQyNzE5NDM0YjRhMmQ1MDNiNTc1NzJlMTNiODE1OTgAAQJBXgECQV4%3DMz; last_login_username=softpythonAI%40163.com; adv_#globalTopBanner_2749=1581569592688; Hm_lvt_f0cfcccd7b1393990c78efdeebff3968=1581309568,1581503187,1581569540,1581655164; IMCDNS=0; Hm_lpvt_f0cfcccd7b1393990c78efdeebff3968=1581696480; zg_f375fe2f71e542a4b890d9a620f9fb32=%7B%22sid%22%3A%201581696026643%2C%22updated%22%3A%201581696533607%2C%22info%22%3A%201581318647072%2C%22superProperty%22%3A%20%22%7B%5C%22%E5%BA%94%E7%94%A8%E5%90%8D%E7%A7%B0%5C%22%3A%20%5C%22%E8%B7%AF%E5%BE%84%E6%95%B0%E6%8D%AE%E7%BB%9F%E8%AE%A1%5C%22%2C%5C%22Platform%5C%22%3A%20%5C%22web%5C%22%7D%22%2C%22platform%22%3A%20%22%7B%7D%22%2C%22utm%22%3A%20%22%7B%7D%22%2C%22referrerDomain%22%3A%20%22class.imooc.com%22%2C%22cuid%22%3A%20%22w6yyP76B_Ss%2C%22%2C%22zs%22%3A%200%2C%22sc%22%3A%200%2C%22firstScreen%22%3A%201581696026643%7D; cvde=5e4618736308b-131	（这是消息报头,包含有关客户端环境及请求正文的信息，如请求正文长度、浏览器所用编码格式等）
     (空行)
(没有请求正文)
```

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_05_03_23_10.png" alt="image-20230503231024088" style="zoom:80%;" />

| 请求格式              | 释义                                                         |
| --------------------- | ------------------------------------------------------------ |
| 请求行                | 包含请求方法、请求地址和HTTP协议版本                         |
| 请求头部\消息报头     | 包含一系列的键值对                                           |
| 空行                  | 空                                                           |
| 请求正文\请求体(可选) | 正文可能有也可能没有，注意和消息报头之间有一个空行，get请求没有 |

| 请求方法 | 释义                                                         | 释义简写                           | 幂等性 | 安全性 |
| -------- | ------------------------------------------------------------ | ---------------------------------- | ------ | ------ |
| GET      | 通常只用于**读取数据**，要求服务器将URL定位的文档资源放在响应报文的数据部分，回送给客户端。<br/>例如，/index.jsp?id=100&op=bind。请求参数在“？”后面，长度受限制 | **获取资源**                       | 幂等   | 安全   |
| POST     | 向服务器发送数据并请求**处理数据返回处理结果**<br />默认格式 application/x-www-form-urlencoded，表示可传输编码汉字，但是对于英文效率变低 | **新建资源**（也可以用于更新资源） | 非幂等 | 不安全 |
| PUT      | **更新**服务器上**已有的资源**                               | **更新资源**                       | 幂等   | 不安全 |
| DELETE   | 请求服务器删除指定的资源                                     | **删除资源**                       | 幂等   | 不安全 |
| HEAD     | 与GET方法类似，从服务器获取资源信息，和GET方法不同是是，HEAD不含有呈现数据，仅仅是HTTP头信息。HEAD的好处在于，使用这个方法可以在不必传输全部内容的情况下，就可以获得资源的元信息(或元数据) |                                    |        |        |
| OPTIONS  | 预请求，服务器回传是否支持跨域请求 、资源所支持的请求源、方法、标头等信息 | 跨域**预检请求**                   |        |        |
| TRACE    |                                                              |                                    |        |        |
| CONNECT  |                                                              |                                    |        |        |



| 消息报头的键值对 | 释义                                                         |
| ---------------- | ------------------------------------------------------------ |
| **Accept**       | 用于指定客户端希望接收那些类型的信息，如text/html image application，application/json |
| Cookie           | 用于维护状态，可以做用户认证，服务器检验，是浏览器存储在用户电脑上的文本片段 |
| Host             | 请求报头域，指定被请求资源的Internet主机和端口号，通常是从HttpURL中提取出来的 |
| Connection       | 表示连接状态，keep-alive是持久连接，即TCP的连接默认是不关闭的，可以被多个请求复用，如果客户端和服务器发现对方有一段时间没有活动，他可以主动关闭连接 |
| Cache-Control    | 用于指定缓存的指令，no-cache，no-store，max-age=3(表示资源在本地缓存了3秒) |
| User-Agent       | 用于标识请求者的一些信息，比如浏览器类型，版本，操作系统，Mozilla是浏览器的规范 |
| Accept-Encoding  | 用于指定可接受的内容编码，即当前浏览器可以进行哪些内容的编码 |
| Accept-Language  | 用于指定可以接受的自然语言，当前浏览器可以解析哪些语言       |
|                  |                                                              |



### GET请求和POST请求的区别

从三个层面来解答

|              | GET                                                 | POST                                                   |
| ------------ | --------------------------------------------------- | ------------------------------------------------------ |
| HTTP报文层面 | 将请求参数放在URL中(路径参数)<br />对数据长度有限制 | 将请求参数放在报文体中(请求参数)<br />对数据长度没限制 |
| 数据库层面   | GET符合幂等性和安全性                               | 一般不符合幂等性                                       |
| 缓存         | 可以被CDN缓存                                       | 不被CND缓存                                            |

幂等性：对数据库的一次操作和多次操作获得的结果一致
安全性：对数据库的操作没有改变数据库中的数据



## 响应报文

HTTP响应报文 = 状态行 + 响应头 + 响应正文

```txt
HTTP/1.1 200 OK			(状态行，协议版本、状态码、简要描述)
Server: nginx
Date: Sat, 15 Feb 2020 03:57:21 GMT
Content-Type: text/html; charset=UTF-8  (响应头中该项必需指明，其他可选)
Transfer-Encoding: chunked
Connection: keep-alive
Vary: Accept-Encoding
Set-Cookie: imooc_isnew=2; expires=Sun, 14-Feb-2021 03:57:21 GMT; Max-Age=31536000; path=/; domain=.imooc.com
Set-Cookie: cvde=5e4618736308b-132; path=/; domain=.imooc.com
Content-Encoding: gzip		(响应头)
Response就是响应正文（可能是HTML页面，可能是Json数据）

Server：服务器用来处理请求的软件信息，如当前服务器在nginx中间件中
Date：时间信息
Content-Type：浏览器响应正文的数据类型（如application/json、text/html）  (响应正文)
```

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_05_03_23_15.png" alt="image-20230503231554614" style="zoom:80%;" />

| 响应格式 | 释义                                           |
| -------- | ---------------------------------------------- |
| 状态行   | 包含HTTP协议版本、状态码和状态描述，以空格分隔 |
| 响应头   | 即消息报头，包含一系列的键值对                 |
| 空行     | 空                                             |
| 响应正文 | 返回内容                                       |



### 响应状态码(重要)

| 状态码  | msg                           | Body Contents              | 释义                                                         |
| ------- | ----------------------------- | -------------------------- | ------------------------------------------------------------ |
| 1xx     | 指示信息                      |                            | 表示请求已接收，继续处理                                     |
|         |                               |                            |                                                              |
| 2xx     | 成功                          |                            | 表示请求已被成功接收、理解、接受                             |
| 200     | OK                            | 资源                       | 操作成功                                                     |
| 201     | Created                       | 资源,元数据                | 对象创建成功                                                 |
| 202     | Accepted                      | N/A(Not applicable 不适用) | 请求已经被接收                                               |
| 203     | Non-Authoritative Information |                            | 非认证消息                                                   |
| 204     | No Content                    | N/A                        | 操作已经执行成功，但是没有返回数据                           |
|         |                               |                            |                                                              |
| 3xx     | 重定向                        |                            | -要完成请求必须进行更进一步的操作                            |
| 301     | Moved Permanently             | link                       | 请求永久重定向，资源已被移除                                 |
| 302     | Moved Temporarily             |                            | 请求临时重定向                                               |
| 303     |                               | link                       | 重定向                                                       |
| 304     | Not Modified                  | N/A                        | 资源没有被修改，可以直接使用缓存的文件                       |
| 305     | Use Proxy                     |                            | 使用代理                                                     |
|         |                               |                            |                                                              |
| 4xx     | 客户端错误                    |                            | 请求有语法错误或请求无法实现                                 |
| **400** | Bad Request                   | 错误提示(消息)             | 参数列表错误(缺少, 格式不匹配)                               |
| **401** | Unauthorized                  | 错误提示(消息)             | 请求未经授权，这个状态代码必须和WWW-Authenticate报头域一起使用 |
| **403** | Forbidden                     | 错误提示(消息)             | 服务器收到请求，但是拒绝提供服务，服务器通常会在响应正文中给出不提供服务的原因，例如IP被禁、授权过期 |
| 404     | Not Found                     | 错误提示(消息)             | 请求资源不存在，例如输入了错误的URL                          |
| 405     |                               | 错误提示(消息)             | 不允许的 http 方法                                           |
| 409     |                               | 错误提示(消息)             | 资源冲突，或者资源被锁定                                     |
| 415     |                               | 错误提示(消息)             | 不支持的数据(媒体)类型                                       |
| 429     |                               | 错误提示(消息)             | 请求过多被限制                                               |
|         |                               |                            |                                                              |
| 5xx     | 服务器端错误                  |                            | 服务器未能实现合法的请求                                     |
| 500     | Internal Server Error         | 错误提示(消息)             | 服务器发生不可预期的错误                                     |
| 501     |                               | 错误提示(消息)             | 接口未实现                                                   |
| 502     | Bad Gateway                   |                            |                                                              |
| 503     | Server Unavailable            |                            | 服务器当前不能处理客户端的请求，一段时间后可能恢复正常       |
| 504     | Gateway Time-out              |                            | 网关超时                                                     |



## HTTPS

| 区别     | http                                                         | https    |
| -------- | ------------------------------------------------------------ | -------- |
| 本质     | 需要到**CA (Certificate Authority证书颁发机构)申请证书**(含公钥)，一般免费证书较少，因而需要一定费用 <br />HTTPS=HTTP+TLS，更安全<br />TLS：加密+身份认证+完整性保护(数字签名) |          |
| 加密     | 明文传输                                                     | 密文传输 |
| 默认端口 | 443端口                                                      | 80端口   |

openssh生成自签名证书(没有经过CA机构认证是不安全的)，cacert.pem证书文件、private.key私钥文件

```bash
openssl genrsa -out private.key 2048  # 生成私钥文件
openssl req -new -key private.key -out cert.csr  # 根据私钥生成证书签名请求文件CRS文件(Certificate Signing Request)
openssl x509 -req -in cert.csr -out cacert.pem -signkey private.key  # 使用私钥对证书申请进行签名从而生成证书文件(pem文件)
```



### TLS 的工作原理

**TLS 握手：使用了非对称加密和对称加密结合的方式。使用非对称加密来交换对称密钥，使用对称密钥加密和解密实际的数据**

1. 客户端向服务器发送HTTPS请求，并将支持的加密算法信息发送给服务器
2. 服务器选择一套浏览器支持的加密算法，并将SSL证书发送给客户端
3. 客户端通过**CA数字签名**验证SSL证书合法性，并生成对称密钥以数据进行加密，使用证书**公钥加密**对称密钥发送给服务器 
4. 服务器使用**私钥解密**得到对称密钥，验证哈希，对称密钥加密响应消息回发客户端
5. 客户端解密响应消息，并对消息进行验真(对源服务器的身份进行身份验证)，之后进行加密交互数据



SSL证书包含域名、CA的数字签名、公钥、期限等信息
公钥本质上是用于加密和签署数据的长字符串。使用公钥加密的数据只能用私钥解密。
对称加密是指加密解密使用同一个密钥。



#### 数字签名原理

数字签名是一种基于公开密钥加密技术的数字安全机制，用于验证数据的真实性和完整性，以及数据的发送者身份。

数字签名的原理可以概括为以下几个步骤：

1. 数据摘要：发送者对需要发送的数据进行**摘要处理** (哈希)，生成一个固定长度的消息摘要（message digest）。
2. 数字签名：发送者使用自己的**私钥**对消息摘要进行**加密，生成数字签名**（digital signature）。
3. 数字证书：发送者将数字签名和原始数据一起发送给接收者，并附带自己的数字证书（digital certificate）。
4. 数字证书验证：接收者使用数字证书中的**公钥**来**解密数字签名，得到消息摘要**。
5. 数据完整性验证：接收者对原始数据进行摘要处理，得到一个新的消息摘要，与解密得到的**消息摘要进行比对**，如果两个消息摘要相同，说明数据在传输过程中没有被篡改。
6. 发送者身份验证：接收者使用数字证书中的发件人信息来验证发送者的身份，以确保数据来自可信的来源。

需要注意的是，数字签名的安全性取决于私钥的保密性和公钥的真实性。因此，在实际应用中，数字签名通常需要由可信的第三方机构（如CA机构）来进行认证和管理，以确保数字证书和公钥的真实性和有效性。同时，数字签名也需要定期更新和管理，以应对各种安全威胁和风险。



## 弃用的内容

旧时代的东西：如今 Cookie 被 localStorage 替代；服务端使用Nginx无状态，不使用Session

### Cookie和Session的区别

Cookie数据存放在客户的**浏览器上**，Session数据放在**服务器上**
**Session**相对于Cookie更**安全** (Cookie欺骗)
若考虑**减轻服务器负担**，应当使用**Cookie**

Cookie：是服务器发给客户端的特殊信息，以文本的形式存放在客户端。
当客户端再次请求服务器时会携带 Cookie。服务器收到后能解析 Cookie 生成与客户端相对应的内容。
应用：记住密码、购物车

Session：是服务器端的机制，能在服务器上保存会话信息。
解析客户端请求获得session id，按需保存状态信息。

Session的实现方式：
使用Cookie来实现，Tomcat 服务器可用 set-cookie: JSESSIONID=xxx
在Cookie不可用时可以使用HTTP请求头中加入X-Session-ID、URL回写来、隐藏表单域、客户端证书方式实现



