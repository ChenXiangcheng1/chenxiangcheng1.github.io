---
title: 计算机网络
date: 2023-05-01
update: 2023-07-19

categories:
  - 计算机网络
keywords: 计算机网络; computer network
description: 主要是计算机网络相关知识点
top_img:
cover: https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_05_01_15_19.gif
---

# 计算机网络主要考点



W3C(World Wide Web Consortium)：指定Web标准

RFC(Request For Comments 意见请求)：制定RFC草案、互联网通信协议标准



## OSI 和 TCP/IP

![1953408-20210620133658723-1723136444](https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_05_01_15_19.gif)



| OSI 参考模型 | 主要作用(每一层都上层提供服务且透明)                         | 常见协议        |
| ------------ | ------------------------------------------------------------ | --------------- |
| 应用层       | 准备数据，提供应用程序间通信，为应用程序提供**网络接口**     | HTTP、SSH、DHCP |
| 表示层       | 在传输之前处理数据的格式：进行数据**压缩/还原**、数据**加/解密**(二进制、编码格式ascii) |                 |
| 会话层       | 建立、维护和管理会话。即在两个互相通信的进程之间对传输的报文提供**同步管理**服务 | TLS、SSL        |
| 传输层       | 建立**端到端**的连接，提供两端应用程序的通信。可靠传输(分段编号)/不可靠传输、流量控制 | TCP、UDP        |
| 网络层       | **点对点**，寻址和路由选择(选择路由路径)                     | IP、ICMP        |
| 数据链路层   | 提供介质访问、链路管理(在网络节点间的线路上传送**数据帧**)   | ARP             |
| 物理层       | 在传输媒体上传输**比特流**、定义了网络设备的机械特性,接口标准,电器标准,过程特性 |                 |



