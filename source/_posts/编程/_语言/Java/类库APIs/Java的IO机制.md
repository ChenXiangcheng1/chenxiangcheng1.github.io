---
title: JavaIO机制
date: 2023-07-13

tags:
categories:
  - Java
  - 类库
keywords: Java,BIO,NIO,AIO
description: 关于BIO机制(阻塞)、NIO机制(非阻塞)、AIO机制(异步)的核心接口与类。
top_img:
cover: https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_07_12_01_09.png" alt="image-20230712010935842
---


# Java的IO机制主要考点



## IO过程

服务端：Accept 接收客户端请求、Read 准备数据、Write 拷贝数据



## 服务端BIO、NIO、AIO模型的区别(总结)

| 属性模型              | 阻塞BIO                                                 | 非阻塞NIO                                                    | 异步AIO                                                      |
| --------------------- | ------------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 客户端之间同步情况    | 客户端阻塞整个步骤                                      | NIO不是严格异步的，NIO的 I/O 操作是非阻塞的，应用程序可以在等待 I/O 完成时继续执行其他任务，但是NIO依赖阻塞或轮询机制来检查就绪状态并处理 I/O 操作 | 允许应用程序在发起I/O请求后，继续执行其他任务，而不需要等待I/O操作完成。 |
| 线程数(client:server) | 1:1<br />每来一个客户端线程，服务端也需要开一个额外线程 | N:1<br />服务端一个额外**NIO线程**(select())响应多个客户端请求 | N:0 <br />服务端直接主线程(异步系统调用epoll边缘触发)，不需要额外线程 |
| 复杂度                | 简单                                                    | 较复杂                                                       | 复杂                                                         |
| 吞吐量                | 低                                                      | 高                                                           | 高                                                           |
| 应用场景              | 适用于连接数少且需要低延迟的场景，如数据库连接          | 适用于连接数目多且短连接的场景，如聊天服务器<br />可以构建多路复用的的 IO 操作 | 适用于连接数目多且长连接的场景，如相册服务器                 |
| 原理                  |                                                         | 通过Java NIO库(底层使用了OS的IO多路复用系统调用)中的**Selector选择器**实现**事件驱动**的异步非阻塞I/O操作，再应用程序通过**获取就绪事件轮询**来处理结果。 | 利用了**epoll系统调用**监听多个FD，当事件产生时操作系统**回调**FD绑定的事件处理器，通知应用程序处理结果。实现了**事件驱动**的异步非阻塞I/O操作 |

> 同步异步 不等于 阻塞，两个概念相关但不同
>
> 阻塞针对的不是当前过程，而是使用同一资源的多个过程之间的协作方式。通常都是 IO 比计算慢，故常见的阻塞是针对 IO 的阻塞。一个 IO 通道，如果只允许同时最多一个过程使用它，在该过程使用它期间其他过程只能等，那么这就是阻塞的。如果**允许同时多个过程使用它**，那么这是非阻塞的。
>
> 同步异步，针对的是调用者和被调用者之间的协作过程。如果被调用者不做完就不回复，同时调用者还一直等着它，那么这是同步的。如果被调用者没处理完就提前做个特殊回复，同时调用者认可这种特殊回复，那么这是异步调用。如果调用者这收到异步特殊回复之后，又再继续等待这个特殊回复转换成完整回复，那么这是异步之后的回转同步，或者，整体上来说，这还是同步调用。

## BIO

Block-IO，基于流模型实现

包：
java.net：URLConnection、Http
java.io

优点：代码简单直观
缺点：IO效率，扩展差



<center><img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_05_10_12_38.png" alt="image-20230510123856719" style="zoom:80%;" /><img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_07_11_18_12.png" alt="image-20230711181136117" style="zoom:95%;" /></center>

<center><img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting2/img/2023_09_25_01_20.webp" alt="img" style="zoom:70%;" /><img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting2/img/2023_09_25_01_21.webp" alt="img" style="zoom:95%;" /></center>



### ServerSocket 服务器套接字

此类实现服务器套接字。服务器套接字等待通过网络传入的请求。它根据该请求执行某些操作，然后可能将结果返回给请求者。

**方法：**

`ServerSocket(int)` 构造函数，将 ServerSocket 绑定到指定端口

