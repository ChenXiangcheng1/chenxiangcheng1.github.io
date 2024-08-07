# IPC

Inter-Process Communication 进程间通信，是为了解决资源隔离的不同进程间互访资源问题。这些进程可以是在同一台计算机上，也可能是在网络联通的不同计算机上。

根据进程所处位置不同，进程间通信的方法包括两类：

- 本地过程调用(**LPC**，Local Procedure Call)	LPC 用在多任务操作系统中，使得同时运行的任务能互相会话。这些任务共享内存空间使任务同步和互相发送信息。
- 远程过程调用(**RPC**，Remote Procedure Call)    RPC 是一种进程间通信方式。它允许程序调用另一个地址空间（通常是共享网络的另一台机器上）的过程或函数，而不用程序员显式编码这个远程调用的细节。即程序员无论是调用本地的还是远程的，本质上编写的调用代码基本相同。
  RPC 中间最重要的几个概念点，分别是**传输协议**，编码协议（**序列化协议**）以及 **IO模型**。这三个也是影响 RPC 框架选型的重要因素。
  传统 HTTP 服务器就是采用 BIO。例如，Tomcat (基于 Http 协议的 Servlet 容器，tomcat6.x 开始支持 NIO 模式)
  HTTP 服务器之所以称为 HTTP 服务器，是因为编码解码协议是 HTTP 协议



## IO 模型

[Java笔记#IO机制](../../Java/类库APIs/Java的IO机制.md)



## LPC

本地过程调用，Local Procedure Call



## RPC 框架

远程过程调用框架，Remote Procedure Call 框架，RPC 是目前主流的 WEB 服务实现方式。

一个典型 RPC 框架如下，包括几个组件：

* **服务提供者**：远程服务的被调用方，提供服务实现。
* **服务消费者**：远程服务的调用方。
* **注册中心**：提供服务的注册和发现。
* **调用监控**：监控远程服务调用情况。



Spring Cloud

Dubbo

gRPC



