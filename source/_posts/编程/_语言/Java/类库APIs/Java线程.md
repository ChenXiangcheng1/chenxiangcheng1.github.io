---
title: Java线程
date: 2023-07-13
update: 2023-09-16
tags:
categories:
  - Java
  - 类库
keywords: Java,Java线程,锁,synchronized,J.U.C并法包
description: 关于Java线程，涉及创建线程、线程方法、线程状态转移，线程同步机制，synchronized和volatile底层实现原理，Excutors中创建线程池的工厂方法，ThreadLocal线程局部变量
top_img:
cover: https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_09_13_01_53.png" alt="image-20230913015318274
---



# Java线程主要考点 

## 并行与并发的区别

并行就是多核CPU分别执行多个进程

并发就是单核CPU轮转调度法



## 进程和线程的区别

总结：

* 进程是资源分配的最小单位，线程是 CPU 调度的最小单位

* 进程有独立的地址空间，**线程属于某个进程共享进程的内存资源**，即每个进程对应一个 JVM 实例有独立的地址空间，多个线程共享 GC 堆
* 多进程的程序比多线程程序健壮

* 线程的切换比进程的切换开销小

Java 采用单线程编程模型，程序会自动创建主线程。由于是单线程所以需要自己将耗时任务放入子线程。



线程和进程的由来：

1. 串行：初期的计算机只能串行执行任务，并且需要长时间等待用户输入
2. 批处理：预先将用户的指令集中成清单，批量串行处理用户指令，仍然无法并发执行
3. 进程：进程独占内存空间，保存各自运行状态，*相互间不干扰*且可以互相切换，为并发处理任务提供了可能
4. 线程：共享进程的内存资源，相互间切换更快速，支持更细粒度的任务控制，使进程内的*子任务*得以并发执行



结构：

|      | 进程                                                         | 线程                                                         |
| ---- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 结构 | 所有与进程相关的资源，都被记录在 PCB 中<br /><img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_06_02_15_16.png" alt="image-20230602151654618" style="zoom:50%;" /> | 线程只由堆栈寄存器、程序计数器和 TCB 组成<br /><img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_06_02_15_18.png" alt="image-20230602151826556" style="zoom:80%;" /> |
| 作用 | 进程是资源分配的最小单位                                     | 线程是 CPU 调度的最小单位                                    |
|      |                                                              | 线程属于某个进程，共享其资源                                 |



## 创建线程

### 继承Thread类

因为 OOP 的单一继承原则(一个父类) 限制，不推荐这种使用方式，使用 Runnable 或 Callable 接口更灵活

```java
public class MyThread extends Thread {
    public void run() {
        // 线程执行的逻辑
}	}
MyThread thread = new MyThread();
thread.start();
```

```java
// 匿名内部类
Thread thread = new Thread() {  
    public void run() {
        // 线程执行的逻辑
    }
};
thread.start();
```

```java
// Lambda表达式
Thread thread = new Thread(() -> {
    // 线程执行的逻辑
});
thread.start();
```



### 实现Runnable接口

Runnable 接口适用于无返回值的任务

```java
public class XXServiceTask implements Runnable {
    public void run() {
        // 线程执行的逻辑
    }
}
```

```java
Thread thread = new Thread(new XXServiceTask());
thread.start();
```



### 实现Callable接口

Callable 函数式接口适用于有返回值的任务，任务执行完成后可以通过 `Future` 对象的 `get()` 方法获取任务的执行结果。 

```java
public class XXServiceCallable implements Callable<String> {
    @Override
    public String call() throws Exception{
        String result = "";
        // 线程执行的逻辑
        return  value;
}	}
```

示例1：

```java
public class FutureTaskDemo {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        FutureTask<String> task = new FutureTask<String>(new XXServiceCallableallable());  //返回Future对象
        new Thread(task).start();
        if(!task.isDone()){
            System.out.println("任务未完成!");
        }
        System.out.println("任务返回: " + task.get());  // 通过Future对象获取任务执行结果
}	}
```

示例2：

```java
public class ThreadPoolDemo {
    public static void main(String[] args) {
        ExecutorService newCachedThreadPool = Executors.newCachedThreadPool();
        Future<String> future = newCachedThreadPool.submit(new XXServiceCallable());  //返回Future对象
        if(!future.isDone()){
            System.out.println("任务未完成!");
        }
        try {
            System.out.println("任务返回: " + future.get());  // 通过Future对象获取任务执行结果
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        } finally {
            newCachedThreadPool.shutdown();
}	}	}
```



## Runnable

### Thread