`accept(): Socket` 当监听端口接收到新的客户端连接，则创建一个新的 Socket 并返回。若没有接收到，则阻塞直到建立连接为止。

`bind(SocketAddress): void` 将ServerSocket绑定到特定地址（IP 地址和端口号）



### Socket 客户端套接字

此类实现客户端套接字（也称为“套接字”）。套接字是两台机器之间通信的端点。



**方法：**

`getInputStream(): InputStream`  返回此套接字的输入流。

`getOutputStream(): OutputStream`  返回此套接字的输出流。



### InetSocketAddress

此类是对 SocketAddress 的扩展，实现了 IP 套接字地址（IP 地址 + 端口号）。

**方法：**

`InetSocketAddress(int)` 构造方法，int为端口号



### IO流

![v2-eb408ac849a679b09941be7ebd734768_r](https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_05_14_23_39.jpg)

### 1Reader 读字符流 

Reader，从输入源中读取字符流数据，处理文本数据



#### InputStreamReader 字符输入流

**方法：**

`InputStreamReader(InputStream)` 构造函数



#### BufferedReader 缓冲字符输入流 (最佳实践)

**方法：**

`BufferedReader(Reader)` 构造函数

`readLine(): String` 读取一行文本



### 2Writer 写字符流 

Writer，向输出目标中写入字符流数据，处理文本数据



#### 方法

`flush():void` 刷新流



#### PrintWriter 字符输出流

**方法：**

`PrintWriter(OutputStream, boolean)` 构造函数

`print(String): void` 打印字符串。如果参数为null，则打印字符串"null" 。



### 3InputStream 输入字节流 

InputStream，从输入源读取字节流数据，处理二进制数据



### 4OutputStream 输出字节流 

OutputStream，向输出目标写入字节流数据，处理二进制数据



### 实现BIOEcho服务器(重点)

Echo：从客户端读取数据并原封不动写回去

```java
public class BIOPlainEchoServer {
    public void improvedServe(int port) throws IOException {
        // 1. 创建ServerSocket并绑定到指定的端口
        final ServerSocket socket = new ServerSocket(port);
        // 创建一个线程池
        ExecutorService executorService = Executors.newFixedThreadPool(6);
        while (true) {
            // 2. 使用accept()阻塞直到收到新的客户端连接
            final Socket clientSocket = socket.accept();
            System.out.println("Accepted connection from " + clientSocket);
            // 3. 将请求提交给线程池去执行
            executorService.execute(() -> {
                try (BufferedReader reader = new BufferedReader(new InputStreamReader(clientSocket.getInputStream()))) {
                    PrintWriter writer = new PrintWriter(clientSocket.getOutputStream(), true);
                    // 从客户端读取数据并原封不动写回去
                    while (true) {
                        writer.println(reader.readLine());
                        writer.flush();
                    }
                } catch (IOException e) {
                    e.printStackTrace();
}	}	}	});	}

```



#### 过程

服务端 BIO 过程：

1. **创建一个 ServerSocket，监听并绑定一个端口**
2. 客户端来请求这个端口
3. **服务器使用 Accept()**，获得一个来自客户端的 Socket 连接对象
4. **一个请求启动一个新线程处理连接**
   1. 读 Socket，得到字节流

   2. 解码协议，得到 Http 请求对象
   3. 处理 Http 请求，得到一个结果，封装成一个 HttpResponse 对象
   4. 编码协议，将结果序列化字节流
   5. 写 Socket，将字节流发给客户端
5. 继续循环步骤3

服务端 BIO 阻塞情况：

* Accept 是阻塞的，只有新连接来了，Accept才会返回，主线程才能继，

* Read 是阻塞的，只有请求消息来了，Read才能返回，子线程才能继续处理

* Write 是阻塞的，只有客户端把消息收了，Write才能返回，子线程才能继续读取下一个请求



## NIO

**NIO是通过选择器和通道实现非阻塞I/O操作，使用事件驱动的方式进行处理。**

NonBlock-IO，引入了**多路复用机制**

**客户端的请求 channel 会注册到多路复用选择器 Selector 上产生 SelectedKey，selector 可以监控许多的IO请求，再调用 Selector 中 ServerSocketChannel 的 select() 方法，会阻塞直到有事件 SelectionKey.OP_XXX 就绪，即当有一个 Socket 的数据准备好时。**todo: 这一段 NIO的工作机制还没理清