| TCP/IP 协议栈                                                | 物理层和数据链路层的底层的协议由网卡NIC通过硬件实现<br/>网络层、传输层的协议由操作系统实现并向应用层提供socket接口 | 默认端口号 |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ---------- |
| **应用层**                                                   |                                                              |            |
| **HTTP** (HyperText Transfer Protocol) 超文本传输协议        | 基于TCP，默认使用80号端口<br />是从 WWW 服务器传输超文本到本地浏览器时所使用的传输协议，无状态的<br />无状态的意思是其数据包的发送、传输和接收都是相互独立的。<br/>无连接的意思是指通信双方都不长久的维持对方的任何信息。 | 80         |
| **SSH** (Secure Shell) 安全外壳协议                          | 基于TCP，默认使用22号端口<br />主要用于远程登录和文件传输    | 22         |
| **DNS** (Domain Name System) 域名解析                        | 基于UDP，默认使用53号端口<br/>本地服务器要配置 DNS 服务器地址，通过 DNS 服务器解析域名得到 IP 地址 | 53         |
| **DHCP** (Dynamic Host Configuration Protocol) 动态主机配置协议 | 基于UDP，默认使用67(DHCP server)，68(DHCP client)端口<br />为 DHCP 客户端配置网关地址、DNS 服务器地址、动态分配 IP 地址 | 67         |
| SMTP(Simple Mail Transfer Protocol)简单邮件传输协议          | 基于TCP，使用25号端口，是一组用于由源地址到目的地址传送邮件的规范，用来控制信件的发送、中转的方式，帮助在发送或中转信件时找到下一个目的地，SMTP服务器就是遵循SMTP协议的发送邮件的服务器 | 25         |
| FTP文件传输协议，File Transfer Protocol                      | 基于TCP，一般上传下载用FTP服务，数据端口是20号，控制端口是21号 | 20、21     |
| TELNET：远程登入协议                                         | 基于TCP，使用23号端口，是Internet远程登录服务器的标准协议和主要方式。为用户提供了在本地计算机上完成远程主机工作的能力。在终端使用者的电脑上使用telnet程序连接到服务器。使用明码传送，保密性差，不安全，但简单方便，专门为局域网设计 | 23         |
|                                                              |                                                              |            |
| **传输层**                                                   |                                                              |            |
| **TCP**：传输控制协议，Transmission Control Protocol         | **面向连接的**(三次握手四次挥手)、字节流、**可靠的** (数据按序到达、确认重传机制)、流量控制 (滑动窗口机制)、**全双工**。适用于对差错不敏感的或对性能要求高的实时的业务 |            |
| **UDP**：用户数据报协议，User Datagram Protocol              | **面向无连接**，面向消息，**不可靠的**(可靠性由上层协议保障)。适用于对性能要求高，对数据正确性要求不高的或者需要多播广播的业务。 |            |
| TLS (Transport Layer Security)                               | 是SSL(Secure Sockets Layer)的标准化版本<br />主要用于Web浏览器和服务器之间的安全通信 |            |
| SCTP：流量传输控制协议                                       | 一种面向连接的流传输协议，SCTP虽然在首发两端有多条路径，但实际只是使用一条路径传输，当该条路径出现故障时，不需要断开连接，而是转移到其他路径 |            |
| MPTCP：多路径传输控制协议                                    | TCP的多路径版本，MPTCP真正意义上实现了多路径并行传输，在链接建立阶段，建立多条路径，然后使用多条路径同时传输数据 |            |
|                                                              |                                                              |            |
| **网络层**                                                   |                                                              |            |
| **IP** (Internet Protocol)                                   | 是 TCP/IP 协议簇中最为核心的协议，所有的上层协议都以 IP 数据包的格式传输<br/>不可靠：不保证数据包成功到达目的地，对于数据传输过程的错误数据包，IP协议将会进行丢弃<br/>无连接：对数据包处理是独立进行的，也不按发送顺序接收 |            |
| **ICMP** (Internet Control Message Protocol)                 | 用于传递控制消息、Ping目标主机是否可达                       |            |
| **ARP** (Address Resolution Protocol) 地址解析协议           | 将目标设备的 IP 地址通过局域网广播，解析目标 MAC 地址        |            |
|                                                              |                                                              |            |
| **链路层**                                                   |                                                              |            |



## TCP

### 三次握手的过程 (基础)

**在TCP/IP协议中，TCP协议提供可靠的连接 (Socket)服务，采用三次握手建立一个连接。**

![image-20230501154151616](https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_05_01_15_41.png)

第一次握手
**准备建立连接。客户端发送SYN包(同步标志为1，序列号为x)，客户端进入同步发送状态，等待服务器的确认**

第二次握手
**服务端发送SYN+ACK包，服务端进入同步收到 syn-received 半连接状态**

第三次握手
**客户端发送ACK包，进入连接建立状态。收到报文服务器端进入连接建立状态，完成TCP三次握手**

seq序列号Sequence Number
SYN同步报文段，同步标志
ACK应答报文段
ack应答序列号Acknowledgment Number

> 准备建立连接，服务端开始监听。
> 第一次握手，客户端发送SYN同步包，进入 SYN-send 状态。
> 第二次握手，服务端收到SYN包，并发送SYN加ACK同步应答包，进入 SYN-received 半连接状态。
> 第三次握手，客户端收到SYN加ACK包，并发送ACK应答包，进入连接建立状态。
> 服务端收到ACK包后也进入连接建立状态，完成TCP三次握手。



#### 为什么要第三次握手

**因为存在请求包超时的情况**。为了防止已失效的超时的请求连接报文段突然又传送到了服务端，因而产生错误。

