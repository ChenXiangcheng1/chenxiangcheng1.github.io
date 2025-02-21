# Program_Language

TODO：https://jingwei.link/2020/05/21/go-java-python-language-model.html



金字塔：C++、Rust、Golang

|                | Python                                                       | Java                                                         | Golang                                      |
| -------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------- |
| 运行           | 脚本语言(解释型语言)：每次都需要编译                         | 编译型语言：编译成jar                                        |                                             |
| 异步并发模型   | **支持coroutine协程**async-**await**<br />协程本质是由**EventLoop**(主线程)调度的、通过await传递消息的一个可以挂起可恢复的函数<br />组合异步任务：通过事件循环处理await，**协程是一种更自然的异步编程模型**(类似于同步代码) 嘎嘎好用<br />Python协程底层是通过Generator实现的 | **回调 CompletableFuture** [美团#CompletableFuture原理与实践](https://mp.weixin.qq.com/s/GQGidprakfticYnbVYVYGQ)，<br />组合异步任务：Future通过链式调用，会导致回调地狱。CompletableFuture 通过几个链式接口解决了回调地狱问题<br />回调麻烦，**更倾向于使用**Thread、ExecutorService**多线程并发**<br />Java不支持协程、Kotlin编译器原生支持、Scala第三方库支持<br />Kotlin协程底层是通过状态机(不涉及OSIO多路复用)暂停并保存当前状态实现的<br />**async-await本质就是EventLoop的语法糖，Java是有NIO的EventLoop和async-await的EventLoop不是同个东西, Java就是没实现通用的协程** | 原始协程 (其它语言是通过其它底层机制实现的) |
| 多线程并发模型 | 由于GIL名存实亡                                              | JDK21支持 Project Loom虚拟线程(本质是**基于平台线程**的用户线程)、Spring也支持虚拟线程<br />Brian Goetz(Java并发大佬)：I think Project Loom is going to kill reactive programming.<br />老项目需要重写为Loom、ThreadLocal需要重写为ScopedValue<br />Netty: 线程模型使用EventLoop 提供 Future接口，无需虚拟线程<br />JDBC之所以不使用EventLoop 主要是因为ORM兼容性问题: 使用EventLoop 方案很多内容都需要重写，而虚拟线程方案可继续保持阻塞式的API 主要改动在连接池方面<br />[编程范式: openjdk#结构化并发](https://openjdk.org/jeps/453) |                                             |
| Generator      | 是特殊函数，使用 `yield` 关键字可以在函数执行过程中暂停并返回一个值，稍后可以从暂停的地方继续执行。<br />底层就是对迭代器的拓展 | 只支持 Future(检查异步任务是否完成、获取结果、取消任务)<br />不支持可暂停的函数(可等待对象) |                                             |
| 基本的GC机制   | 引用计数法+标记-清除(解决循环引用问题)                       | 分代ZGC (弱三色标记-整理+混合写屏障)                         | 弱三色标记-清除+混合写屏障                  |



## 流式接口

流式接口(以grpc的streaming rpc为例)



Python

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

Java
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