优先：可以构建多路复用的、异步非阻塞的 IO 操作，提供了更接近操作系统底层的高性能数据操作方式——零拷贝操作[^零拷贝]
特点：程序需要不断的询问内核数据准备好

**采用 Selector，服务器仅仅 `selector.select();` 阶段会阻塞，避免大量客户端连接时频繁切换线程**

<center><img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_07_11_23_09.png" alt="image-20230711230907621" style="zoom: 50%;" /><img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_07_11_23_10.png" style="zoom: 60%;" /></center>

<center><img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting2/img/2023_09_25_01_30.webp" alt="123" style="zoom:50%;" /><img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_07_11_21_18.png" alt="image-20230711211827871" style="zoom: 45%;" /><img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting2/img/2023_09_25_01_34.webp" alt="img" style="zoom:70%;" /></center>



### 事件驱动

事件驱动的目标是将程序的控制权交给事件循环

* 事件源，是事件的产生者，会在特定的条件满足时触发相应的事件。
* 事件监听器，注册回调函数，当事件源触发事件时，事件监听器会被调用来执行相应的操作
* 事件循环，不断地轮询事件源，将事件传递给正确的事件监听器进行处理
* 回调函数，会被注册到响应的事件监听器，在特定事件发生时被调用，回调函数通常是异步执行的避免阻塞主线程



### 零拷贝

Java 的内存分为堆内存+栈内存+字符串常量池等等，其中 GC 堆是占用内存空间最大的一块，也是 Java 对象存放的地方，一般我们的数据如果需要从 IO 读取到堆内存，中间需要经过 Socket 缓冲区，也就是说一个数据会被拷贝两次才能到达他的的终点，如果数据量大，就会造成不必要的资源浪费。

NIO 使用零拷贝，当他需要接收数据的时候，他会在堆内存之外开辟一块内存，数据就直接从 IO 读到了那块内存中去。



### Channels

Channel 和 Stream 都是基于 IO 操作的抽象。

Channel是双向的，使用基于块的IO操作，适合处理大量数据和高并发场景。
Stream是单向的

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_07_11_18_11.png" alt="image-20230711181104925" style="zoom:67%;" />

**AbstractInterruptibleChannel-SelectableChannel的方法**

`close(): void` 关闭该通道



#### FileChannel

文件IO

**方法：**

`transferFrom(ReadableByteChannel, long, long): long` 把另外一个 ReadableByteChannel 中的数据拷贝到 FileChannel

`transferTo(long, long, WritableByteChannel): long` 把 FileChannel 中的数据拷贝到另外一个 WritableByteChannel

该接口常用于需要高效的网络文件的数据传输和大文件拷贝，不需要将原数据从内核态拷贝到用户态，再从用户态拷贝到目标通道的内核态，避免了两次用户态和内核态间的上下文切换，即**”零拷贝”**，效率较高于 BIO 中提供的方法 (零拷贝的底层涉及OS底层运行机制)

[^零拷贝]: 零拷贝



#### DatagramChannel

网络IO UDP



#### SocketChannel

网络IO TCP客户端

**方法：**

`configureBlocking(boolean): SelectableChannel ` 调整该通道的阻塞模式，true为阻塞状态，false为非阻塞状态。

`register(Selector, int, Object): SelectionKey` 向给定的 Selector 注册此 Channel 会产生一个 SelectionKey 并返回。
int 是枚举类型，表示 Selector 的兴趣 (关注的点)。
Object 传入一个对象实例作为缓存。例如，`ByteChannel` 可以传入`ByteBuffer.allocate(100)`
Channel 在阻塞模式下向 Selector 注册会抛出异常，只有非阻塞模式才能注册

| 枚举类型(若关注多个事件可通过 \| 连接) | 释义                                 |
| -------------------------------------- | ------------------------------------ |
| SelectionKey.OP_ACCEPT                 | SocketChannel 已准备好接受另一个连接 |
| SelectionKey.OP_CONNECT                | 建立连接                             |
| SelectionKey.OP_READ                   | 读                                   |
| SelectionKey.OP_WRITE                  | 写                                   |



`read(ByteBuffer): int` 从 channel 里读取数据存入到 ByteBuffer 里面