**当**客户端发出的第一个连接请求报文段并没有丢失，而是在某个网络结点长时间的滞留了，以致延误到连接释放以后的某个时间才到达服务端。本来这是一个早已失效的报文段。但服务端收到此失效的连接请求报文段后，就**误认为是客户端再次发出的一个新的连接请求**。于是就向客户端发出确认报文段，同意建立连接。
		**假设**不采用“三次握手”，那么只要server发出确认，新的连接就建立了。由于现在client并没有发出建立连接的请求，因此不会理睬server的确认，也不会向server发送数据。但server却以为新的运输连接已经建立，并一直等待client发来数据。这样，server的很多资源就白白浪费掉了。
		**假设**采用“三次握手”的办法可以防止上述现象发生。例如刚才那种情况，client不会向server的确认发出确认。server由于收不到确认，就知道client并没有要求建立连接。”三次握手就是为了把是否连接的决定权交给client，在双方都确认对方准备好了才开始

>为了避免超时的请求包被误认为是新的连接请求。



### TCP的 SYN Cookie 机制

SYN Flood 攻击会伪造 SYN 包，占用服务器资源，有63秒的半连接状态。

针对 SYN Flood 的防护措施有 SYN Cookie 机制
**TCP 的 SYN Cookie 机制**：当服务端 SYN 队列满后，通过 tcp_syncookies 参数向客户端发送 SYN Cookie。若该客户端为正常连接则会向服务器回发 SYN Cookie，**能直接建立连接**。



### TCP 保活机制

向客户端发送保活深测报文，若未收到响应则继续发送。若尝试次数达到保活探测数，但是仍未收到响应则中断该连接。



### 四次挥手的过程 (基础)

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_05_01_16_24.png" alt="image-20230501162443442" style="zoom:80%;" />

使用 Linux 命令: `netstat -n | awk '/^tcp/{++S[$NF]}END{for(a in S) print a,S[a]}' ` 查看服务器TCP连接状态数

第一次挥手
**主机1 (客户端或服务器端)，发送FIN报文，进入FINAL WAIT状态**

第二次挥手
**主机2发送ACK报文，进入CLOSE WAIT状态**

第三次挥手
**主机2发送FIN报文，进入LAST ACK状态**

第四次挥手
**主机1发送ACK报文，进入TIME WAIT状态。主机2收到应答报文进入CLOSE状态，主机1等待两倍的最长报文寿命(30s)进入CLOSE状态。**

> 第一次挥手，客户端发送FIN终止包，进入 final-wait 状态。
> 第二次挥手，服务端收到FIN包，并发送ACK应答包，进入close-wait 状态。
> 第三次挥手，服务端发送FIN终止包，进入 last-ack 状态。
> 第四次挥手，客户端收到上面ACK包和FIN包，并发送给服务端一个ACK包，进入 time-wait 状态，等待两倍的MSL最长报文寿命时间后进入 Close 状态。
> 服务端收到ACK包后也进入 Close 状态。完成TCP四次挥手。



#### 为什么要四次挥手

因为TCP是**全双工模式**的，所以客户端和服务端都需要发送FIN报文和ACK报文



#### 为什么连接三次握手，关闭却是四次挥手

当请求连接时，服务端把SYN和ACK放一个报文里发给客户端
当关闭连接时，服务端不会立即关闭SOCKET连接，会先回复ACK报文，再**由应用层决定发送FIN报文**



#### 为什么主机1要有TIME_WAIT状态

**存在应答报文丢失的情况**

**当**主机1发送的应答报文丢失，主机2会再次发送FIN片段
		**假设**不采用TIME WAIT状态，则主机2会一直等待应答报文
		**假设**主机1处于TIME WAIT状态，可以再次发送应答报文。就是延迟关闭，确保主机1和主机2都关闭了。

> 因为当客户端的ACK应答包丢失，服务端会再次发送FIN终止包。
> 两倍的MSL时间的延迟关闭，能够确保服务端已经关闭了再关闭自己。





#### 服务器出现大量 CLOSE_WAIT 状态的原因

原因：对方关闭scocket连接，我方忙于读或写，没有及时关闭连接 (没有及时发送FIN终止包)
解决：检查代码，特别是**释放资源**的代码、检查配置，特别是处理请求的线程配置



### TCP和UDP的区别

