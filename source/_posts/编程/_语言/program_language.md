# Program_Language

TODO：https://jingwei.link/2020/05/21/go-java-python-language-model.html



金字塔：~~C++~~、Rust、Golang

> Mark Russinovich(微软CTO) 2022:
> Speaking of languages, it's time to halt starting any new projects in C/C++ and use Rust for those scenarios where a non-GC language is required. For the sake of security and reliability. the industry should declare those languages as deprecated.

|                  | ~~Python~~                                                   | ~~Java~~                                                     | [Golang](https://github.com/golang/go)                       |                                   |
| ---------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | --------------------------------- |
| 解释器           | Bytecode First <br />CPython(.pyc字节码)                     | Bytecode First <br />OpenJDK JVM(.class)                     | Native First                                                 |                                   |
| 运行             | 脚本语言(解释型语言)：每次都需要编译                         | 编译型语言：编译成jar                                        |                                                              |                                   |
| 缺点             |                                                              |                                                              | 类型系统不完善<br />Golang接口不能包含field字段、不支持默认实现<br />不支持泛型方法(编译时类型检查) |                                   |
| 应用场景         |                                                              |                                                              | 高性能的网络服务、云计算基础设施、分布式系统                 |                                   |
| 函数形参传递方式 | 不可变类型(含基本类型)：值传递<br />可变类型(含数组、类)：引用传递(浅拷贝引用) | arr[0] = 100;  // 可以修改原数组void process(int[] arr) {基本类型：值传递<br />对象类型(含数组、类)：引用传递(**浅拷贝引用**) | 都是：值传递<br />值类型(固定大小)(含基本类型、**数组**、**struct(类)**)：不含指针的值传递(**值类型 完整拷贝**)<br />引用类型(动态大小)(**含Slice, Map, Chan, Interface**)：底层包含指针的结构体值传递(拷贝后共享底层)(其实是对指针传递的封装) | C++数组退化为指针，RUST所有权机制 |
| OOP继承          | 支持多重继承(MRO解决菱形继承问题)                            | 单继承                                                       | 组合                                                         |                                   |
| 异步并发模型     | **支持非抢占式协程**async-**await**<br />协程本质是由**EventLoop**(主线程)调度的、通过await传递消息的一个可以挂起可恢复的函数<br />组合异步任务：通过事件循环处理await，**协程是一种更自然的异步编程模型**(类似于同步代码) 嘎嘎好用<br />Python协程底层是通过Generator实现的 | }  arr[0] = 100;  // 可以修改原数组**回调 CompletableFuture** [美团#CompletableFuture原理与实践](https://mp.weixin.qq.com/s/GQGidprakfticYnbVYVYGQ)，<br />组合异步任务：Future通过链式调用，会导致回调地狱。CompletableFuture 通过几个链式接口解决了回调地狱问题<br />回调麻烦，**更倾向于使用**Thread、ExecutorService**多线程并发**<br />Java不支持协程、Kotlin编译器原生支持、Scala第三方库支持<br />Kotlin协程底层是通过状态机(不涉及OSIO多路复用)暂停并保存当前状态实现的<br />**async-await本质就是EventLoop的语法糖，Java是有NIO的EventLoop和async-await的EventLoop不是同个东西, Java就是没实现通用的协程** | 原始协程 (其它语言是通过其它底层机制实现的)                  |                                   |
| co-routine(协程) | 抢占式协程(协程调度器)：不支持抢占式协程<br />非抢占式协程(协作式)：async-await | 抢占式协程(协程调度器)：Virtual threads<br />非抢占式协程(协作式)：不支持非抢占式协程，提供CompletableFuture进行异步编程 | 抢占式协程(协程调度器)：goroutine  // A goroutine is a lightweight thread managed by the Go runtime.<br />非抢占式协程(协作式)：不支持非抢占式协程 |                                   |
| 多线程并发模型   | 由于GIL名存实亡                                              | }JDK21支持 Project Loom虚拟线程(本质是**基于平台线程**的用户线程)、Spring也支持虚拟线程<br />Brian Goetz(Java并发大佬)：I think Project Loom is going to kill reactive programming.<br />老项目需要重写为Loom、ThreadLocal需要重写为ScopedValue<br />Netty: 线程模型使用EventLoop 提供 Future接口，无需虚拟线程<br />JDBC之所以不使用EventLoop 主要是因为ORM兼容性问题: 使用EventLoop 方案很多内容都需要重写，而虚拟线程方案可继续保持阻塞式的API 主要改动在连接池方面<br />[编程范式: openjdk#结构化并发](https://openjdk.org/jeps/453) |                                                              |                                   |
| Generator        | 是特殊函数，使用 `yield` 关键字可以在函数执行过程中暂停并返回一个值，稍后可以从暂停的地方继续执行。<br />底层就是对迭代器的拓展 | 只支持 Future(检查异步任务是否完成、获取结果、取消任务)<br />不支持可暂停的函数(可等待对象) |                                                              |                                   |
| 基本的GC机制     | 引用计数法+标记-清除(解决循环引用问题)                       | 分代ZGC (弱三色标记-整理+混合写屏障)                         | 弱三色标记-清除+混合写屏障<br />小对象(array struct)栈内联优化(不用gc) |                                   |
| 其它区别         | 访问控制并不强制(`_ClassName__var` 可以访问 `__var`)         |                                                              | 和C++指针不同，Golang指针本质是含指向接口具体实现指针的struct<br /><br />没有继承、工厂方法取代构造函数 |                                   |



## 函数形参传递方式(数组)

C++数组：指针传递
Java、Python数组：引用传递
Golang数组：值传递

本质是：Java、Python将数组看作对象、Golang将数组看作值类型(同时引入切片作为补充)、C++数组退化为指针

```java
int[] arr = new int[5];  // arr 是引用，指向堆上的数组对象
void process(int[] arr) {
    arr[0] = 100;  // 可以修改原数组
}
// Java使用同一的内存模型，对象都分配在堆上，避免大对象复制带来的性能开销
// Java数组是对象
```



```go
var arr [5]int         // arr 直接存储数组内容
func process(arr [5]int) {
    arr[0] = 100       // 修改的是副本
}

// Golang中值类型分配在堆上
// Golang数组是值类型

// 所以在Golang中
// 小数组可以在栈上分配
arr := [3]int{1, 2, 3}  // 栈分配，性能好
// 大数组建议使用切片
slice := make([]int, 1000)  // 底层数组在堆上
```



## 流式接口

流式接口(以grpc的streaming rpc为例)



### Python

以Generator、Iterator方式处理

```python
# 服务端
def hello_TwoWayStream(self, request_iterator: grpc._server._RequestIterator, context: grpc._server._Context):
    for request in request_iterator:
        request: hello_proto_pb2.TestTwoWayStreamRequest
        data = request.data
        yield hello_proto_pb2.TestTwoWayStreamResponse(result=f"Server received: {data}")

# 客户端
def test_TwoWayStream():
    channel: Channel = grpc.insecure_channel(f"{_ADDRESS}:{_PORT}")
    stub = hello_proto_pb2_grpc.Hello_serviceStub(channel)
    req = _create_generator()
    response: grpc._channel._MultiThreadedRendezvous = stub.hello_TwoWayStream(req, timeout=10)
    for res in response:
        res: hello_proto_pb2.TestTwoWayStreamResponse
        print(res.result)
```

### Java

没有Generator，以泛型类StreamObserver<Hello_func_Request>处理请求、StreamObserver<Hello_func_Response>发送响应

```java
public StreamObserver<Hello_func_Request> hello_TwoWayStream(
        StreamObserver<Hello_func_Response> responseObserver) {
    return new StreamObserver<Hello_func_Request>() {
        @Override
        public void onNext(Hello_func_Request request) {
            // 处理每个请求
            System.out.println("Received: " + request.getStrField());
            Hello_func_Response response = Hello_func_Response.newBuilder()
                    .setResult("Hello, " + request.getStrField())
                    .build();
            responseObserver.onNext(response);  // 发送响应
        }
    };
}

// 客户端
StreamObserver<Hello_func_Request> requestObserver = stub.hello_TwoWayStream(new StreamObserver<Hello_func_Response>() {
    @Override
    public void onNext(Hello_func_Response response) {
        System.out.println("Received: " + response.getResult());
    }
});
for (int i = 0; i < 5; i++) {
    Hello_func_Request request = Hello_func_Request.newBuilder()
            .setStrField("Message " + i)
            .build();
    requestObserver.onNext(request);
    Thread.sleep(1000);
}
requestObserver.onCompleted();
```