`write(ByteBuffer): int` 将 ByteBuffer 里的数据写入到channel里



#### ServerSocketChannel

网络IO TCP服务端

**方法：**

`open(): ServerSocketChannel` 静态方法，打开新的一个 Socket 的 Channel 并返回，该通道的 Socket 未绑定

`socket(): ServerSocket ` 返回与该 Channel 关联的 ServerSocket

`configureBlocking(boolean): SelectableChannel ` 调整该通道的阻塞模式，true为阻塞状态，false为非阻塞状态。

`register(Selector, int): SelectionKey` 向给定的 Selector 注册此 Channel 会产生一个 SelectionKey 并返回。
int 是枚举类型，表示 Selector 的兴趣 (关注的点)。
Channel 在阻塞模式下向 Selector 注册会抛出异常，只有非阻塞模式才能注册

| 枚举类型(若关注多个事件可通过 \| 连接) | 释义                                 |
| -------------------------------------- | ------------------------------------ |
| SelectionKey.OP_ACCEPT                 | SocketChannel 已准备好接受另一个连接 |
| SelectionKey.OP_CONNECT                | 建立连接                             |
| SelectionKey.OP_READ                   | 读                                   |
| SelectionKey.OP_WRITE                  | 写                                   |



`accept(): SocketChannel` 接受与该通道套接字的连接，返回客户端 SocketChannel。



### Buffers

以下 Buffer 覆盖了我们能通过IO发送的基本数据类型

#### ByteBuffer

`allocate(int): ByteBuffer` 返回一个分配的新的字节缓冲区，int是新缓冲区的容量单位字节

`flip(): ByteBuffer` 翻转该缓冲区。用于 flip() 后才能读取 ByteBuffer 缓冲区

`compact(): ByteBuffer ` 将未读取的数据移动到缓冲区的起始位置，但不会清空已读取的数据。用于实现缓存的部分读取和写入



**CharBuffer**

**DoubleBuffer**

**FloatBuffer**

**IntBuffer**

**LongBuffer**

**ShortBuffer**

**MappedByteBuffer**
用于表示内存映射文件



### Selectors

优点：使单线程可以处理多个网络 IO



**源码：**

Selectors 通过 SelectorProvider 创建

```java
@SuppressWarnings("removal")
static SelectorProvider provider() {
    PrivilegedAction<SelectorProvider> pa = () -> {
        SelectorProvider sp;
        if ((sp = loadProviderFromProperty()) != null)
            return sp;
        if ((sp = loadProviderAsService()) != null)
            return sp;
        return sun.nio.ch.DefaultSelectorProvider.get();
    };
    return AccessController.doPrivileged(pa);
}
```

[DefaultSelectorProvider.java 源码](https://hg.openjdk.org/jdk8u/jdk8u/jdk/file/7fcf35286d52/src/solaris/classes/sun/nio/ch/DefaultSelectorProvider.java)

```java
/**
 * Returns the default SelectorProvider.
 */
public static SelectorProvider create() {
    String osname = AccessController
        .doPrivileged(new GetPropertyAction("os.name"));
    if (osname.equals("SunOS"))
        return createProvider("sun.nio.ch.DevPollSelectorProvider");  // devpoll
    if (osname.equals("Linux"))
        return createProvider("sun.nio.ch.EPollSelectorProvider");  // epoll
    return new sun.nio.ch.PollSelectorProvider();  // poll
}
```

NIO底层使用了操作系统中的 IO 多路复用的系统调用：select、poll、epoll 



#### 底层操作系统的I/O多路复用机制 select、poll、epoll 的区别

第一个维度：支持一个进程所能打开的最大连接数

第二个维度：**FD(文件句柄、文件描述符)** 剧增后带来的 IO 效率问题

第三个维度：消息传递方式

|        | 一个进程所能打开的最大连接数                                 | 遍历连接的速度                                               | 消息传递方式                                     |
| ------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------ |
| select | 单个进程所能打开的最大连接数由 FD_SETSIZE 宏定义，其大小是32个整数的大小（在32位的机器上，大小是32\*32，64位机器上FD SETSIZE为32\*64)，我们可以对其进行修改，然后重新编译内核，但是性能无法保证，需要做进一步测试 。基于数组存储，连接数有限。**有限 小** | 因为每次调用时都会对连接进行线性遍历，所以随着FD的增加会造成遍历速度的"线性下降"的性能问题 | 内核需要将消息传递到用户空间，需要内核的拷贝动作 |
| poll   | 本质上与select没有区别，但是它没有最大连接数的限制，原因是它是基于链表来存储的。**无限** | 同上                                                         | 同上                                             |
| epoll  | 虽然连接数有上限，但是很大，1G 内存的机器上可以打开 10 万左右的连接。**有限 大** | 由于 epoll 是根据每个 fd 上的 callback 函数来实现的，只有活跃的 socket 才会主动调用 callback 所以在活跃 socket 较少的情况下，使用 epoll 不会有"线性下降"的性能问题，但是所有 socket 都很活跃的情况下，可能会有性能问题 | 通过内核和用户空间共享一块内存来实现，性能较高   |