连接：TCP面向连接、UDP无连接适合消息**多播**
**可靠性：TCP有握手机制、重传机制**
有序性：TCP有序列号
速度：UDP适合媒体、多人在线游戏
量级：TCP20字节、UDP8字节



### TCP流量控制 滑动窗口机制

TCP流量控制：是指通过滑动窗口机制来控制发送方的数据发送速率，从而确保接收方能够及时处理这些数据。

> 滑动窗口机制
>
> TCP滑动窗口机制是建立在确认重传机制上的
> 接收方接受窗口的大小等于最大接收缓存减发送方已发送但自己未接受的大小
> 当接收方收到数据后，会向发送方发送一个ACK确认包并告诉发送方自己当前的接收窗口大小。
> 发送方的发送窗口大小就等于接收窗口大小减去已发送但未确认的大小。

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_05_02_20_18.png" alt="image-20230502201834857" style="zoom:80%;" />

发送窗口只有收到接收方的ACK确认(累计确认)才会移动发送窗口的左边界

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_05_02_20_19.png" alt="image-20230502201853949" style="zoom:80%;" />

接收窗口只有在前面所有的段都确认的情况下才移动左边界

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_05_02_20_22.png" alt="image-20230502202232177" style="zoom:40%;" />

rwnd 接收窗口大小 = 最大的接收缓存 - (发送方已发送且未接受)
swnd 发送窗口大小 = 接收窗口大小 - (发送未确认)



### TCP拥塞控制 拥塞窗口

TCP拥塞控制：是指通过不同的机制来控制发送方的数据发送速率 (**swnd = min(rwnd, cwnd)**)，从而**避免超时重传**。

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_07_19_00_26.png" alt="image-20230719002559415" style="zoom:80%;" />

**慢启动策略**：TCP连接刚开始时 (cwnd 拥塞窗口 < ssthresh slow start threshold) ，cwnd 会以指数级增加，直到网络出现拥塞为止。

**拥塞避免策略**：在慢启动结束后 (cwnd > ssthresh) 使用，cwnd 会以线性增加的方式调整发送速率，以避免网络拥塞。

当发送超时重传，ssthresh = cwnd/2，cwnd = 1

**快速重传策略**：发送方收到三个连续的重复 ACK 确认，就立即重传丢失的数据包，而不是超时重传。

![image-20230719002548631](https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_07_19_00_25.png)

**快速恢复策略**：发送方收到三个连续的重复 ACK 确认，ssthresh = cwnd，cwnd = cwnd/2，跳过慢开始。



## 应用层

### https

```url
https://<username>:<password>@<hostname>:<port>/<path>

https://riot:WHqgo66Fw1eF9en3iUP3Kg@127.0.0.1:59190
https://127.0.0.1:59190/lol-patch/v1/game-version
```



### SSH

SSH：建立连接，交换密钥，使用非对称加密发送消息



建立连接
TCP握手、SSH握手(协议版本、密钥交换初始化、ECDH密钥交换初始化)

密钥交换过程(DH算法)：(加密==签名)
确保中间人拿不到：sender用私钥加密发送，receiver也用私钥加密发送，sender解密发送，receiver解密拿到数据
确保中间人没篡改哈希值：sender哈希发送，receiver需要确保计算哈希值和接受哈希值一致

对称加密：sender、receiver使用同一个密钥
非对称加密：sender有公钥和私钥，用私钥加密信息发送；receiver只有公钥，用公钥解密



过程：客户端临时私钥
客户端临时公钥
获得服务端临时公钥，后生成和服务端一样的共享安全密钥和交换哈希值
获得服务端交换哈希值[服务端host临时私钥签名]，并用服务端host临时公钥解密得到交换哈希值，与计算出来的交换哈希值比较

服务端临时私钥
**服务端临时公钥**
服务端host临时私钥
**服务端host临时公钥**
共享安全密钥 = 服务端临时私钥 + 服务端临时公钥 + 客户端临时公钥
交换哈希值 = 共享安全密钥 + ...
**交换哈希值[服务端host临时私钥签名]**