[API 文档#Thread](https://devdocs.io/openjdk~17/java.base/java/lang/thread)

Thread 实现了 Runnable 接口，表示一个线程



`currentThread(): Thread` 返回对当前正在执行的线程对象的引用

`getName(): String` 返回线程名字

`getState(): State` 返回线程状态 



#### 构造方法

`Thread()` 构造方法

`Thread(Runnable)` 构造方法，分配一个新的线程对象，其运行对象为 Runnable，名称为 String



#### `start():void` 

`start():void` 主线程调用本地方法 `start0()` 方法创建子线程后返回，子线程再调用该线程的 `Thread#run()` 方法。当线程执行完毕不能 restart

```java
// Thread.java
public synchronized void start() {    
    if (threadStatus != 0)  // threadStatus != NEW
        throw new IllegalThreadStateException();
    group.add(this);
    boolean started = false;
    try {
        start0();
        started = true;
    } finally {
        try {
            if (!started) {
                group.threadStartFailed(this);
            }
        } catch (Throwable ignore) {
}	}	}
```

```java
private native void start0();  // JNI本地方法
```

```c
//https://github.com/openjdk/jdk/blob/f55e799491c39dcaf7b3935b6d560ee0a3239191/src/java.base/share/native/libjava/Thread.c#L39
static JNINativeMethod methods[] = {
    {"start0", "()V", (void *)&JVM_StartThread},
	// ...
};
```

```java
//https://github.com/openjdk/jdk/blob/f55e799491c39dcaf7b3935b6d560ee0a3239191/src/hotspot/share/prims/jvm.cpp#L2999
JVM_ENTRY(void, JVM_StartThread(JNIEnv* env, jobject jthread))  // JVM的入口点
  ...
  {
      native_thread = new JavaThread(&thread_entry, sz);  // 创建一个C++级别的线程，thread_entry是Java线程的入口点
      ...
  }
  ...
JVM_END
```

```cpp
// https://github.com/openjdk/jdk/blob/master/src/hotspot/share/prims/jvm.cpp
static void thread_entry(JavaThread* thread, TRAPS) {
  HandleMark hm(THREAD);
  Handle obj(THREAD, thread->threadObj());
  JavaValue result(T_VOID);
  JavaCalls::call_virtual(&result,  // 调用Java级别的方法，即调用该线程的thread#run()方法
                          obj,
                          vmClasses::Thread_klass(),
                          vmSymbols::run_method_name(),
                          vmSymbols::void_method_signature(),
                          THREAD);
}
```



#### `run():void` 

`run():void` 主线程调用该 Runnable 成员变量的 `run()` 方法

```java
// Thread.java
private Runnable target;
public void run() {
    if (target != null) {
        target.run();
}	}
```



##### 构造函数传参

```java
public class XXServiceCallable implements Callable<String> {
    private int param1;    
    public XXServiceCallable(int param) {
        param1 = param;
    }
    @Override
    public String call() throws Exception{
        String result = "";
        // 线程执行的逻辑
        return  value;
}	}
```



##### set方法传参

```java
public class XXServiceCallable implements Callable<String> {
    private int param1;    
    public void setParam1(int param) {
        param1 = param;
    }
    @Override
    public String call() throws Exception{
        String result = "";
        // 线程执行的逻辑
        return  value;
}	}
```



##### 回调函数传参

[回调函数](./Java的IO机制.md#事件驱动)	|	[匿名函数](./Lambda.md#匿名函数)

```java
public interface Callback {  // 匿名函数接口
    void execute(int param);
}
```

```java
public class XXServiceCallable implements Callable<String> {
    private Callback param1;    
    public XXServiceCallable(Callback param) {  // 回调函数传参
        param1 = param;
    }
    @Override
    public String call() throws Exception{
        String result = "";
        // 线程执行的逻辑
        return  value;
}	}
```

```java
public class FutureTaskDemo {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        FutureTask<String> task = new FutureTask<String>(new XXServiceCallable(param -> {
            // 处理回调函数的逻辑，使用param进行操作
        }));                
        new Thread(task).start();
        if(!task.isDone()){
            System.out.println("任务未完成!");
        }
        System.out.println("任务返回: " + task.get());
}	}
```



##### 实现处理线程的返回值

* 主线程等待法。缺点，没法精准控制，代码臃肿

```java
while (cw.value == null){
	Thread.currentThread().sleep(100);
}
```

* 使用 Thread 类的 join() 阻塞当前线程以等待子线程处理完毕。缺点，粒度不够细，无法实现更精准的控制
* **Callable+Future**。`Callable` 函数式接口的 `call()` 方法与 `Runnable` 接口的`run()`方法不同， `call()` 方法可以返回一个结果。任务执行完成后，可以通过 `Future` 对象获取任务的执行结果。 



#### `Thread.sleep(long):void`

`Thread.sleep(long):void` 使当前线程睡眠(暂时停止执行)，该线程不会失去任何 monitor 锁的所有权



#### `yield(): void`

`yield(): void` 向调度程序提示当前线程愿意出让其当前对处理器的使用。调度程序可以随意忽略此提示。使用 yield() 是为了改善线程之间的相对进度，否则会过度利用CPU。这个方法没什么用。

轻量级线程采用协作式调度方式，也称为合作式调度。在协作式调度中，线程主动释放CPU控制权，让其他线程执行，而不是由操作系统强制进行调度。线程在适当的时机显式地主动让出CPU，通过调用特定的函数或指令，通常称为 yield 或 yielding 操作。



#### `interrupt():void`

`interrupt():void` 如果此线程被阻塞在调用 Object 类的 wait()、join()、Thread.sleep() 方法，那么清除中断状态并抛出 InterruptedException 异常。如果线程处于正常活动状态，那么该线程的中断标志将被设置为 true 线程不受影响继续正常运行，通过 `isInterrupted(): Boolean` 测试该线程是否被中断，需要手动检查进行处理

示例：

```java
Thread t1 = new Thread(() -> {
    while (!isInterrupted()) {
            // 业务逻辑
            if (isInterrupted()) {  // 手动检查中断标志
                // 安全地停止线程、释放资源
                break;
            }
        }
}).start();
```

`stop()` 已被抛弃的方法，太过暴力(线程停止清理工作无法完成)且不安全(立刻释放锁可能引发数据不同步)
`suspend()` 已被抛弃的方法
`resume()` 已被抛弃的方法



#### `join(): void` 

`join(): void` 等待 this 线程死亡， 当一个线程终止时会调用 this.notifyAll() 

**等效于执行 this线程.wait() 使当前线程等待直到 this 线程终止被 this线程.notifyAll() 唤醒**

```java
// Thread.java
public final synchronized void join(final long millis) throws InterruptedException {
    if (millis > 0) {
        if (isAlive()) {
            final long startTime = System.nanoTime();
            long delay = millis;
            do {
                wait(delay);
            } while (isAlive() && (delay = millis -
                    TimeUnit.NANOSECONDS.toMillis(System.nanoTime() - startTime)) > 0);
        }
    } else if (millis == 0) {
        while (isAlive()) {  // 若this线程存活
            wait(0);  // this线程.wait() 使当前线程等待直到被唤醒(this线程.notifyAll)
        }
    } else {
        throw new IllegalArgumentException("timeout value is negative");
}	}
```

示例：

```java
@Test
public void test3() throws InterruptedException {
    Thread t1 = new Thread(() -> {
        for (int i = 0; i < 5; i++) {
            try {
                Thread.sleep(500);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            System.out.println("子线程A" + i);
        }
    });
    Thread t2 = new Thread(() -> {
        for (int i = 0; i < 5; i++) {
            try {
                Thread.sleep(500);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            System.out.println("子线程B" + i);
        }
    });
    t1.start();
    t2.start();
    for (int i = 0; i < 5; i++) {
        if (i == 1) {
            t2.join();  // 主线程等待 t2.notify()
        }
        Thread.sleep(500);
        System.out.println("主线程" + i);
    }
}
```

```java
主线程0
子线程B0
子线程A0
子线程B1
子线程A1
子线程B2
子线程A2
子线程B3
子线程A3
子线程B4
子线程A4
主线程1
主线程2
主线程3
主线程4
```



### RunnableFuture<V>

[J.U.C并发包笔记#Future 接口](./J.U.C并发包.md#Future 接口)

```java
public interface RunnableFuture<V> extends Runnable, Future<V> {
    void run();
}
```



#### FutureTask<V>

实现了 RunnableFuture<V> 接口，可以包装一个 Callable 或 Runnable 对象，表示一个异步执行的任务。



##### `FutureTask(Callable<V>) `

`FutureTask(Callable<V>) `构造函数



##### `FutureTask(Runnable, V)`

`FutureTask(Runnable, V)` 构造函数



##### 示例

```java
FutureTask<String> task = new FutureTask<String>(new MyCallable());
// Demo1: FatureTask可以结合Thread使用
new Thread(task).start();  // 创建线程，任务FatureTask给线程池  
// Demo2: FatureTask可以结合线程池使用    
ExecutorService newCachedThreadPool = Executors.newCachedThreadPool();
Future<String> future = newCachedThreadPool.submit(task);  // 创建线程池，任务FatureTask给线程池
// 通过异步计算的结果future，可以阻塞地获取线程的返回值
```



## Object 

[LiteAPIs笔记#Object](./LiteAPIs.md#Object)

### `wait():void`

`wait():void` **使当前线程等待，直到被唤醒**，通常是通过 notify() 或超时中断。当前线程必须拥有 monitor 锁对象，如果没有会抛出IllegalMonitorStateException 异常

* 等待：当前线程将其自身置于该对象**等待池**_WaitSet中，然后放弃所有同步声明，即 _owner = null **释放 monitor 锁**。

* 唤醒：需要被唤醒线程从该对象的等待集合移除，被唤醒的线程还不能继续直到重新竞争 monitor 锁，得到后 wait() 调用结束
  唤醒方式：
  1其他线程调用该对象的 notify() 方法，并且线程恰好被任意选择为要唤起的线程。
  2其他线程调用该对象的 notifyall() 方法
  3其他线程中断该线程
  4定时结束



### `notify():void`

`notify():void` 唤醒正在该对象 monitor 上等待的**一个单独的线程**，被唤醒的线程还需要竞争 monitor 锁才能启动，此方法只能由拥有该对象 monitor 锁的线程调用。



获取 monitor 锁的方式：
1执行 synchronized 方法
2执行 synchronized 语句块



### `notifyAll():void`

`notifyAll():void` 唤醒正在该对象 monitor 上等待的**所有线程**。被唤醒的线程还需要竞争 monitor 锁才能启动，此方法只能由拥有该对象 monitor 锁的线程调用。



## 线程的生命周期

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_09_13_01_53.png" alt="image-20230913015318274" style="zoom: 67%;" />

* **新建状态** (NEW)：线程创建后还没 start() 启动
* **运行状态** (RUNNABLE)：线程正在 JVM 中执行，但它可能正在 wait 来自操作系统的其他资源，例如处理器时间片
* **阻塞状态** (BLOCKED)：进入同步块，等待获取 monitor 排它锁
* **非限期等待状态** (WAITING)：等待另一个线程执行特定操作(显式被唤醒)
  * 对一个对象调用 `Object.wait()` 的线程等待另一个线程对该对象调用 `Object.notify()` 或 `Object.notifyAll()`
  * 调用 `Thread.join()` 的线程等待指定线程终止
  * `LockSupport.park()` 方法
* **限期等待状态** (TIMED_WAITING)：等待(在一定时间后由系统自动唤醒)
  * `Thread.sleep()` 方法
  * 设置了 Timeout 参数的 `Object.wait()` 方法
  * 设置了 Timeout 参数的 `Thread.join()` 方法
  * `LockSupport.parkNanos()` 方法
  * `LockSupport.parkUntil()` 方法
* **终止状态** (Terminated)：已终止线程的状态，线程已经结束执行
  已终止的线程不能再一次启动 start ，在终止的线程上调用 `Thread.start()` 方法会抛出 `IllegalThreadStateException` 异常

```java
// Thread.java
public enum State {  // 线程状态枚举类型
        NEW,
        RUNNABLE,
        BLOCKED,
        WAITING,
        TIMED_WAITING,
        TERMINATED;
}
```



## 线程同步机制

多线程同步机制，就是通过 happens-before 顺序性保证，使各线程操作满足原子性、有序性、可见性



> **happens-before 关系**用于确定并发程序中操作的顺序和内存可见性。
> hb(x,y) 表示x操作发生在y操作之前，x操作对y操作可见。
>
> **happens-before 顺序**：
>
> * hb(**同一线程中**按代码顺序x在y之前, y)
> * hb(对象的构造方法, 对象的终止)
> * hb(x, y) hb(y, z) 则 hb(x, z)
> * hb(释放 monitor 锁操作, 获取 monitor 锁)       // 使得 synchronized 得到同步
>
> * hb(写入volatile字段, 读取volatile字段)               // 使得 volatile 具有修改可见性
> * hb(线程start(), 在已启动线程上的任何操作)
> * hb(线程所有操作, 其他线程调用该线程.join())
> * hb(任何对象的初始化, 程序的任何其他操作)



>编译器优化 - **指令重排序**
>
>重排序不能打破 happens-before 顺序
>存在数据依赖关系的不允许重排序



**Java内存模型**
Java 内存模型定义多线程并发访问共享内存的规范，决定了程序中每个点可以读取哪些值。

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_06_11_23_28.png" alt="image-20230611232832468" style="zoom: 50%;" />



### 发生死锁

发生死锁：申请锁时发生了交叉的闭环申请，例如带锁方法的嵌套

死锁的的四个必要条件：

* 互斥条件：一个锁只能被一个线程占用
* 请求与保持条件：等待获取其他锁时，继续持有锁
* 不可抢占条件：已获得的锁不能被其他线程抢占
* 循环等待条件：多线程形成循环争锁

发生死锁的常见场景：
	嵌套锁：一个方法获取锁，又调用自己或其它获取该锁的方法，死锁了
	锁顺序：两个方法获取锁的顺序不对

避免死锁：
	使用超时机制
	资源分级按顺序申请锁
	避免嵌套锁：取消子方法的锁，添加文档注释 该方法不是线程安全的调用之前需要获取锁



```java
// 死锁代码
public class ThreadLockDemo {
    static class SynAddRunalbe implements Runnable {
        int a, b;
        public SynAddRunalbe(int a, int b) {
            this.a = a;
            this.b = b;
        }
        @Override
        public void run() {
            synchronized (Integer.valueOf(a)) {  // 返回Integer实例，会缓存缓存经常请求的值，缓存范围(-128,127]
                synchronized (Integer.valueOf(b)) {
                    System.out.println(a + b);
    }	}	}	}
    public static void main(String[] args) {
        for (int i = 0; i < 100; i++) {
            new Thread(new SynAddRunalbe(1, 2)).start();
            new Thread(new SynAddRunalbe(2, 1)).start();  // 没有按顺序获取锁
}	}	}
```



### 互斥锁

[锁的分类](../../MySQL/MySQL8.md#锁的分类)

互斥锁的特性：

* 互斥性：在任意时刻，只有一个线程能够持有该锁
* 可见性：指当一个线程对共享变量进行修改后，其他线程能够立即看到该修改。
* 可重入性：一个线程能重复获取它已经拥有的锁。不可重入会增大死锁的情况



#### CAS

CAS (Compare And Swap) 是原子操作，比 monitor 锁效率高，用于实现线程安全性的方法，属于乐观锁

原理：
	操作数：内存位置(V)、预期原值(A)、新值(B)
	基于原子操作：若V处值与A相等则写入B，若不相等则重试

优点：
	无重量级锁 free-lock，各线程是非阻塞的一个线程执行其余线程自旋

缺点：
	ABA问题，所以只能**保证一个共享变量的原子操作**，而不能实现原子方法 (多个共享变量需要用锁) 
	若线程长时间抢不到锁，**自旋**重试会消耗CPU性能

应用：
	J.U.C 并发包中的 atomic 原子数据类型，通过当前线程数据值为A与内存值比较
	J.U.C 并发包中对一个共享变量进行原子操作的方法，通常需要提供V、A、B参数



##### 原子类型

[J.U.C并发包笔记#原子类型](./J.U.C并发包.md#3原子类型 java.util.concurrent.atomic)



#### synchronized

synchronized 关键字，执行线程获取互斥锁，使得多线程能够同步访问一个对象。

每个对象都有一个与之关联的 monitor 锁。按照惯例，一个线程在需要独占且一致地访问一个对象的字段之前，需要先获取该对象的 monitor 锁

示例1：

```java
class Test {
    public static void main(String[] args) {
        Test t = new Test();
        synchronized(t) {
            synchronized(t) {  // 具有可重入性
                System.out.println("made it!");
}	}	}	}
```

```运行
made it!
```

示例2：粒度粗

```java
public class Box {
    private Object boxContents;
    public synchronized Object get() {  //  多线程同时执行，同一Box实例的get()调用与put()调用同步
        Object contents = boxContents;
        boxContents = null;
        return contents;
    }
    public synchronized boolean put(Object contents) {  //
        if (boxContents != null) return false;
        boxContents = contents;
        return true;
    }
}
```

示例3：粒度细

```java
public class MsLunch {
    private long c1 = 0;
    private long c2 = 0;
    private Object lock1 = new Object();
    private Object lock2 = new Object();
    public void inc1() {
        synchronized(lock1) {  // 多线程同时执行，c1字段的所有更新同步，但c1更新不与c2更新同步
            c1++;
    }	}
    public void inc2() {
        synchronized(lock2) {  // 同理
            c2++;
}	}	}
```



##### 锁的对象

* this 实例对象 (调用该方法的对象)
  * 同步代码块
    synchronized(this)
    synchronized(类实例对象)
  * 同步非静态方法
    synchronized method

* 类的 Class 对象 
  * 同步代码块
    synchronized(类.class)
  * 同步静态方法
    synchronized static method

锁对象可选择专门的锁对象、final int、final Object，不要选择[包装类型](../OOPs/包装类型.md)



##### 字节码层面

synchronized 在字节码层面的具体语义实现：

```java
public class SyncBlockAndMethod {
    public void syncsTask() {  
        synchronized (this) {  // 1. 同步代码块
            System.out.println("Hello");
            synchronized (this){
                System.out.println("World");
    }	}	}
    public synchronized void syncTask() {  // 2. 同步非静态方法
        System.out.println("Hello Again");
}	}
```

```powershell
public void syncsTask();					  // 1. 具有同步代码块的方法
    descriptor: ()V
    flags: (0x0001) ACC_PUBLIC
    Code:
      stack=2, locals=5, args_size=1
         0: aload_0
         1: dup
         2: astore_1
         3: monitorenter					  // 同步代码块的开始位置
         4: getstatic     #7                  // Field java/lang/System.out:Ljava/io/PrintStream;
         7: ldc           #13                 // String Hello
         9: invokevirtual #15                 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        12: aload_0
        13: dup
        14: astore_2
        15: monitorenter					  // 同步代码块的开始位置
        16: getstatic     #7                  // Field java/lang/System.out:Ljava/io/PrintStream;
        19: ldc           #21                 // String World
        21: invokevirtual #15                 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        24: aload_2
        25: monitorexit						  // 同步代码块的结束位置，释放Monitor锁，并赋值计数器_count=0
        26: goto          34
        29: astore_3
        30: aload_2
        31: monitorexit						  // 同步代码块的结束位置，释放Monitor锁，并赋值计数器_count=0
        32: aload_3
        33: athrow
        34: aload_1
        35: monitorexit						  // 在异常处理器执行完成时，释放Monitor的指令
        36: goto          46
        39: astore        4
        41: aload_1
        42: monitorexit						  // 在异常处理器执行完成时，释放Monitor的指令
        43: aload         4
        45: athrow
        46: return
```

```powershell
public synchronized void syncTask();		  // 2. 同步非静态方法
    descriptor: ()V
    flags: (0x0021) ACC_PUBLIC, ACC_SYNCHRONIZED  // ACC_SYNCHRONIZED 访问标志，执行线程需要持有Monitor锁，该方法无论是否正常结束都会释放Monitor锁
    Code:
      stack=2, locals=1, args_size=1
         0: getstatic     #7                  // Field java/lang/System.out:Ljava/io/PrintStream;
         3: ldc           #23                 // String Hello Again
         5: invokevirtual #15                 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
         8: return   
```



##### 锁升级机制

[OpenJDK Wiki#sysnchronized](https://wiki.openjdk.org/display/HotSpot/Synchronization)

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_09_17_04_08.png" alt="image-20230917040803387" style="zoom: 70%;" />

| 锁的状态                         | 状态转移                                                     | 优点                               | 缺点                                                         | 应用场景                                                     |
| -------------------------------- | ------------------------------------------------------------ | ---------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 无锁                             |                                                              |                                    |                                                              |                                                              |
| 轻量级锁                         | CAS 操作修改锁对象的 MarkWord 并拷贝 MarkWord 到线程栈上的 Lock Record。<br />**各线程是非阻塞的一个线程执行其余线程自旋** | 竞争的线程不会阻塞，提高了响应速度 | 若线程长时间抢不到锁，自旋会消耗CPU性能                      | 线程交替执行同步块<br />**锁竞争时膨胀为重量级锁**(一个线程获取已被其他线程持有的锁) |
| 重量级锁                         | **每个对象都有一个与之关联的 monitor 锁。**按照惯例，一个线程在需要独占且一致地访问一个对象的字段之前，需要先获取该对象的 monitor 锁 | 线程竞争不使用自旋，不会消耗CPU    | 线程阻塞，响应时间缓慢，在多线程下，频繁的获取释放锁，会带来巨大的性能消耗 | 追求吞吐量，同步块或者同步方法执行时间较长的场景             |
| ~~无锁无偏向但启用了偏向锁模式~~ | 锁对象的 MarkWord 中 ThreadID = 0                            |                                    |                                                              | 偏向锁模式的延迟机制：HotSpot VM 启动的 4s 后才会对新建的每个对象开启偏向锁模式 |
| ~~偏向锁~~ JDK15弃用且禁用       | 获取偏向锁/释放锁后，依然是偏向锁状态<br />匿名偏向状态转化为偏向锁状态时，是通过 CAS 将将当前线程ID设置到锁对象的 MarkWord 中<br />持有偏向锁的线程进入同步块，JVM不会进行任何同步操作<br />锁撤销：调用hashcode() | 消除CAS 操作的开销                 | 如果线程间存在锁竞争，会带来额外的锁撤销的消耗               | 在大多数情况下，锁不存在竞争，还总是由同一线程多次获得       |



```java
Object obj = new Object();
new Thread(() -> {
    System.out.println(Thread.currentThread().getName() + "开始执行 \n"
            + ClassLayout.parseInstance(obj).toPrintable());
    synchronized (obj) {
        System.out.println(Thread.currentThread().getName() + "获取到锁 \n"
                + ClassLayout.parseInstance(obj).toPrintable());
    }
    System.out.println(Thread.currentThread().getName() + "释放锁 \n"
            + ClassLayout.parseInstance(obj).toPrintable());
}, "ThreadA").start();
```

```运行
ThreadA开始执行 
java.lang.Object object internals:
OFF  SZ   TYPE DESCRIPTION               VALUE
  0   8        (object header: mark)     0x0000000000000001 (non-biasable; age: 0)
  8   4        (object header: class)    0x00000d58
 12   4        (object alignment gap)    
Instance size: 16 bytes
Space losses: 0 bytes internal + 4 bytes external = 4 bytes total

ThreadA获取到锁 
java.lang.Object object internals:
OFF  SZ   TYPE DESCRIPTION               VALUE
  0   8        (object header: mark)     0x0000002f00aff4c8 (thin lock: 0x0000002f00aff4c8)
  8   4        (object header: class)    0x00000d58
 12   4        (object alignment gap)    
Instance size: 16 bytes
Space losses: 0 bytes internal + 4 bytes external = 4 bytes total

ThreadA释放锁 
java.lang.Object object internals:
OFF  SZ   TYPE DESCRIPTION               VALUE
  0   8        (object header: mark)     0x0000000000000001 (non-biasable; age: 0)
  8   4        (object header: class)    0x00000d58
 12   4        (object alignment gap)    
Instance size: 16 bytes
Space losses: 0 bytes internal + 4 bytes external = 4 bytes total
```



###### 轻量级锁CAS

**轻量级锁的内存语义**

当线程获取锁时，JVM 将线程虚拟机栈中的共享变量值置为无效，使得线程需要从GC堆中读取共享变量

当线程释放锁时，JVM 将线程虚拟机栈中的共享变量刷新到GC堆中

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_06_11_10_20.png" alt="image-20230611102022261" style="zoom: 67%;" />

<center>轻量级锁通信</center>

**轻量级锁原理**

[上面#CAS笔记](./Java线程.md#CAS)



**MarkWord**

[OpenJDK Github#MarkWord](https://github.com/openjdk/jdk/blob/3b0a6d2a6842962218b8cebcd9c0672cb4ee6720/src/hotspot/share/oops/markWord.hpp#L104)

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_06_11_00_25.png" alt="image-20230611002515557" style="zoom: 67%;" />

| hotspot 对象在内存中的布局                  |
| :------------------------------------------ |
| 对象头 = Mark Word + Class Metadata Address |
| 实例数据                                    |
| 对齐填充                                    |

| 对象头结构             | 说明                                                         |
| ---------------------- | ------------------------------------------------------------ |
| Mark Word              | 是非固定数据结构，**存储对象运行时数据**，默认存储对象的 hashCode、分代年龄、锁类型、锁标志位、Monitor 等信息 |
| Class Metadata Address | 类型指针，**指向对象的类元数据**，JVM 通过这个指针确定该对象是哪个类的数据 |



> **偏向锁(已被弃用)**
>
> Biased Locking偏向锁：**减少同一线程获取锁的代价，避免了 CAS 操作**
>
> **升级条件：**偏向锁运行在一个线程进入同步块的情况下
>
> **应用场景：**大多数情况下，锁不存在多线程竞争，总是由同一线程多次获得。不适用于锁竞争比较激烈的多线程场合。
>
> **核心思想：**如果一个线程获得了锁，那么锁就进入偏向模式，此时 Mark Word 的结构也变为偏向锁结构，当该线程再次请求锁时，无需再做任何同步操作即获取锁的过程只需要检查 **Mark Word 中的锁标记位** == 偏向锁 以及 当前线程 Id == **Mark Word 中的 ThreadID** 即可，这样就省去了大量有关锁申请的操作。
>
> 
>
> **轻量级锁**
>
> Lightweight Locking
>
> **升级条件：**轻量级锁是由偏向锁升级来的，当第二个线程加入锁争用的时候，偏向锁就会升级为轻量级锁。
>
> **应用场景：**线程交替执行同步块
>
> **上锁过程：**
>
> 1. 在代码进入同步块的时候，如果同步对象锁状态为无锁状态 (锁标志位为"01"状态)时，虚拟机首先将在当前线程的栈帧 (线程私有) 中建立一个名为锁记录 (Lock Record) 的空间，用于存储锁对象目前的 **Mark Word** 的拷贝，官方称之为 **Displaced Mark Word**。这时候线程堆栈与对象头的状态如图所示。
>
> <img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_06_11_00_14.png" alt="image-20230611001406207" style="zoom:75%;" />
>
> <center>无锁状态</center>
>
> 2. 拷贝对象头中的 Mark Word 复制到锁记录中。
>
> 3. 拷贝成功后，虚拟机将使用 **CAS(Compare And Swap)无锁算法操作**尝试将对象的Mark Word更新为指向 Lock Record 的指针，并将Lock record里的 owner指针 指向 object mark word。如果更新成功，则执行步骤(4)，否则执行步骤(5)。
> 4. 如果这个更新动作成功了，那么这个线程就拥有了该对象的锁，并且对象 Mark Word 的锁标志位设置为"00"，即表示此对象处于轻量级锁定状态，这时候线程堆栈与对象头的状态如图所示。
>
> <img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_06_11_00_13.png" alt="image-20230611001356510" style="zoom:80%;" />
>
> <center>轻量级锁</center>
>
> 5. 如果这个更新操作失败了，虚拟机首先会检查对象的 Mark Word 是否指向当前线程的栈帧
>
>    如果是就说明当前线程已经拥有了这个对象的锁，那就可以直接进入同步块继续执行（互斥锁的可重入性）。
>
>    否则说明当前栈帧已经指向一个对象锁，存在多个线程竞争锁，轻量级锁就要膨胀为重量级锁，锁标志的状态值变为"10”，Mark Word 中存储的就是指向重量级锁（互斥量)的指针，一旦变成重量级锁后面等待锁的线程也要进入阻塞状态。而当前线程便尝试使用**自旋**来获取锁 (不让当前线程阻塞)
>
> **解锁过程：**
>
> 1. 通过 CAS 操作尝试把线程中复制的 Displaced Mark Word 对象替换当前的 Mark Word。
> 2. 如果替换成功，整个同步过程就完成了。
> 3. 如果替换失败，说明有其他线程尝试过获取该锁（此时锁已膨张），那就要在释放锁的同时，唤醒被挂起的线程。
>
> 
>
> **重量级锁**
>
> 升级条件：若存在同一时间访问同一锁的情况，就会导致轻量级锁膨胀为重量级锁
>
> 线程阻塞



###### 重量级锁monitor

**每个对象都有一个与之关联的 monitor 锁。按照惯例，一个线程在需要独占且一致地访问一个对象的字段之前，需要先获取该对象的 monitor 锁**

[OpenJDK Hotspot源码 Github#objectMonitor](https://github.com/openjdk/jdk/blob/14408bc8f846447312fd18dde1f8c615ddad61c0/src/hotspot/share/runtime/objectMonitor.cpp#L491)

```cpp
// objectMonitor.cpp
int ObjectMonitor::TryLock(JavaThread* current) {
  void* own = owner_raw();
  if (own != nullptr) return 0;
  if (try_set_owner_from(nullptr, current) == nullptr) {  // 
    assert(_recursions == 0, "invariant");
    return 1;
  }
  return -1;
}
```

```cpp
// objectMonitor.inline.hpp
inline void* ObjectMonitor::try_set_owner_from(void* old_value, void* new_value) {
  void* prev = Atomic::cmpxchg(&_owner, old_value, new_value);  // 赋值_owner=当前线程，获得 monitor 锁
  if (prev == old_value) {
    log_trace(monitorinflation, owner)("try_set_owner_from(): mid="
                                       INTPTR_FORMAT ", prev=" INTPTR_FORMAT
                                       ", new=" INTPTR_FORMAT, p2i(this),
                                       p2i(prev), p2i(new_value));
  }
  return prev;
}
```

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_06_11_00_21.png" alt="image-20230611002147788" style="zoom:80%;" />





##### 锁优化技术

###### Adaptive Spinning 自适应自旋锁

* 自旋锁，命令参数 `--PreBlockSpin`

  * 许多情况下，共享数据的锁定状态持续时间较短，切换线程不值得

  * 通过让线程执行**忙循环**等待锁的释放，取代线程挂起不让出CPU

  * 缺点：若锁被其他线程长时间占用，会带来许多性能上的开销

* 自适应自旋锁

  * 自旋的次数不再固定

  * 由前一次在同一个锁上的自旋时间及锁的拥有者的状态来决定



###### Lock Eliminate 锁消除

是更彻底的优化

JIT (just-in-time compilation) 即时编译时，对运行上下文进行扫描，去除不可能存在竞争的锁。即 JVM 编译优化自动消除非共享资源内部的锁

```java
public void add(String str1, String str2) {
    // 由于sb只会在append方法中使用，不可能被其他线程引用，且StringBuffer 是线程安全
    // 因此sb属于不可能共享的资源，JVM会自动消除内部的锁
    StringBuffer sb = new StringBuffer();
    sb.append(str1).append(str2);
}
```



###### Lock Coarsening 锁粗化

通过扩大加锁的范围，避免反复加锁和解锁

```java
public static String copyString100Times(String target){
    StringBuffer sb = new StringBuffer();
    for (int i = 0; i<100; i++)
        sb.append(target);
    return sb.toString();
}
```



#### ReenterLock

[J.U.C并发包笔记#ReentrantLock](./J.U.C并发包.md#ReentrantLock)



### volatile 易失变量

* 内存可见性
  使该变量的线程本地缓存失效，直接访问主内存
* 禁止指令重排序
  通过内存屏障指令(该指令前后的命令顺序不变)实现

* 不保证线程安全，因为不能保证所有操作的原子性

```java
volatile int value = 0;
value++;  // 线程不安全
```



#### volatile 和 synchronized 的区别

| volatile                                               | synchronized                                               |
| ------------------------------------------------------ | ---------------------------------------------------------- |
| 本质：该变量的线程本地缓存(工作内存)失效，直接访问主存 | CAS锁-互斥锁，使得线程获取锁才可访问该变量，其他线程被阻塞 |
| volatile 仅能使用在变量级别                            | synchronized 则可以使用在变量、方法和类级别                |
| volatile 仅能实现变量的**修改可见性**，不能保证原子性  | synchronized 则可以保证变量修改的可见性和**原子性**        |
| volatile **不会造成线程的阻塞**                        | synchronized 可能会造成线程的阻塞                          |
| volatile 标记的变量不会被编译器优化                    | synchronized 标记的变量可以被编译器优化                    |
|                                                        |                                                            |



## 线程池 

目的：
提高线程的可管理性，可以统一分配、调优、监控
重用线程，降低资源消耗避免频繁的创建和销毁线程



## Executors

Executors 类提供了使用使用常用配置创建 ExecutorService、ScheduledExecutorService、ThreadFactory、Callable(没什么用) 的静态工厂方法



### `Executors.defaultThreadFactory():ThreadFactory`

`Executors.defaultThreadFactory():ThreadFactory` 返回用于创建新线程的默认线程工厂。
ThreadFactory 用于在同一个线程组中创建 Executor 使用的所有新线程。ThreadGroup 是树型的，只允许线程访问自己线程组的信息
使新线程有 ThreadGroup 相同的优先级线程
设置线程名称



### `Executors.newCachedThreadPool():ExecutorService`

`Executors.newCachedThreadPool():ExecutorService` 能提高执行**短期异步任务**的程序的性能，对 `execute()` 的调用将**重用先前构造的线程**(如果可用)。如果没有可用的现有线程，则会创建一个新线程并将其添加到池中。**六十秒**内未使用的线程将被终止并从缓存中删除(所以系统长时间闲置的时候，不会消耗什么资源)。
**本质：(0, max, 60, SQ, tfactory, hander)**

```java
// Executors.java
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}
```

```java
// ThreadPoolExecutor.java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue) {
    this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
         Executors.defaultThreadFactory(), defaultHandler);
}
```



### `Executors.newFixedThreadPool(int nThreads):ExecutorService`

`Executors.newFixedThreadPool(int nThreads):ExecutorService` 重用共享无界队列上运行的**固定数量的线程**，在任何时候最多nThreads个线程将处于活动状态，如果在所有线程都处于活动状态时提交其他任务，任务将在队列中等待，直到有线程可用。
**本质：(n, n, 0, LBQ, tfactory, hander)**



### `Executors.newScheduledThreadPool(int corePoolSize):ScheduledExecutorService`

`Executors.newScheduledThreadPool(int corePoolSize):ScheduledExecutorService` 创建一个线程池，可以安排命令在给定的延迟后运行，或定期执行。
**本质：(, max, 10ms, DWQ, tfactory, hander)**

```java
// Executors.java
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
    return new ScheduledThreadPoolExecutor(corePoolSize);
}
```

```java
// ScheduledThreadPoolExecutor.java
// public class ScheduledThreadPoolExecutor extends ThreadPoolExecutor implements ScheduledExecutorService {
public ScheduledThreadPoolExecutor(int corePoolSize) {
    super(corePoolSize, Integer.MAX_VALUE,
          DEFAULT_KEEPALIVE_MILLIS, MILLISECONDS,
          new DelayedWorkQueue());
}
```



### `Executors.newSingleThreadExecutor(): ExecutorService`

`Executors.newSingleThreadExecutor(): ExecutorService` 创建一个单线程的 executor，该执行器使用无界队列上的单个工作线程，保证任务按顺序执行，在任何给定时间不会有超过一个任务是活动的。
**本质：(1, 1, 0, LBQ, tfactory, hander)**
**区别：与 `Executors.newFixedThreadPool(1)` 不同，`Executors.newSingleThread()` 不可以重新配置以使用更多的线程**



### `Executors.newSingleThreadScheduledExecutor(): ScheduleExecutorService`

`Executors.newSingleThreadScheduledExecutor(): ScheduleExecutorService` 创建一个单线程的定期 executor，可以安排命令在给定的延迟后运行，或定期执行。保证任务按顺序执行，在任何给定时间不会有超过一个任务是活动的。
**本质：(1, max, 10ms, DWQ, tfactory, hander)**
区别：与 `Executors.newFixedThreadPool(1)` 不同，`Executors.newSingleThread()` 不可以重新配置以使用更多的线程



### `Executors.newWorkStealingPool(int parallelism): ExecutorService`

`Executors.newWorkStealingPool(int parallelism): ExecutorService` 使用可用处理器数作为**并行**等级，创建一个**工作窃取**线程池，并且可以使用多个队列来减少争用。实际线程数可能会动态增长和收缩。工作窃取池**不保证**提交**任务的执行顺序**。
本质：(availableProcessors(), defaultForkJoinWorkerThreadFactory(), null, true)

```java
// Executors.java
public static ExecutorService newWorkStealingPool(int parallelism) {
    return new ForkJoinPool
        (parallelism,
         ForkJoinPool.defaultForkJoinWorkerThreadFactory,
         null, true);
}
```

```java
// ForkJoinPool.java
public ForkJoinPool(int parallelism,
                    ForkJoinWorkerThreadFactory factory,
                    UncaughtExceptionHandler handler,
                    boolean asyncMode) {
    this(parallelism, factory, handler, asyncMode,
         0, MAX_CAP, 1, null, DEFAULT_KEEPALIVE, TimeUnit.MILLISECONDS);
}
```



### 示例

```java
class NetworkService implements Runnable {
    private final ServerSocket serverSocket;
	private final ExecutorService pool;
    class Handler implements Runnable {
	private final Socket socket;
	Handler(Socket socket) { this.socket = socket; }
	public void run() {
		// read and service request on socket
	}
	public NetworkService(int port, int poolSize) throws IOException {
		serverSocket = new ServerSocket(port);
		pool = Executors.newFixedThreadPool(poolSize);
   }
   public void run() { // run the service
		try {
			for (;;) {
				pool.execute(new Handler(serverSocket.accept()));
			}
		catch (IOException ex) {
			pool.shutdown();
}	}	}
```



## 队列

### ArrayBlockingQueue

`public class ArrayBlockingQueue<E> extends AbstractQueue<E> implements BlockingQueue<E>, java.io.Serializable {`

`ArrayBlockingQueue(int, boolean)` 构造方法

`take(): E` 队列条件判断后(等待有消息)才返回

`enqueue(E): void` 入队并通知等待

`acquire(): ` 获取锁或资源



## ThreadLocal<T>

java.lang.ThreadLocal<T>

该类提供线程局部变量，每个线程都有该变量的自己的独立初始化的副本。ThreadLocal 实例通常是与线程相关的私有静态字段(例如用户ID或事务ID)
thread-local storage(TLS): 上下文存储方案：ThreadLocal、ScopedValue、将上下数据作为方法参数显示传递、类属性或实例属性(传递this/self)



### `get():T`

`get():T` 返回线程局部变量的当前线程中副本的值



### `remove():void`

`remove():void` 删除线程局部变量的当前线程中副本的值



### `set(T):void`

`set(T):void` 将线程局部变量的当前线程中副本的值置为 T



### 示例1

```java
public class ThreadId {
	private static final AtomicInteger nextId = new AtomicInteger(0);  // 包含用于分配的下一个线程ID的原子整数
    // 包含每个线程ID的线程局部变量
	private static final ThreadLocal<Integer> threadId = new ThreadLocal<Integer>() {  
		@Override protected Integer initialValue() {
			return nextId.getAndIncrement();
		}
	};
	public static int get() {  // 返回当前线程ID
		return threadId.get();
}	}
```



### 示例2

案例：服务端存储访问的用户信息

```java
public class LocalUser {
    private static ThreadLocal<Map<String, Object>> threadLocal = new ThreadLocal<Map<String, Object>>();
    public static void set(User user, Integer scope) {
        Map<String, Object> map = new HashMap<>();
        map.put("user", user);
        map.put("scope", scope);
        LocalUser.threadLocal.set(map);
    }
    public static User getUser() {
        Map<String, Object> map = LocalUser.threadLocal.get();
        User user = (User) map.get("user");
        return user;
    }
    public static Integer getScope() {
        Map<String, Object> map = LocalUser.threadLocal.get();
        Integer scope = (Integer) map.get("scope");
        return scope;
 	}
    public static void clear() {
        LocalUser.threadLocal.remove();
}	}
```



## Random

#### nextInt(int bound)

返回 [0, bound - 1] 区间中的一个 `int`



## ThreadLocalRandom

`ThreadLocalRandom.current():ThreadLocalRandom` 返回当前线程池的 ThreadLocalRandom 对象，该对象的方法只能由当前线程调用