#### Selector 多路选择器

选择器的实现

**方法：**

`open(): Selector` 静态方法，打开一个新的多路复用选择器 Selector 并返回

`select(): int` 选择一组已经就绪的 select-key，返回已经就绪的 select-key 的数量。若没有就绪的 Channel 与该 Selector 建立连接则一直阻塞。

`selectedKeys(): Set<SelectionKey>` 返回此 Selector 选定的所有 selected-key 实例集合。



#### SelectionKey

每一个 Channel 注册到一个 Selector 就会产生一个 SelectionKey

Acceptable状态，表示 Channel 准备好建立连接
Readable状态，表示 Channel 有数据，可读出数据
Writable状态，表示 Channel 空，可写入数据

**方法：**

`channel(): SelectableChannel` 返回创建 SelectKey 的Channel

`attachment(): Object` 检索当前附件(缓存 XXBuffer，在channel.register() 时传入)并返回

`cancel(): void` 请求取消该键的 Channel 与其 Selector 的注册。

`isAcceptable(): boolean` 返回该 SelectionKey 是否是准备好建立 Socket 连接的状态。

`isReadable()` 返回该 SelectionKey 是否是准备好读取的状态

`isWritable()` 返回该 SelectionKey 是否是准备好写入的状态



### 实现NIOEcho服务器(重点)

Echo：从客户端读取数据并原封不动写回去

```java
public class NIOPlainEchoServer {
    public void serve(int port) throws IOException {
        System.out.println("Listening for connections on port " + port);
        // 1. 打开ServerSocketChannel，用于监听客户端的连接，它是所有客户端连接的父管道
        ServerSocketChannel serverChannel = ServerSocketChannel.open();
        ServerSocket ss = serverChannel.socket();
        InetSocketAddress address = new InetSocketAddress(port);
        // 将ServerSocket绑定到指定的端口里
        ss.bind(address);
        serverChannel.configureBlocking(false);
        // 2. 打开Selector来处理Channel，即创建epoll.
        Selector selector = Selector.open();
        // 3. 将ServerChannel注册到Selector里，并说明让Selector关注的点，这里是关注建立连接这个事件
        serverChannel.register(selector, SelectionKey.OP_ACCEPT);
        while (true) {
            try {
                // 4. 阻塞等待就绪的Channel，即没有与客户端建立连接前就一直轮询
                selector.select();
            } catch (IOException ex) {
                ex.printStackTrace();
                // 代码省略的部分是结合业务，正确处理异常的逻辑
                break;
            }
            // 5. 获取到Selector里所有就绪的SelectedKey实例，每将一个Channel注册到Selector就会产生一个SelectedKey
            Set<SelectionKey> readyKeys = selector.selectedKeys();
            Iterator<SelectionKey> iterator = readyKeys.iterator();
            while (iterator.hasNext()) {
                SelectionKey key = (SelectionKey) iterator.next();
                // 6. 将就绪的SelectedKey从Selector中移除，因为马上就要处理它，防止重复处理
                iterator.remove();
                try {
                    // 7_1 若SelectedKey处于Acceptable状态
                    if (key.isAcceptable()) {
                        ServerSocketChannel server = (ServerSocketChannel) key.channel();
                        // 接受客户端的连接
                        SocketChannel client = server.accept();
                        System.out.println("Accepted connection from " + client);
                        client.configureBlocking(false);
                        // 7_1 向selector注册SocketChannel，主要关注读写，并传入一个ByteBuffer实例供读写缓存
                        client.register(selector, SelectionKey.OP_WRITE | SelectionKey.OP_READ,
                                ByteBuffer.allocate(100));
                    }
                    // 7_2 若SelectedKey处于可读状态
                    if (key.isReadable()) {
                        SocketChannel client = (SocketChannel) key.channel();
                        ByteBuffer output = (ByteBuffer) key.attachment();
                        // 7_2 从clientChannel里读取数据存入到ByteBuffer里面
                        client.read(output);
                    }
                    // 7_3 若SelectedKey处于可写状态
                    if (key.isWritable()) {
                        SocketChannel client = (SocketChannel) key.channel();
                        ByteBuffer output = (ByteBuffer) key.attachment();
                        output.flip();
                        // 7_3 将ByteBuffer里的数据写入到clientChannel里。
                        client.write(output);
                        output.compact();
                    }
                } catch (IOException ex) {
                    key.cancel();
                    try {
                        key.channel().close();
                    } catch (IOException cex) {
                        // ignore on close
}	}	}	}	}	}
```