其实第一次登录收到的服务端host临时公钥还是不安全的
解决一：可以Client用ssh-keygen生成密钥对、ssh-copy-id将Client公钥放服务器，之后不用密码登录
解决二：将收到的服务器host临时公钥，与服务器公开的临时公钥对比 (github就是这样)

```bash
ssh-keygen -t <ed25519|dsa|ecdsa|rsa> -b 4096 -C "xxyy"  # 在Client端执行，在~/.ssh目录下生成密钥对id_<ed25519|dsa|ecdsa|rsa>私钥和 id_<ed25519|dsa|ecdsa|rsa>.pub公钥
# 将公钥复制到Server端 `~/.ssh/authorized_keys`，之后不用密码登录
```

```bash
ssh-keygen -A  # 在Server端执行，在/etc/ssh目录下生成 ssh_host_<ed25519|dsa|ecdsa|rsa>_key私钥 和 .pub公钥  
# 用于启动sshd.service服务，所需的生成服务端host临时公钥  
# 用于Client首次连接到Server, Client将Server公钥保存到~/.ssh/known_hosts
```



```bash
ssh -p 22 root@192.168.100.101 
```



## 网络层

### IP地址

IP 地址具有层次特点，分为网络号、主机号
子网掩码的作用就是划分网络号和主机号



#### 层次划分

|                 |                                        | 私有IP地址                                                   |      |
| --------------- | -------------------------------------- | ------------------------------------------------------------ | ---- |
| A类地址         | 0.0.0.0 到 127.255.255.255，0开头      | 10.0.0.0/8 (10.0.0.0 到 10.255.255.255)<br />本地回环地址127.0.0.1 到 127.255.255.254 |      |
| B类地址         | 128.0.0.0 到 191.255.255.255，10开头   | 172.16.0.0/12 (172.16.0.0 到 172.31.255.255)                 |      |
| C类地址         | 192.0.0.0 到 223.255.255.255，110开头  | 192.168.0.0/16 (192.168.0.0 到 192.168.255.255)              |      |
| D类地址组播地址 | 224.0.0.0 到 239.255.255.255，1110开头 |                                                              |      |
| E类地址保留地址 | 240.0.0.0 到 255.255.255.255，1111开头 |                                                              |      |
|                 |                                        |                                                              |      |
|                 |                                        |                                                              |      |



#### NAT

NAT(Network Address Translation)，私有 IP 地址转换为公共 IP 地址，解决 IPv4 网络地址不足的问题



#### CIDR

CIDR格式通常为 IP地址/子网掩码位数，用于实现IP地址的聚合和划分



#### URI与URL的区别

Uniform Resource Identifier 统一资源标识符：更广泛(标识网络资源+本地资源)

Uniform Resource Locator 统一资源定位符：仅网络资源(含协议头)



#### localhost与127.0.0.1的区别

localhost 不经网卡传输，不受防火墙和网卡的限制 (更推荐使用)

127.0.0.0/8 表示loop back设备



### ICMP

Internet Control Message Protocol 用于传递控制消息，是一些网络链路本身的信息
ICMP的请求包(TYPE:0x08，CODE:0X00)和回复包(TYPE:0x00，CODE:0X00)，使用包中TYPE字段进行区分
**ICMP是IP的一部分，可以进行retry重试**

应用：
ping 主要获得到目标主机链路的时延信息
时延 = 发送时延 + 传播时延 + 处理时延 + 排队时延

区别：
ARP是在局域网内广播的，ICMP不受网络范围限制，也可以判断外网链路情况



## 数据链路层

数据链路层将网络层的数据包封装成数据帧再在网络中进行传输。网卡接收数据帧，根据帧中目的MAC地址判断是否丢弃，所以在发送数据包之前必须知道目的主机的MAC地址

