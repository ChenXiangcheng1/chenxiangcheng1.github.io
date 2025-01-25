---
title: J.U.C并发包
date: 2023-09-17
update: 2023-09-17
tags:
categories:
  - Java
  - 类库
keywords: Java,J.U.C并发包
description: 关于J.U.C并发包，涉及Executor，ReenterLock、Atomic原子类型、并发集合
top_img:
cover: https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_07_10_14_22.png
---



# J.U.C 并发包

`java.util.concurrent`：提供了并发编程的解决方案，有两大核心 CAS 和 AQS

* CAS 无锁操作是 `java.util.concurrent..atomic` 包的基础
* AQS 框架是 `java.util.concurrent..locks` 包以及一些常用类比如 Semophore，ReentrantLock 等类的基础

![image-20230710142249461](https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_07_10_14_22.png)



## Callable

[Java线程#实现Callable接口](./Java线程.md#实现Callable接口)

```java
// Callable.java
@FunctionalInterface
public interface Callable<V> {
    V call() throws Exception;
}
```



## Future 接口

**用于表示异步计算的结果**，可以阻塞地获取线程的返回值



### `get():V`

`get():V` 阻塞地返回结果



### `isDone():Boolean`

`isDone():Boolean`子线程是否执行结束



### `cancel(boolean):boolean`

`cancel(boolean):boolean` 尝试取消此任务的执行，如果任务已启动则参数决定是否中断，返回任务是否被取消



### 示例

```java
ExecutorService newCachedThreadPool = Executors.newCachedThreadPool();
Future<String> future = newCachedThreadPool.submit(new MyCallable());
if(!future.isDone())  // 通过异步计算的结果future，可以阻塞地获取线程的返回值
    System.out.println("task has not finished, please wait!");
try {
    System.out.println(future.get());  // 通过 Future 对象获取任务执行结果
} catch (Exception e) {
    e.printStackTrace();
} finally {
    newCachedThreadPool.shutdown();
}
```



## 1Executor

Executor 接口执行提交的 `Runnable` 任务的对象。提供了**将任务提交与每个任务运行机制**(包括线程使用、调度等详细信息)**解耦**的方法。通常使用 Executor 而不是显式创建线程 `new Thread(new RunnableTask()).start()`



<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_06_12_18_07.png" alt="image-20230612180727299" style="zoom:67%;" />

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_07_10_23_47.png" alt="image-20230710234713495" style="zoom: 67%;" />



### `execute(Runnable): void`

`execute(Runnable): void` 执行给定的命令在将来的某个时间，该命令可以在新线程、池线程或调用线程中执行



### ExecutorService

ExecutorService 接口，提供了终止的方法、创建 Future(用于跟踪异步任务进度) 的方法



#### `submit(Runnable): Future<?>`

扩展 execute() 方法，提交任务，创建并返回 Future(用于取消执行、等待完成)，其 get() 方法返回 null 将在任务成功完成后



#### `submit(Callable<T>): Future<T>` 

扩展 execute() 方法，提交一个有返回值的任务，创建并返回 Future 表示任务待处理的结果，其 get() 方法返回任务的结果在任务成功完成后。



#### `shutdown():void`

 `shutdown():void` 开始有序关闭，禁止提交新任务，允许已提交任务执行完后终止



#### `shutdownNow():List<Runnable>`

`shutdownNow():List<Runnable>` 尝试停止所有正在执行的任务，阻止正在等待的任务启动，并返回正在等待执行的任务的列表。



#### `invokeAny(Collection<? extends Callable<T>>):T`

`invokeAny(Collection<? extends Callable<T>>):T` 执行给定的任务，等待至少一个任务完成，返回已成功完成的任务的结果，未完成的任务将被取消



#### `invokeAll(Collection<? extends Callable<T>>):List<Future<T>>` 

`invokeAll(Collection<? extends Callable<T>>):List<Future<T>>` 执行给定的任务，等待全部任务完成，当所有任务完成或超时到期时，返回保存其状态和结果的 Future 列表





##### ScheduledExecutorService

ScheduledExecutorService接口：扩展了 ExecutorService 接口，支持定期执行任务



##### ThreadPoolExecutor

[OpenJDK API 文档#ThreadPoolExecutor](https://devdocs.io/openjdk~17/java.base/java/util/concurrent/threadpoolexecutor)

ThreadPoolExecutor类：是对 AbstractExecutorService 接口的简单实现



线程池中的工作线程被包装为 Worker 对象

ThreadPoolExecutor 的核心内部类 Worker，继承了AQS 实现了不可重入的互斥锁，实现了 Runnable 接口任务，用于维护正在运行任务的线程的中断控制状态

`private final class Worker extends AbstractQueuedSynchronizer implements Runnable {`



###### 构造函数

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_06_12_18_19.png" alt="image-20230612181921052" style="zoom:67%;" />

```java
// ThreadPoolExcutor.java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue) {
    this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
         Executors.defaultThreadFactory(), defaultHandler);
}


public ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler) {
    if (corePoolSize < 0 || maximumPoolSize <= 0 || maximumPoolSize < corePoolSize || keepAliveTime < 0)
        throw new IllegalArgumentException();
    if (workQueue == null || threadFactory == null || handler == null)
        throw new NullPointerException();
    this.corePoolSize = corePoolSize;
    this.maximumPoolSize = maximumPoolSize;
    this.workQueue = workQueue;
    this.keepAliveTime = unit.toNanos(keepAliveTime);
    this.threadFactory = threadFactory;
    this.handler = handler;
}
```

* corePoolSize：**核心线程数量**，活动线程数小于核心线程数会创建新线程而不会复用线程
  CPU密集型：线程数 = 按照核数或者核数 + 1
  I/O密集型：线程数 = CPU核数 * (1 + 平均等待时间 / 平均工作时间)
* maximumPoolSize：**最大线程数**
* keepAliveTime：**保活时间**，超过核心线程数量的空闲线程会被终止
* TimeUnit：keepAliveTime 参数的时间单位
* workQueue：**任务等待队列**(阻塞队列/同步队列)，不同队列有不同排队机制。任务太多则封装成work对象进入任务等待队列。例如LinkedBlockingQueue：阻塞队列。有容量限制。当队空出队操作阻塞，当队满入队操作阻塞
  SynchronousQueue：同步队列。没有容量限制。插入元素会阻塞直到另一个线程取出该元素(异步形式，插入和取出立即返回)，同步队列在提交任务时会立即将任务交给线程池中的线程执行
  DelayedWorkQueue：按照延迟时间对元素进行排序，只有在其延迟时间到达时才能被取出，常用于实现定时任务调度
* threadFactory：指定**线程工厂**，默认为 `Executors.defaultThreadFactory()`
  [Executors.defaultThreadFactory():ThreadFactory](./Java线程.md#`Executors.defaultThreadFactory():ThreadFactory`)

* handler：**拒绝执行策略**，添加任务失败(没有更多的线程shutdown或队列槽饱和)
  * AbortPolicy：默认是中止策略，直接抛出异常
  * CallerRunsPolicy：用调用者所在的线程来执行任务
  * DiscardOldestPolicy：丢弃任务等待队列中靠最前的任务，并执行当前任务
  * DiscardPolicy：直接丢弃任务
  * 实现 RejectedExecutionHandler 接口的自定义 handler：来根据业务需求选择不同的策略



###### 线程池状态转移

```java
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));  // ctl：表示线程池的控制状态，高3位保存 runState 运行状态(提供生命周期控制，随时间递增)，低29位保存 workerCount 表示活动线程数量
```

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_09_16_01_47.png" alt="image-20230916014722306" style="zoom:67%;" />

* RUNNING - 接受新任务，处理等待队列中的任务
  shutdown() 
* SHUTDOWN - 不接受新任务，但处理等待队列中已提交的任务
  shutdownNow() 
* STOP - 不接受新任务，也不处理等待队列中已提交的任务，并中断正在执行的任务
  线程池、等待队列全空

* TIDYING(整理) - 所有任务都已终止，即workerCount为0，正在进行打扫工作
  terminated()
* TERMINATED - 钩子方法 terminated()已完成



###### `execute(Runnable):void`

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_06_12_18_16.png" alt="image-20230612181633064" style="zoom: 67%;" />

<center>同步队列在提交任务时会立即将任务交给线程池中的线程执行(与阻塞队列不同)</center>

线程池 execute() 过程：
提交任务，如果活动线程数量小于核心线程数量，则创建新线程执行任务
如果活动线程大于核心线程数量，则将任务添加到工作队列(同步队列会立即提交给线程池，阻塞队列会等核心线程空闲)
		工作队列满时，如果核心线程数量<活动线程数量<最大线程数量，则创建新线程执行任务，执行完成后空闲到保活时间，线程终止并回收
		如果活动线程数量大于最大线程数量，则走拒绝执行策略

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_06_12_18_00.png" alt="image-20230612180011776" style="zoom:80%;" />



```java
// ThreadPoolExcutor.java
// 执行给定任务在未来的某个时间
public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();       
    int c = ctl.get();  // 获取线程池控制状态ctl
    if (workerCountOf(c) < corePoolSize) {  // 1.如果活动线程数量少于核心线程数量，尝试创建一个线程以指定命令作为它第一个任务
        if (addWorker(command, true))  // 调用addWorker原子地检查runState和workerCount是否能添加活动线程，通过返回false，避免了不应该添加线程情况下的错误警报
            return;  // 如果成功添加线程，则方法返回并结束
        c = ctl.get();  // 添加线程失败(线程池被关闭或活动线程数量超过核心线程数量)，此时任务尝试进入等待队列
    }
    if (isRunning(c) && workQueue.offer(command)) {  // 2.如果线程池正在运行并且任务成功进入等待队列
        int recheck = ctl.get();  // 双重检测是否需要添加一个线程(因为可能上次检查后一个线程死亡或者线程池shutdown)
        if (!isRunning(recheck) && remove(command))  // 如果线程池不是运行状态则回滚入队，再执行拒绝执行策略
            reject(command);
        else if (workerCountOf(recheck) == 0)  // 线程池是运行状态，活动线程数量等于0，则开启一个新线程
            addWorker(null, false);  // null是开启线程
    }
    else if (!addWorker(command, false))  // 3.如果线程池不是运行状态或等待队列饱和了，则执行拒绝执行策略
        reject(command);
}
```

```java
// ThreadPoolExcutor.java
// 检查是否能添加新的活动线程，根据当前线程池状态和核心线程数量配置、最大线程数量配置，
private boolean addWorker(Runnable firstTask, boolean core) {
    retry:
    for (int c = ctl.get();;) {  // 自旋
        // 仅在必要时检查队列是否为空
        if (runStateAtLeast(c, SHUTDOWN)  // 如果STOP、TIDYING、TERMINATED直接返回false，不能添加新活动线程
            && (runStateAtLeast(c, STOP)
                || firstTask != null
                || workQueue.isEmpty()))
            return false;

        for (;;) {  // 自旋
// 如果在调用addWorker之前活动线程数量少于核心线程数量，则core=true，此时进入addWorker后反而活动线程数量超过核心线程数量，所以不能添加新的活动线程，返回false
// 如果core=false，则活动线程数量超过最大线程数量，所以不能添加新的活动线程，返回false
            if (workerCountOf(c)
                >= ((core ? corePoolSize : maximumPoolSize) & COUNT_MASK))  
                return false;
            // 活动线程数量满足要求
            if (compareAndIncrementWorkerCount(c))  // CAS操作将c自增1
                break retry;  // CAS操作成功，break retry跳出外层循环，进入下一步新增活动线程
            // CAS操作失败
            c = ctl.get();  // Re-read ctl
            if (runStateAtLeast(c, SHUTDOWN))
                continue retry;            
            // else CAS失败由于活动线程数量变化，重试内循环
        }
    }
	// 活动线程数量满足要求，线程池状态也满足要求
    boolean workerStarted = false;  // 新增活动线程是否开启
    boolean workerAdded = false;  // 新增活动线程加入等待队列
    Worker w = null;  // work类扩展了AQS
    try {
        w = new Worker(firstTask);
        final Thread t = w.thread;
        if (t != null) {
            final ReentrantLock mainLock = this.mainLock;
            mainLock.lock();
            try {
                int c = ctl.get();  // 获取锁之后首先要进行再次检查，因为可能在上次检查到这里线程池的状态改变了(ThreadFactory失败或线程池shutdown)       
                if (isRunning(c) ||
                    (runStateLessThan(c, STOP) && firstTask == null)) {  // 线程池状态RUNNING或SHUTDOWN且firstTask=null(不是新增任务请求，是已提交任务)
                    if (t.getState() != Thread.State.NEW)
                        throw new IllegalThreadStateException();
                    workers.add(w);  // 工作线程加入workers集合HashSet<Worker>即等待队列
                    workerAdded = true;
                    int s = workers.size();  // 获取工作线程数量
                    if (s > largestPoolSize)
                        largestPoolSize = s;
                }
            } finally {
                mainLock.unlock();
            }
            if (workerAdded) {  // 如果新增的工作线程已加入workers集合
                t.start();
                workerStarted = true;
            }
        }
    } finally {
        if (!workerStarted)  // 如果新增的工作线程最终没能启动成功
            addWorkerFailed(w);  // 回滚
    }
    return workerStarted;  // 返回新增活动线程是否开启
}
```



##### ForkJoinPool



###### 构造函数

```java
// ForkJoinPool.java
public ForkJoinPool(int parallelism,
                        ForkJoinWorkerThreadFactory factory,
                        UncaughtExceptionHandler handler,
                        boolean asyncMode,
                        int corePoolSize,
                        int maximumPoolSize,
                        int minimumRunnable,
                        Predicate<? super ForkJoinPool> saturate,
                        long keepAliveTime,
                        TimeUnit unit) {
        checkPermission();
        int p = parallelism;
        if (p <= 0 || p > MAX_CAP || p > maximumPoolSize || keepAliveTime <= 0L)
            throw new IllegalArgumentException();
        if (factory == null || unit == null)
            throw new NullPointerException();
        this.factory = factory;
        this.ueh = handler;
        this.saturate = saturate;
        this.keepAlive = Math.max(unit.toMillis(keepAliveTime), TIMEOUT_SLOP);
        int size = 1 << (33 - Integer.numberOfLeadingZeros(p - 1));
        int corep = Math.min(Math.max(corePoolSize, p), MAX_CAP);
        int maxSpares = Math.min(maximumPoolSize, MAX_CAP) - p;
        int minAvail = Math.min(Math.max(minimumRunnable, 0), MAX_CAP);
        this.bounds = ((minAvail - p) & SMASK) | (maxSpares << SWIDTH);
        this.mode = p | (asyncMode ? FIFO : 0);
        this.ctl = ((((long)(-corep) << TC_SHIFT) & TC_MASK) |
                    (((long)(-p)     << RC_SHIFT) & RC_MASK));
        this.registrationLock = new ReentrantLock();
        this.queues = new WorkQueue[size];
        String pid = Integer.toString(getAndAddPoolIds(1) + 1);
        this.workerNamePrefix = "ForkJoinPool-" + pid + "-worker-";
    }
```

* parallelism：并行度，默认`Runtime.availableProcessors.factory`
* ForkJoinWorkerThreadFactory：默认`defaultForkJoinWorkerThreadFactory`
* UncaughtExceptionHandler：未捕获异常处理器，执行任务时工作线程意外终止的处理策略
* asyncMode：true异步模式(任务执行顺序与提交顺序一致)，false同步模式(任务执行顺序与提交顺序不一致)
* corePoolSize
* maximumPoolSize
* minimumRunnable
* Predicate<? super ForkJoinPool>：
* keepAliveTime
* TimeUnit

![image-20230612160911813](https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_06_12_16_09.png)

并行执行，能够更好地利用多处理器

把大任务分割成若干个小任务并行执行，最终汇总每个小任务结果后得到大任务结果的框架

Work-Stealing 算法：某个线程从其他双端队列尾里窃取任务来执行 (避免子任务处理完处理器闲置)



## AbstractQueuedSynchronizer 

提供一个框架用于实现阻塞锁和依赖等待队列的同步器。

`public abstract class AbstractQueuedSynchronizer extends AbstractOwnableSynchronizer implements java.io.Serializable {`

底层：使用了 Unsafe 类、使用 `LockSupport.park()` 和 `LockSupport.unpark()` 实现线程的阻塞和唤起



**应用：**
**Java 并发构建锁和其他同步组件都是基于 AQS 框架实现的**
例如 ReentrantLock、Condition、CountDownLatch(闭锁)、FutureTask、Semaphore(信号量)、ThreadPoolExecutor 的核心内部类 Worker(用于维护正在运行任务的线程的中断控制状态) `private final class Worker extends AbstractQueuedSynchronizer implements Runnable {`



**方法：**
`getState(): int`

`setState(int): void`

`acquire()` 获取资源独占权

`release()` 释放资源



## 2显式锁 java.util.concurrent.locks

![image-20230711005959436](https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_07_11_00_59.png)

### ReentrantLock

ReentrantLock 与 monitor 锁具有相同的基本行为与语义，但扩展了些功能

ReentrantLock 较复杂了，线程池用的更多



#### `ReentrantLock(boolean)` 

`ReentrantLock(boolean)` 构造方法，true 为公平锁(不推荐)，false 为非公平锁
公平锁倾向于授予等待时间最长的线程，不会出现线程饥饿(默认策略也很少导致饥饿情况发生，并且引入公平性会有额外的开销会导致吞吐量下降)
公平锁比非公平锁慢很多
注意，锁的公平性不代表线程调度的公平性



#### `lock(): void`

`lock(): void` 获取锁。
如果锁未被其他线程持有，则获取该锁并立即返回，并将锁持有计数设置为 1
如果当前线程已经持有锁，则锁持有计数加1，并且该方法立即返回
如果锁被另一个线程持有，则当前线程将被禁用出于线程调度目的，并处于阻塞状态直到获取到锁，此时锁持有计数设置为 1



#### `unlock(): void`

`unlock(): void` 尝试释放锁
如果当前线程是该锁的持有者，则锁持有计数会减1。如果现在锁持有计数为零，则释放锁
如果当前线程不是该锁的持有者，则抛出 `IllegalMonitorStateException` 



#### `tryLock():boolean`

`tryLock():boolean` 

如果锁未被其他线程持有，则获取锁并立即返回 `true` ，将锁持有计数设置为 1。无论其他线程是否正在等待该锁(如果是公平锁会破坏公平性，使用 `tryLock(0,TimeUnit.SECONDS)` 遵守公平性)
如果当前线程已持有此锁，则立即返回 `true` ，将锁持有计数加 1。
如果锁被另一个线程持有，则立即返回 `false` 



#### `tryLock(long,TimeUnit):boolean`

`tryLock(long,TimeUnit):boolean` 如果在给定的超时时间内  没有其他线程持有锁并且当前线程没有被中断，则获取锁。

如果锁未被其他线程持有，则获取锁并立即返回值 `true` ，将锁持有计数设置为 1。如果是公平锁且有其他线程正在等待该锁，则不会获取可用锁(遵守公平性策略)
如果当前线程已持有此锁，则立即返回 `true` ，将锁持有计数加 1。
如果锁被另一个线程持有，则当前线程将出于线程调度目的而被禁用，并处于阻塞状态，直到发生以下三种情况之一：锁被当前线程获取、其他线程中断当前线程，超时时间已过。
如果指定的超时时间过去了，则返回值 `false` 。如果时间小于或等于零，则该方法根本不会等待。



#### `newCondition():Condition`

`newCondition():Condition` 返回绑定该锁的 Condition 实例



#### ReentrantLock 和 synchronized 的区别

| ReentrantLock                                 | synchronized               |
| --------------------------------------------- | -------------------------- |
| 本质：调用 Unsafe 类的 park() 方法            | CAS锁+monitor锁            |
| ReentrantLock 是类                            | synchronized 是关键字      |
| 显示锁，手动获取锁 lock()，释放锁 unlock()    | 自动锁                     |
| 可以实现公平锁                                | 非公平锁，线程看运气获取锁 |
| tryLock(long,TimeUnit) 设置超时时间，避免死锁 |                            |
| 可以获取各种锁的信息                          |                            |
|                                               | 低锁竞争情况，性能更好     |
|                                               |                            |



#### 示例

```java
public class ReentrantLockTask implements  Runnable{
    private static ReentrantLock lock = new ReentrantLock(true);  // 锁对象
    public void run(){
        while (true){
            lock.lock();
            try{
                System.out.println(Thread.currentThread().getName() + " get lock");  // 临界区do something...
                Thread.sleep(1000);
            } catch (Exception e){
                e.printStackTrace();
            } finally {
                lock.unlock();  // 即时抛出异常也要确保锁成功释放
}	}	}	}
ReentrantLockTask task = new ReentrantLockTask();
new Thread(task).start();
new Thread(task).start();  // 两个线程竞争的是同一个锁
```



### ReentrantReadWriteLock.ReadLock

共享读锁



### ReentrantReadWriteLock.WriteLock

排它写锁



### Condition 接口

**条件变量：为线程提供等待条件变量和条件变量通知的机制**

Condition 分解 Object 的 monitor 方法为不同的对象。
Lock 替换 synchronized 的使用，Condition 替换对象 monitor 方法的使用。条件变量为线程提供了等待和通知的方法。

Condition 本质上绑定一个锁，通过 `lock.newCondition()` 获取

底层：通过 AQS框架中的 ConditionObject 实现



#### `await():void`

`await():void` 使当前线程等待直到收到信号或中断



#### `signal():void`

`signal():void` 唤醒一个等待线程



#### `signalAll():void`

`signalAll():void` 唤醒所有等待线程



#### 使用步骤

1. 创建锁对象：通常使用 ReentrantLock 锁对象作为底层的锁机制
2. 创建条件对象：`Condition condition = lock.newCondition()` 创建一个与锁关联的 Condition 对象。
3. 获取锁：在进入需要同步的代码块(临界区)之前，通过 `lock.lock()` 获取锁。
4. 线程等待：在某个条件不满足时，调用 `condition.await()` 将当前线程等待，并释放锁。此时，当前线程将进入等待状态，直到其他线程调用相应的 `signal()`/`signalAll()` 方法来唤醒该线程。
5. 线程唤醒：在满足某个条件时，可以通过调用 `condition.signal()` 或者 `condition.signalAll()` 方法来唤醒一个或者所有等待的线程。
6. 释放锁：在唤醒线程后，通过 `lock.unlock()` 释放锁，以允许其他线程获取锁并执行相应的操作。



#### 示例

使用两个 Condition 对象 notEmpty、notFull 作为同步信号量

```java
class BoundedBuffer<E> {
	final Lock lock = new ReentrantLock();
	final Condition notFull  = lock.newCondition(); 
    final Condition notEmpty = lock.newCondition(); 
    final Object[] items = new Object[100];
    int putptr, takeptr, count;
    public void put(E x) throws InterruptedException {
		lock.lock();
		try {
			while (count == items.length)
				notFull.await();
				items[putptr] = x;
				if (++putptr == items.length) putptr = 0;
				++count;
				notEmpty.signal();
		} finally {
			lock.unlock();
	}	}
	public E take() throws InterruptedException {
		lock.lock();
		try {
			while (count == 0)
         	notEmpty.await();
       		E x = (E) items[takeptr];
       		if (++takeptr == items.length) takeptr = 0;
       		--count;
       		notFull.signal();
       		return x;
     	} finally {
       		lock.unlock();
}   }  	}
```



## 3原子类型 java.util.concurrent.atomic

原子基础类型、原子引用、原子数组，提供了对应的原子方法
底层：CAS 操作开销比锁少	[Java线程笔记#CAS](./Java线程.md#CAS)

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_07_11_01_28.png" alt="image-20230711012805827" style="zoom:67%;" />

**应用场景：**

原子类型变量适用于简单的原子操作(该类型对应的原子方法)
如果业务逻辑复杂，还是需要使用锁机制或其他线程同步机制，如 synchronized 或 ReentrantLock 等



### AtomicInteger

原子整数

**方法：**

`getAndIncrement(): int` 线程安全的自增

```java
@IntrinsicCandidate (内在候选)
public final int getAndAddInt(Object o, long offset, int delta) {
    int v;
    do {
        v = getIntVolatile(o, offset);
    } while (!weakCompareAndSetInt(o, offset, v, v + delta));  // CAS操作，发现冲突则自旋重试
    return v;
}
```



**示例：**

```java
AtomicInteger count = new AtomicInteger(0);
count.incrementAndGet();  // 线程同步的递增操作
```



### AtomicStampedReference 



### AtomicIntegerArray



## 4并发工具类 tools

![image-20230710211146619](https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_07_10_21_11.png)

4个同步器：用于协助线程同步，闭锁、栅栏、信号量、交换器。



### Executors 工厂方法模式

[工厂方法模式笔记#]()

[Java线程笔记#Executors]()



### CountDownLatch 闭锁 

**作用：**

让主线程等待一组事件发生后继续执行，即主线程在执行一组`countDown()`方法后执行

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_07_10_15_42.png" alt="image-20230710154215708" style="zoom:67%;" />

**方法：**

`CountDownLatch(int)` 构造方法

`await():void` 当前线程等待，直到锁存器(latch)递减为零，除非线程被中断

`countDown()void` 递减锁存器的计数，如果计数达到零，则释放所有等待线程。



**例子**

```java
public class CountDownLatchDemo {
    private void go() throws InterruptedException {
        CountDownLatch countDownLatch = new CountDownLatch(3);
        // 依次创建3个线程，并启动
        new Thread(new Task(countDownLatch), "Thread1").start();
        Thread.sleep(1000);
        new Thread(new Task(countDownLatch), "Thread2").start();
        Thread.sleep(1000);
        new Thread(new Task(countDownLatch), "Thread3").start();
        countDownLatch.await();  // 当前线程等待
        System.out.println("所有线程已到达，主线程开始执行" + System.currentTimeMillis());
    }
    @AllArgsConstructor
    class Task implements Runnable {
        private CountDownLatch countDownLatch;
        public void run() {
            System.out.println("线程" + Thread.currentThread().getName() + "已到达" + System.currentTimeMillis());
            countDownLatch.countDown();
    }	}
    public static void main(String[] args) throws InterruptedException {
        new CountDownLatchDemo().go();
}	}
```

```运行
线程Thread1已到达1688974327600
线程Thread2已到达1688974328605
线程Thread3已到达1688974329610
所有线程已到达，主线程开始执行1688974329611
```



### CyclicBarrier 栅栏 

**作用：**

阻塞当前线程，等待其他线程

* 等待其它线程且会阻塞自己当前线程，所有线程必须同时到达栅栏位置后，才能继续执行；
* 所有线程到达栅栏处，可以触发执行另外一个预先设置的线程

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_07_10_16_34.png" alt="image-20230710163413648" style="zoom:67%;" />

**方法：**

`CyclicBarrier(int)` 构造方法

`await():int`等待所有线程执行`await()`，返回当前线程的到达索引



**例子：**

```java
public class CyclicBarrierDemo {
    private void go() throws InterruptedException {
        CyclicBarrier cyclicBarrier = new CyclicBarrier(3);
        new Thread(new Task(cyclicBarrier), "Thread1").start();
        Thread.sleep(1000);
        new Thread(new Task(cyclicBarrier), "Thread2").start();
        Thread.sleep(1000);
        new Thread(new Task(cyclicBarrier), "Thread3").start();
    }
    @AllArgsConstructor
    class Task implements Runnable {
        private CyclicBarrier cyclicBarrier;
        public void run() {
            System.out.println("线程" + Thread.currentThread().getName() + "已到达" + System.currentTimeMillis());
            try {
                cyclicBarrier.await();
            } catch (InterruptedException | BrokenBarrierException e) {
                e.printStackTrace();
            }
            System.out.println("线程" + Thread.currentThread().getName() + "开始处理" + System.currentTimeMillis());
    }	}
    public static void main(String[] args) throws InterruptedException {
        new CyclicBarrierDemo().go();
}	}
```

```运行
线程Thread1已到达1688977442928
线程Thread2已到达1688977443930
线程Thread3已到达1688977444936
线程Thread1开始处理1688977444937
线程Thread2开始处理1688977444937
线程Thread3开始处理1688977444937
```





### Semaphore 信号量

**作用：**

控制某个资源可被同时访问的线程个数

![image-20230710175343332](https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_07_10_17_53.png)

**方法：**

`Semaphore(int)` 构造方法

`acquire(): void` 从信号量获取许可(permit)，若没有可用许可，则线程阻塞直到有可用许可

`release()` 释放许可并将其返回到信号量



**例子：**

```java
public class SemaphoreDemo {
    public static void main(String[] args) {        
        ExecutorService exec = Executors.newCachedThreadPool();  // 线程池        
        final Semaphore semp = new Semaphore(3);  // 只能3个线程同时访问      
        for (int index = 0; index < 6; index++) {  // 模拟6个客户端访问
            final int NO = index;
            Runnable run = new Runnable() {
                public void run() {
                    try {                        
                        semp.acquire();  // 获取许可
                        System.out.println(System.nanoTime() + "Accessing: " + NO);
                        Thread.sleep((long) (Math.random() * 10000));
                        
                        semp.release();  // 访问完后，释放
                        System.out.println(System.nanoTime() + "-------------" + semp.availablePermits());
                    } catch (InterruptedException e) {
                        e.printStackTrace();
            };	}	}
            exec.execute(run);
        }        
        exec.shutdown();  // 退出线程池
}	}
```

```运行
676727537410300Accessing: 1
676727537406700Accessing: 0
676727537534400Accessing: 2
676731068462200Accessing: 5  // 在获取许可
676731068377500-----------------1  // 先释放许可
676733191022300-----------------1
676733191051200Accessing: 4
676735006998700-----------------1
676735007069400Accessing: 3
676736635466300-----------------1
676737401446300-----------------2
676741602430300-----------------3
```



### Exchanger 交换器 

**作用：**

两个线程到达同步点后，相互交换数据

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_07_10_17_51.png" alt="image-20230710175144096" style="zoom:80%;" />

**方法：**

`Exchanger()` 构造方法

`exchange(V): V` 等待另一个线程到达此交换点（除非当前线程被中断），然后将给定的对象传输给它，并接收其对象作为返回。



**例子：**

```java
public class ExchangerDemo {
    public static void main(String[] args) {
        Exchanger<String> exchanger = new Exchanger<>();
        ExecutorService service = Executors.newFixedThreadPool(2);  // 只有两个线程的固定线程池
        service.execute(() -> {  // 代表女生         
            try {
                girl = exchanger.exchange("我其实暗恋你很久了.....");
                System.out.println("女生说：" + girl);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        });
        service.execute(() -> {  // 代表男生
            System.out.println("女生慢慢的从教室里面走出来....");
            try {
                TimeUnit.SECONDS.sleep(3);                
                String boy = exchanger.exchange("我很喜欢你.....");  // 男生对女生说的话
                System.out.println("男生说：" + boy);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
}	}	});	}
```

```运行
女生慢慢的从教室里面走出来....
男生说：我其实暗恋你很久了.....
女生说：我很喜欢你.....  // 两线程成功交换数据
```



## 5并发集合 concurrent

java.util.concurrent 提供线程安全的 Java 容器

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_07_11_02_01.png" alt="image-20230711020134231" style="zoom:80%;" />

### BlockingQueue 接口

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_07_11_02_02.png" alt="image-20230711020207896" style="zoom:60%;" />

**作用：**

提供可阻塞的入队和出队操作。主要用于生产者-消费者模式，在多线程场景时生产者线程在队列尾部添加元素，而消费者线程则在队列头部消费元素，通过这种方式能够达到将任务的生产和消费进行隔离的目的



#### 实例方法

`add(E):boolean` 向队列尾部添加元素，添加成功返回 true，添加失败抛出 IllegalStateException 

`offer(E):boolean` 向队列尾部添加元素，添加成功返回 true，添加失败返回 false

`offer(E, long, TimeUnit):boolean` 向队列尾部添加元素，若队列满则最多等待timeout指定时间，若等待过程被中断则抛出InterruptedException，若时间到期则返回 false

`poll(long, TimeUnit):E` 从队列头部取出元素，若队列为空则最多等待timeout指定时间，若等待过程被中断则抛出InterruptedException

`put(E):boolean` 向队列尾部添加元素，若队列满阻塞直到添加成功

`take(): E` 从队列头部取出元素，若队列为空则阻塞直到队列有元素

`remainingCapacity()` 获取当前队列剩余可存储元素的数量

`remove(Object): boolean` 从队列中移除指定的对象

`contains(Object): boolean` 判断队列中是否存在指定的对象

`drainTo(Collection<? super E>): int` 将队列中的元素转移到指定的集合当中



#### 接口实现

todo:闲的话可以看看阻塞队列123的实现

1. **ArrayBlockingQueue**：**数组阻塞队列**，一个由**数组**结构组成的**有界(初始化时需指定容量大小)阻塞队列**。
   通过 ReentrantLock、Condition 实现 (使用了两个 Condition 对象 notEmpty、notFull，作为同步信号量) 
2. **LinkedBlockingQueue**：**链表阻塞队列**，一个由链表结构组成的有界/无界阻塞队列；
3. **PriorityBlockingQueue**：**优先阻塞队列**，一个支持优先级排序的无界阻塞队列；
4. DealyQueue：一个使用优先级队列实现的无界阻塞队列。数据中的元素必须实现`Delayed`接口，只有延迟期满才能从队列中获取元素；
5. SynchronousQueue：一个不存储元素的阻塞队列。队列只能容纳一个元素；
6. LinkedTransferQueue：一个由链表结构组成的无界阻塞队列。=SynchronousQueue+LinkedBlockingQueue，所以 CAS 无锁操作性能比 LinkedBlockingQueue 高，又比 SynchronousQueue 能存储更多的元素；
7. LinkedBlockingDeque：一个由链表结构组成的双向阻塞队列。
   双端队列与 Executor 具体实现的 fork/join 框架中 work-stealing 机制相关，
   即每个工作者都有一个自己的双端队列，当工作者完成自己双端队列的工作后可以窃取其他工作者双端队列末尾的任务，
   因为工作者线程不会竞争一个共享的任务队列，所以窃取工作者模式比传统的生产消费者模式有更好的可伸缩性，大多时候不会窃取和从尾部窃取可以减少竞争；



### ConcurrentMap 接口

这部分笔记去看 Collection_map集合体系.md 笔记