#### 过程

NIO 中，当一个 Socket 建立好之后，Thread 并不会阻塞去接受(Accept)这个 Socket，而是将这个请求交给 Selector，Selector 会不断的去遍历所有的 Socket，一旦有一个 Socket 建立完成，他会通知 Thread，然后 Thread 处理完数据再返回给客户端——这个过程是不阻塞的，这样就能让一个Thread处理更多的请求了。单线程能处理更多的连接。



## AIO

**AIO是通过操作系统提供的异步I/O机制实现非阻塞I/O操作，使用事件驱动的方式进行处理**
事件驱动：通过不断地检查事件队列中是否有新的事件到达(监听事件的发生)，并根据事件的类型触发相应的操作

Asynchronous IO，基于事件和回调机制

用户现场发送IO请求，当后台处理完成，操作系统再通知用户线程进行后续工作

read，write 都是异步方法，异步的完成后会主动调用回调函数
当有流可读取时，操作系统会将可读的流传送到 read 方法的缓冲区，并通知应用程序
写操作write写入完毕时，操作系统主动通知应用程序

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_07_12_01_09.png" alt="image-20230712010935842" style="zoom:80%;" />

**AIO原理：**

OS内核维护全局的**打开文件表**

进程维护**FD**(File Descriptor即文件描述符)
		FD是一个非负整数索引值。是打开文件的元数据(文件属性状态)到文件本身的映射。

**IO多路复用函数**：使用linux的**epoll**、select系统调用，监听多个FD的可读可写情况，实现异步处理多个连接请求。
select，水平触发(处于某状态时通知)，轮询方式O(n)
epoll，支持边缘触发(变化时通知)，Linux事件驱动机制O(1)



### 异步的 AIO 如何进一步加工处理结果?

* 基于回调：实现 CompletionHandler 接口，调用时触发回调函数
* 返回 Future：通过 isDone() 查看是否准备好，通过 get() 等待返回数据



APIs
AsynchronousServerSocketChannel
AsynchronousSocketChannel



### 实现AIOEcho服务器(重点)

Echo：从客户端读取数据并原封不动写回去