### ARP

Address Resolution Protocol 用于根据IP地址查MAC地址

过程：先查ARP缓冲表，查不到则在局域网中发送ARP广播

应用：
Python中 scapy 模块：就实现ARP协议，可以知道可以通达的目标主机的MAC地址，可以知道网络有哪些存活的主机
使用户能够嗅探、剖析、伪造、发送网络数据包，可以构建扫描、探测、攻击网络的工具



## Socket 套接字

**Socket 是对 TCP/IP 协议的抽象**，是操作系统对外开放的**接口**，可以视作一个**唯一标识**即 Socket = IP + 协议 + 协议端口号。

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_05_06_14_40.png" alt="image-20230506144053203" style="zoom:75%;" />

Socket通信流程

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_05_06_14_41.png" alt="image-20230506144116472" style="zoom:90%;" />

### Java Socket 编程题

编写一个网络应用程序，有客户端与服务器端，**客户端**向服务器**发送一个字符串**，**服务器**收到该字符串后将其打印到命令行上，然后向客户端**返回该字符串的长度**，最后，客户端输出服务器端返回的该字符串的长度，分别用**TCP**和**UDP**两种方式去实现。

#### TCP

```java
public class TCPServer {
    public static void main(String[] args) throws Exception {
        //创建socket,并将socket绑定到65000端口
        ServerSocket ss = new ServerSocket(65000);
        //死循环，使得socket一直等待并处理客户端发送过来的请求
        while (true) {
            //监听65000端口，直到客户端返回连接信息后才返回
            Socket socket = ss.accept();
            //获取客户端的请求信息后，执行相关业务逻辑
            new LengthCalculator(socket).start();
        }
    }
}
```