```java
public class AIOPlainEchoServer {
    
    public void serve(int port) throws IOException {
        System.out.println("Listening for connections on port " + port);
        // 1. 打开一个AsynchronousServerSocketChannel，用于监听客户端的连接，
        final AsynchronousServerSocketChannel serverChannel = AsynchronousServerSocketChannel.open();
        InetSocketAddress address = new InetSocketAddress(port);
        // 将ServerSocket绑定到指定的端口里
        serverChannel.bind(address);
        final CountDownLatch latch = new CountDownLatch(1);
        // 2. 开始接受新的客户端请求，一旦一个客户端请求被接受，CompletionHandler就会被调用
        serverChannel.accept(null, new CompletionHandler<AsynchronousSocketChannel, Object>() {

            @Override
            public void completed(AsynchronousSocketChannel channel, Object attachment) {
                // 3_1. 一旦完成处理，再次接受新的客户端请求
                serverChannel.accept(null, this);
                ByteBuffer buffer = ByteBuffer.allocate(100);
                // 在channel里植入一个读操作EchoCompletionHandler，一旦从channel里读取到数据，EchoCompletionHandler就会被调用
                channel.read(buffer, buffer, new EchoCompletionHandler(channel));
            }

            @Override
            public void failed(Throwable exc, Object attachment) {
                try {
                    // 3_2. 若遇到异常，关闭Channel
                    serverChannel.close();
                } catch (IOException e) {
                    // ingnore on close
                } finally {
                    latch.countDown();
                }
            }
        });
        try {
            latch.await();
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }

    private final class EchoCompletionHandler implements CompletionHandler<Integer, ByteBuffer> {
        private final AsynchronousSocketChannel channel;

        EchoCompletionHandler(AsynchronousSocketChannel channel) {
            this.channel = channel;
        }

        @Override
        public void completed(Integer result, ByteBuffer buffer) {
            buffer.flip();
            // 在channel里植入一个写操作CompletionHandler,一旦channel有数据写入，CompletionHandler便会被唤醒
            channel.write(buffer, buffer, new CompletionHandler<Integer, ByteBuffer>() {

                @Override
                public void completed(Integer result, ByteBuffer buffer) {
                    if (buffer.hasRemaining()) {
                        // 如果buffer里还有内容，则再次触发写入操作将buffer里的内容写入到channel
                        channel.write(buffer, buffer, this);
                    } else {
                        buffer.compact();
                        // 如果channel里还有内容需要读入到buffer里，则再次触发写入操作将channel里的内容读入到buffer
                        channel.read(buffer, buffer, EchoCompletionHandler.this);
                    }
                }

                @Override
                public void failed(Throwable exc, ByteBuffer attachment) {
                    try {
                        channel.close();
                    } catch (IOException e) {
                        // ingnore on close
                    }
                }
            });
        }

        @Override
        public void failed(Throwable exc, ByteBuffer attachment) {
            try {
                // 若遇到异常，关闭Channel
                channel.close();
            } catch (IOException e) {
                // ingnore on close
            }
        }
    }

}

```



## Netty 网络NIO框架

Netty 是基于 Java NIO 的网络通讯框架。可以定制编码解码协议以实现服务器。并发高、传输快、封装好



### Channel

Channel 表示数据传输通道，每一个请求对应一个 Channel



<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting2/img/2023_09_25_01_46.webp" alt="img" style="zoom:67%;" />

ChannelPipeline - 用于保存处理过程需要用到的 ChannelHandler 和 ChannelHandlerContext 

ChannelHandler - 核心处理业务就在这里，用于处理业务请求

ChannelHandlerContext - 用于传输业务数据



### ByteBuf

作用：Netty 是基于 NIO 的，使用 ByteBuf 可以通过零拷贝对数据进行直接进行操作，从而加快传输速度。

ByteBuf 是一个存储字节的容器，它既有自己的读索引和写索引方便你对整段字节缓存进行读写，也支持get/set方便你对其中每一个字节进行读写

ByteBuf 的数据结构：

![img](https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting2/img/2023_09_25_01_37.webp)

ByteBuf 的三种使用模式：

1. Heap Buffer 堆缓冲区
   堆缓冲区是ByteBuf最常用的模式，他将数据存储在堆空间。

2. Direct Buffer 直接缓冲区

   直接缓冲区是ByteBuf的另外一种常用模式，他的内存分配都不发生在堆，jdk1.4引入的nio的ByteBuffer类允许jvm通过本地方法调用分配内存，这样做有两个好处 

   - 通过免去中间交换的内存拷贝, 提升IO处理速度; 直接缓冲区的内容可以驻留在垃圾回收扫描的堆区以外。
   - DirectBuffer 在 -XX:MaxDirectMemorySize=xxM大小限制下, 使用 Heap 之外的内存, GC对此”无能为力”,也就意味着规避了在高负载下频繁的GC过程对应用线程的中断影响.

3. Composite Buffer 复合缓冲区
   复合缓冲区相当于多个不同ByteBuf的视图，这是netty提供的，jdk不提供这样的功能。



### Codec 编码/解码器

Netty中的编码/解码器，通过他你能完成字节与pojo、pojo与pojo的相互转换，从而达到自定义协议的目的。
在Netty里面最有名的就是HttpRequestDecoder和HttpResponseEncoder了。