```java
public class LengthCalculator extends Thread {
    //以socket为成员变量
    private Socket socket;

    public LengthCalculator(Socket socket) {
        this.socket = socket;
    }

    @Override
    public void run() {
        try {
            //获取socket的输出流
            OutputStream os = socket.getOutputStream();
            //获取socket的输入流
            InputStream is = socket.getInputStream();

            //buff主要用来读取输入的内容，存成byte数组，ch主要用来获取读取数组的长度
            int ch = 0;
            byte[] buff = new byte[1024];
            ch = is.read(buff);

            //将接收流的byte数组转换成字符串，这里获取的内容是客户端发送过来的字符串参数
            String content = new String(buff, 0, ch);
            System.out.println(content);
            //往输出流里写入获得的字符串的长度，回发给客户端
            os.write(String.valueOf(content.length()).getBytes());

            //不要忘记关闭输入输出流以及socket
            is.close();
            os.close();
            socket.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

```java
public class TCPClient {
    public static void main(String[] args) throws Exception {
        //创建socket，并指定连接的是本机的端口号为65000的服务器socket
        Socket socket = new Socket("127.0.0.1", 65000);
        //获取输出流
        OutputStream os = socket.getOutputStream();
        //获取输入流
        InputStream is = socket.getInputStream();
        //将要传递给server的字符串参数转换成byte数组，并将数组写入到输出流中
        os.write(new String("hello world").getBytes());

        int ch = 0;
        byte[] buff = new byte[1024];
        //buff主要用来读取输入的内容，存成byte数组，ch主要用来获取读取数组的长度
        ch = is.read(buff);
        //将接收流的byte数组转换成字符串，这里是从服务端回发回来的字符串参数的长度
        String content = new String(buff, 0, ch);
        System.out.println(content);
        //不要忘记关闭输入输出流以及socket
        is.close();
        os.close();
        socket.close();
    }
}
```

#### UDP

```java
public class UDPServer {
    public static void main(String[] args) throws Exception {
        // 服务端接受客户端发送的数据报
        DatagramSocket socket = new DatagramSocket(65001); //监听的端口号

        // 可以像TCPServer中一样创建线程处理，但是这里为了简便没创建线程
        byte[] buff = new byte[100]; //存储从客户端接受到的内容
        DatagramPacket packet = new DatagramPacket(buff, buff.length);
        //接受客户端发送过来的内容，并将内容封装进DatagramPacket对象中
        socket.receive(packet);
        byte[] data = packet.getData(); //从DatagramPacket对象中获取到真正存储的数据
        //将数据从二进制转换成字符串形式
        String content = new String(data, 0, packet.getLength());
        System.out.println(content);
        //将要发送给客户端的数据转换成二进制
        byte[] sendedContent = String.valueOf(content.length()).getBytes();
        // 服务端给客户端发送数据报
        //从DatagramPacket对象中获取到数据的来源地址与端口号
        DatagramPacket packetToClient = new DatagramPacket(sendedContent,
                sendedContent.length, packet.getAddress(), packet.getPort());
        socket.send(packetToClient); //发送数据给客户端
    }
}
```

```java
public class UDPClient {
    public static void main(String[] args) throws Exception {
        // 客户端发数据报给服务端
        DatagramSocket socket = new DatagramSocket();
        // 要发送给服务端的数据
        byte[] buf = "Hello World".getBytes();
        // 将IP地址封装成InetAddress对象
        InetAddress address = InetAddress.getByName("127.0.0.1");
        // 将要发送给服务端的数据封装成DatagramPacket对象 需要填写上ip地址与端口号
        DatagramPacket packet = new DatagramPacket(buf, buf.length, address,
                65001);
        // 发送数据给服务端
        socket.send(packet);

        // 客户端接受服务端发送过来的数据报
        byte[] data = new byte[100];
        // 创建DatagramPacket对象用来存储服务端发送过来的数据
        DatagramPacket receivedPacket = new DatagramPacket(data, data.length);
        // 将接受到的数据存储到DatagramPacket对象中
        socket.receive(receivedPacket);
        // 将服务器端发送过来的数据取出来并打印到控制台
        String content = new String(receivedPacket.getData(), 0,
                receivedPacket.getLength());
        System.out.println(content);
    }
}
```



## 局域网

Internet 因特网

Ethernet 以太网是一种局域网中传输数据的技术，介质访问控制方法使用CSMA/CD(载波侦听多路访问/碰撞检测)协议(避免多个节点同时发送数据导致的碰撞，从而保证数据的可靠传输)



## 硬件

### 双绞线

CAT：5类线100Mbps、5e超五类1000Mbps、6类线1Gbps

568B：白橙橙白绿蓝

 awg 直径：24粗、26细，24粗好



### 路由器与交换机的区别

| 二层交换机                                                   | 路由器                   |
| ------------------------------------------------------------ | ------------------------ |
| 通过传输媒介和接口连接设备                                   | 通过路由表来连接设备     |
| 设备要在同一个子网内(三层交换机可以跨网段，三层指网络层具有路由功能) | 设备可以不再同一个子网内 |
| 利用网线连接不同网络设备，起到设备连接功能，决定网线使用哪个接口转发(数据交换) | 路由转发(跨网段通信)     |



# 我的

路由器固件：H大的padavan老毛子(192.168.123.1)+breed控制台、openwrt、[istoreos](https://github.com/istoreos/istoreos)(推荐)



## 请求重定向和请求转发的区别

|                        | 请求重定向Redirect                                       | 请求转发Forward                                             |
| ---------------------- | -------------------------------------------------------- | ----------------------------------------------------------- |
| URL 地址(表象)         | 变                                                       | 不变                                                        |
| **URL 地址变化(本质)** | 服务器发送302给浏览器，告诉浏览器重新向新URL请求         | 服务器内部操作                                              |
| 资源消耗(影响)         | 多                                                       | 少                                                          |
| 数据传递(影响)         | 无法直接传递请求参数,必须通过 URL 附加参数的方式进行传递 | 可以直接在服务器端共享请求对象,因此可以很方便地传递请求参数 |

