---
title: JVM
date: 2023-07-13
update: 2023-10-31

tags:
categories:
  - Java
  - VM
keywords: Java,JVM,GC垃圾回收
description: 主要是JVM架构、GC机制，还介绍了Java工具
top_img:
cover: https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_05_13_14_00.png
---

# JVM主要考点

[Java Language and Virtual Machine Specifications (Java 语言和虚拟机规范)](https://docs.oracle.com/javase/specs/)



JVM：

* HotSpot：由OpenJDK维护

* OpenJ9：[官网](https://eclipse.dev/openj9/)	|	[github](https://github.com/eclipse-openj9/openj9)

* Azul Zing(收费)



## 1 HotSpot VM 架构 

JVM是内存中的虚拟机主要由 Class Loader 类加载器、Runtime Data Area 运行时数据区、Execution Engine 执行引擎、Native Interface 本地接口四部分构成，主要通过 Class Loader 将符合格式要求的class文件加载到内存，并通过 Execution Engine 解析里面的字节码。

![image-20230513140058796](https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_05_13_14_00.png)



* **Class Loader**：类加载器，将符合格式要求的 class 字节码文件加载到内存
* **Execution Engine**：执行引擎，字节码解释成机械指令码
* **Native Interface**：本地方法接口，融合不同开发语言的原生库为Java所用 (例把C/C++编译成了DLL)，通过 native 本地方法调用，例如 Class.forname 就调用了原生方法 forName0
  
* **Runtime Data Area**：运行时数据区，Java 内存空间结构模型



### 类从编译到执行的过程

1. 编译器将 ClassName.java 源文件**编译**为 ClassName.class 字节码文件
2. **ClassLoader** 将字节码转换为 **JVM 中的 Class\<ClassName>对象** (类的装载过程：加载、链接、初始化) 类文件常量池
3. JVM 利用 Class\<ClassName> 对象**实例化**为 ClassName 对象

执行过程：编译时、字节码加载时、运行时 (=动态)



### Native 原生方法

可以通过 [github openJDK 源码](https://github.com/openjdk/jdk/tree/master/src/java.base/share/native/libjava) 查看



## 反射

看 [Java反射机制](../Java语言特性/Java反射机制.md)



## 执行引擎

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting2/img/2023_1027_194939.png" alt="image-20231027194938982" style="zoom: 45%;" />

JIT (Just-In-Time)，将程序源代码(热点代码)在**运行时编译**成机械码并缓存，提高了运行效率，但启动速度更慢

AOT (Ahead-Of-Time)，将程序源代码**提前编译**成可执行文件，提高了启动速度和运行效率，但编译时间更长



## 2 ClassLoader

ClassLoader 是 Java 的核心组件，它主要工作在 Class 装载的加载阶段，负责从系统外部获得 **Class 二进制数据流**，所有的 Class 都是由ClassLoader 进行加载的，然后交给 Java 虚拟机进行连接、初始化等操作。



### 层次结构

Java17 的 ClassLoader 种类：

**Bootstrap ClassLoader** 引导类加载器：是 JVM 的一部分，负责加载核心库 (java.\*)
**Platform ClassLoader** 平台类加载器：是 JVM 的一部分， 负责加载扩展库 (Java SE platform APIs)
	**Application ClassLoader** 应用类加载器：JVM内置，负责加载应用程序的类 (默认类加载器)。
		**自定义 ClassLoader**：能实现特定的类加载需求。



#### 自定义 ClassLoader 可使用字节码增强技术

自定义 ClassLoader：

1. 继承 ClassLoader 抽象类

2. 重写 `findClass()` 方法返回 Class\<ClassName> 对象

3. 自定义找到 .class 字节码文件读出，按字节流加载

4. 调用 `defineClass()` 方法解析字节数组来生成 Class\<ClassName> 实例，并加载到 JVM。返回 Class\<ClassName> 对象



调用：

1. new 自定义 ClassLoader
2. 调用 `loadClass(“ClassName”)` 方法，`loadClass()` 方法会掉用自定义 `findClass()` 方法，返回 Class\<ClassName> 对象



应用场景：

​	通过对字节数组操作（**字节码增强技术**ASM）可以**访问远程类**、对**敏感信息加密**。
​	实现AOP、Cglib



例子：

```java
@AllArgsConstructor
public class MyClassLoader extends ClassLoader {
    private String path;
    private String classLoaderName;
    @Override
    public Class findClass(String name) {  // 用于寻找类文件
        byte[] b = loadClassData(name);
        return defineClass("com.interview.javabasic.reflect.Wali", b, 0, b.length);
        // return defineClass(name, b, 0, b.length);
    }
    private byte[] loadClassData(String name) {  //用于加载类文件
        // name = name.split("\\.")[name.split("\\.").length];
        name = path + name + ".class";
        InputStream in = null;
        ByteArrayOutputStream out = null;
        try {
            in = new FileInputStream(new File(name));
            out = new ByteArrayOutputStream();
            int i = 0;
            while ((i = in.read()) != -1) {
                out.write(i);
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                if (in != null)
                    in.close();
                if (out != null)
                    out.close();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
        assert out != null;
        return out.toByteArray();        
    }
    public static void main(String[] args) throws ClassNotFoundException, IllegalAccessException, InstantiationException {
        MyClassLoader m = new MyClassLoader("F:\\dev-projects\\IdeaProjects\\javabasic\\src\\com\\interview\\javabasic\\reflect\\", "myClassLoader");
        Class c = m.loadClass("Wali");
        // Class c = m.loadClass("com.interview.javabasic.reflect.Wali");
        System.out.println(c.getClassLoader());
        System.out.println(c.getClassLoader().getParent());
        System.out.println(c.getClassLoader().getParent().getParent());
        System.out.println(c.getClassLoader().getParent().getParent().getParent());
}	}
```

```运行结果
com.interview.javabasic.reflect.MyClassLoader@30dae81
jdk.internal.loader.ClassLoaders$AppClassLoader@e9e54c2
jdk.internal.loader.ClassLoaders$PlatformClassLoader@7106e68e
null
```



### 双亲委派机制

**ClassLoader 在加载类时会委派给父类加载器尝试加载**。

双亲委派模型的作用：逐层查找，避免多份同样的字节码的加载



<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_05_16_19_42.png" alt="image-20230516194228426" style="zoom: 50%;" />



### 类的装载过程

[Java 语言规范(中文翻译)#执行](https://github.com/feedfor/jls-chinese/blob/master/12-%E6%89%A7%E8%A1%8C.md)

我们约定用装载表示 Class 对象的生成过程，而加载是其中的一个部分

1. **加载** `loadClass()`：通过 ClassLoader 的 `loadClass()` 方法加载 .class 文件字节码到 JVM 内存中，并将静态数据转化为运行时方法区中的类结构数据，在运行时**数据区**中生成代表这个类的 **Java.lang.Class 对象**作为**方法区 (MetaSpace元空间)**这个**类结构数据**的访问入口

2. **链接** `resolveClass()`：
   	**校验**：检查加载的 .class 文件的正确性和安全性
   	**准备**：为 **static 类变量 **(static 字段、static 常量) **分配存储空间**并设置类变量初始值，类变量随类型信息存放在方法区(MetaSpace 元空间)中，static 变量生命周期很长使用不当容易造成内存泄漏
   	**解析符号引用**(可选)：JVM 将常量池内的**符号引用转换为直接引用** (符号引用是类的**全限定名**、**方法名**等，需要先解析再定位到目标)

3. **初始化**：执行父类初始化、类变量初始化和执行静态代码块



```java
// ClassLoader.java
protected Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException {
    synchronized (getClassLoadingLock(name)) {
        // First, check if the class has already been loaded
        Class<?> c = findLoadedClass(name);
        if (c == null) {
            long t0 = System.nanoTime();
            try {
                if (parent != null) {
                    c = parent.loadClass(name, false);
                } else {
                    c = findBootstrapClassOrNull(name);
                }
            } catch (ClassNotFoundException e) {
                // ClassNotFoundException thrown if class not found
                // from the non-null parent class loader
            }
            if (c == null) {
                // If still not found, then invoke findClass in order
                // to find the class.
                long t1 = System.nanoTime();
                c = findClass(name);  // 调用自定义ClassLoader的findClass()方法
                // this is the defining class loader; record the stats
                PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                PerfCounter.getFindClasses().increment();
            }
        }
        if (resolve)
            resolveClass(c);
        return c;
}	}
```



#### 显示装载和隐式装载

隐式装载：new (**在GC堆中**分配了内存空间，**创建实例**)

显式装载：ClassLoader.loadClass (**没有初始化**)、Class.forName (**初始化完成**)等



##### loadClass 和 forName 的区别

* `Class.forName()` 得到的 class，是已经初始化完成的

* `Classloder.loadClass()` 得到的 class，是还没有初始化的

例子：

* Mysql5 驱动 com.mysql.jdbc.Driver，因为Driver中有静态代码段需要执行，所以需要使用 Class.forName 加载

* 在 Spring IOC 中使用 lazyload 延迟加载技术，即在资源加载器读入 bean 的配置文件时，如果是以 Classpath 的方式就通过 Classloder.loadClass 加载。



## 3 Java内存区域划分

C语言内存结构模型：数据段(=堆+栈+静态数据区) + 代码段

JDK8 内存结构模型：分为**线程独占部分**和**线程共享部分**

每一个进程对应一个 JVM 实例，对应一个堆

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_05_17_14_12.png" alt="image-20230517141200811" style="zoom: 40%;" />



### 线程私有部分 (本地内存)

属于线程私有数据区域，对其他线程不可见，不存在线程安全问题



**PC**

* 是逻辑计数器，存字节码指令的内存地址

* 对 Java 方法计数，如果是 Native 方法则计数器值为 Undefined



#### 虚拟机栈

Java方法执行时的内存模型，包含多个栈帧，存储了当前方法的所有本地变量信息



**栈帧**

* **栈帧 = 局部变量表 + 操作数栈 + 动态链接 + 返回地址**
  局部变量表：包含方法执丸行过程中的所有变量，为操作数栈提供数据支撑
  操作数栈：入栈、出栈、复制、交换所产生的消费变量

* **方法执行结束自动释放栈帧，不需要 GC 回收**

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_05_17_14_19.png" alt="image-20230517141924805" style="zoom: 40%;" />

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_05_17_14_45.png" alt="image-20230517144531900" style="zoom:40%;" />

**本地方法栈**

与虚拟机栈相似，存储本地方法的信息



##### SOF 栈溢出异常、OOM 内存溢出异常

* 递归深度太深，会引发 java.lang.StackOverflowError 异常，栈溢出

* 虚拟机栈过多 (线程过多) 会引发 java.lang.OutOfMemoryError 异常，内存不够

```java
public void stackLeakByThread()(
	while(true){
		new Thread(){
			public void run(){
				while(true) {}                
			}
		}.start();
}	}
```



### 线程共享部分 (主内存)

属于数据共享的区域，多线程并发操作时会引发线程安全问题

主内存共享的方式是：线程各拷贝一份数据到工作内存(线程独占)，操作完成后刷新回主内存



#### 元空间 使用本地内存

**方法区**是 JMM 规范中定义的一种内存区域，**存储类的元信息**
具体实现有：

* 永久代(PermGen)(Java[7-)，使用 JVM 内存固定分配 `-XX:MaxPermSize`
* **元空间(MetaSpace)(Java[8+)，使用操作系统的本地内存进行动态分配**

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_09_10_17_18.png" alt="image-20230910171807627" style="zoom:50%;" />

MetaSpace 相比 PermGen 的优势

* 字符串常量池存在永久代(JVM)中，容易出现性能问题和内存溢出

* 类和方法的信息大小难易确定，给永久代的大小指定带来困难 (元空间动态分配，不用指定初始化大小)

* 永久代会为 GC 带来不必要的复杂性，需要特别处理回收效率低 (元空间使用本地内存，不会因空间不足而触发 Full GC)

* 元空间方便 HotSpot VM 与其他 VM 如 rockit 的集成

| PermGen的缺点                                             | MetaSpace的优势                                  |
| --------------------------------------------------------- | ------------------------------------------------ |
| 字符串常量池存在永久代(JVM)中，容易出现性能问题和内存溢出 |                                                  |
| 类和方法的信息大小难易确定，给永久代的大小指定带来困难    | 元空间动态分配，不用指定初始化大小               |
| 永久代会为 GC 带来不必要的复杂性，需要特别处理回收效率低  | 元空间使用本地内存，不会因空间不足而触发 Full GC |
|                                                           | 元空间方便 HotSpot VM 与其他 VM 如 rockit 的集成 |

类文件常量池：

字符串常量池：

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_09_11_20_19.png" alt="image-20230911201951346" style="zoom: 65%;" />

运行时常量池：

浮点数常量：3.14
布尔常量：true、false
整型常量：范围为[-128, 128)
类和接口的全限定名
符号引用：类或接口的名称、方法的名称和描述符、字段的名称和描述符

```java
Integer a=128;
Integer b=128;
System.out.println(a==b);
Integer a=-128;
Integer b=128;
```

```运行
false
true
```

基本数据的包装类型存储在 GC 堆



#### GC堆

* **常量池** (=静态字段 + 符号引用(=类全限定名+方法名) )
* **实例对象的分配区域** (数组+类对象)

堆是 GC 管理的主要区域，因此也被称为 GC 堆

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_05_17_15_46.png" alt="image-20230517154645449" style="zoom: 67%;" />



##### GC 堆内存区域的划分(重点)

* **年轻代**：是用于存放新创建的对象的区域
  * **Eden 区8**：
  * **两个 Survivor 区**：分为 **From 区1**、**To 区1**
* **老年代20**：是用于存放长时间存活的对象的区域。在经过多次 Minor GC 后，仍然存活的对象会被晋升到老年代。

<table>
    <tr>
    	<td><center><img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_05_19_18_34.png" alt="image-20230519183404821" style="zoom: 50%;" /></center></td>
    	<td><center><img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_05_19_18_35.png" alt="image-20230519183415831" style="zoom: 40%;" /><img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_05_19_19_08.png" alt="image-20230519190841976" style="zoom:80%;" /></center></td>
    </tr>
    <tr>
    	<td><center/>JDK[7-</center></td>
    	<td><center/>JDK[8+</center></td>
    </tr>
</table>



### 常考题解析

#### 三大性能调优参数

`java -Xms128m -Xmx128m -Xss256k -jar xxxx.jar`

-Xss：规定了每个线程虚拟机栈（堆栈）的大小，常 256k
-Xms：堆的初始值
-Xmx：堆能达到的最大值
通常 Xms 和 Xmx 设置成一样的，避免内存抖动



#### 堆和栈的区别

内存分配策略：

静态存储：**编译时**确定每个数据目标在运行时的存储空间需求，要求程序代码中没有可变数据结构，没有嵌套和递归代码
栈式存储：数据区需求在编译时未知，**运行时**进入 module 模块入口前需要能确定运行时的存储空间需求，动态分配
堆式存储：编译时或运行时模块入口都无法确定，**动态分配**



**联系：**

堆和栈都是 JMM 规范中定义的概念。

**GC堆 = 常量池 + 存储实例对象**
**栈 = 存储地址**，指向对应的堆中的实例对象

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_05_17_17_12.png" alt="image-20230517171251767" style="zoom: 60%;" />

##### 区别 (重点)：

|          | 栈                                       | 堆               |
| -------- | ---------------------------------------- | ---------------- |
| 线程     | 线程私有                                 | 线程共享         |
| 管理方式 | 自动释放                                 | 需要 GC 垃圾回收 |
| 空间大小 | 栈比堆小，使用 JVM 内存                  | 使用 OS 内存     |
| 碎片相关 | 栈产生的碎片远远小于堆，因为堆空间活动多 |                  |
| 分配方式 | 支持静态分配和动态分配                   | 仅支持动态分配   |
| 效率     | 栈的使用效率比堆高                       |                  |
|          |                                          |                  |



##### 栈、堆、元空间(内存角度)

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_05_17_17_06.png" alt="image-20230517170601579" style="zoom:60%;" />



## 4 GC 机制

### GC堆中标记强引用垃圾

无论引用计数算法还是可达性分析算法都是基于强引用而言的

目前大多数 JVM 采用可达性分析算法



#### 引用计数法

通过判断对象的引用数量来决定对象是否可以被回收

* 堆中每个对象实例都有一个引用计数器，被引用则+1，完成引用(生命周期结束、新值)则-1
* 任何引用计数为0的对象实例可以被当作垃圾收集

优点：执行效率高，程序执行受影响较小，几乎不打断程序的执行，适合实时环境
**缺点：**无法检测出**循环引用，会导致内存泄露**



#### 可达性分析

Reachability analysis，通过判断对象的引用链是否可达来决定对象是否可以被回收

可以作为 GC Root 的对象：

* 虚拟机**栈中的引用对象**（栈帧中的本地变量表，局部变量）
* 方法区中的常量引用的对象（常量）
* 方法区中的类静态属性引用的对象
* 本地方法栈中 JNI (Native方法) 的引用对象
* **活跃线程**的引用对象
* **不包括 GC 堆里**面的对象指针

RootSet = 全局根+线程根

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_09_10_01_48.png" alt="image-20230910014801040" style="zoom: 67%;" />



#### 方法区标记垃圾

垃圾：废弃常量(可达性分析)、无用的类(条件：该类所有实例已被回收、对应的ClassLoader已被回收、无Class引用)



### 回收垃圾的4种算法

标记-清除算法、标记-整理算法、复制算法、分代搜集算法(主流)

<table>
    <tr>
    	<td width=50%>
        	<p>
<center><strong>标记-清除算法</strong></center><br/>
Tracing 跟踪 Collector 收集器<br/>
&emsp;&emsp;1. Mark 标记阶段：使用可达性分析算法，从 GCRoot 根集合进行扫描，对存活的对象进行标记<br/>
&emsp;&emsp;2. Sweep 清除阶段：对堆内存从头到尾进行线性遍历，回收不可达对象的内存<br/>
&emsp;&emsp;优点：不需要对对象进行移动，仅对不存活的对象处理。在存活对象比较多时高效<br/>
&emsp;&emsp;缺点：需要使用一个空闲列表记录有所的空闲区域与大小，可用内存碎片化<br/>
<strong>标记阶段完成后，直接清理</strong><br/><br/>
            </p>
        </td>
        <td  width=50>
        	<p>
<center><strong>标记-整理算法</strong></center><br/>
Compacting 压缩 Collector 收集器<br/>
&emsp;&emsp;1. Mark 标记阶段：使用可达性分析算法，从 GCRoot 根集合进行扫描，对存活的对象进行标记<br/>
&emsp;&emsp;2. Sweep 清除阶段：移动所有存活的对象，且按照内存地址次序依次排列，然后将末端内存地址以后的内存全部回收<br/>
&emsp;&emsp;优点：新对象的分配只需要通过指针碰撞就可以完成，空闲区域始终可知也不会再有碎片化问题<br/>
&emsp;&emsp;缺点：GC 暂停的时间会增长，因为需要将对象拷贝到新的地方还需要更新它们的引用地址<br/>
场景：用于老年代 (对象存活率高的场景)<br/>
<strong>标记阶段完成后，移动后清理</strong><br/>    
            </p>
        </td>
    </tr>
    <tr>
    	<td><center><img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_09_10_01_53.png" alt="image-20230910015302181" style="zoom:100%;" /></center></td>
       	<td><center><img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_09_10_02_00.png" alt="image-20230910020038800" style="zoom:80%;" /></center></td>
    </tr>
</table>
<table>
    <tr>
    	<p>
<strong>标记-复制算法</strong><br/>
&emsp;&emsp;1. 标记存活对象<br/>
&emsp;&emsp;2. 复制 Eden 区存活对象<br/>
&emsp;&emsp;3. 清空 Eden 区<br/>
&emsp;&emsp;4. 交换 Survivor 区<br/>
&emsp;&emsp;5. 年龄计算<br/>
<br/>
复制算法 Copying，内存分为对象面和空闲面<br/>
&emsp;&emsp;1. 对象在对象面上创建<br/>
&emsp;&emsp;2. 将存活的对象从对象面复制到空闲面<br/>
&emsp;&emsp;3. 回收对象面的不可达对象内存<br/>
优点：两个内存区域交替使用，<strong>解决了可用内存碎片化问题</strong>(顺序分配内存只需移动堆顶指针)，多区域不用统一回收可以减少STW<br/>
缺点：浪费内存<br/>
场景：用于年轻代(对象存活率低的场景)<br/>        
        </p>
    </tr>
    <tr>
    	<td width=33%><center><img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_05_19_18_48.png" alt="image-20230519184847441" style="zoom: 67%;" /></center></td>
    	<td width=33%><center><img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_05_19_18_49.png" alt="image-20230519184901206" style="zoom: 50%;" /></center></td>
        <td width=33%><img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_05_19_18_22.png" alt="image-20230519182219841" style="zoom: 50%;" /></td>
    </tr>
</table>




### 常见的垃圾收集器

[指定JVM指定版本的垃圾收集器的可用性](https://chriswhocodes.com/gc-explorer.html)

指标：低延迟(STW时间尽可能短)、高吞吐(回收内存尽可能快)

不同的垃圾收集器，表示不同的回收策略。

目前现状：GC 还在飞速发展... (JDK11推出 Epsilon GC(用于测试) 和 ZGC、JDK15推出Shenandoah、JDK21推出分代ZGC)

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_05_20_19_53.png" alt="image-20230520195351495" style="zoom: 50%;" />

| JDK            | 年轻代的垃圾收集器 | 启动参数                         | 回收垃圾的算法    | 线程   | 应用线程与GC线程 | 使用场景                                                     |
| -------------- | ------------------ | -------------------------------- | ----------------- | ------ | ---------------- | ------------------------------------------------------------ |
|                | Serial GC          | `-XX:+UseSerialGC`               | 标记-复制算法     | 单线程 |                  | 适用于小型或客户端应用，目标是最大化响应时间                 |
|                | Serial Old         | `-XX:+UseSerialOldGC`            | 标记-整理算法     | 单线程 |                  | 适用于小型或客户端应用。                                     |
|                | ParNew GC          | `-XX:+UseParNewGC`               | 标记-复制算法     | 多线程 |                  | 是 Serial GC 的多线程版本，适用于多核服务器，目标是提高吞吐量 |
| (5默认)        | Parallel Scavenge  | `-XX:+UseParallelGC`             | 标记-复制算法     | 多线程 |                  | 适用于注重系统吞吐量而对响应时间要求相对较低的应用场景       |
| (5默认)        | Parallel Old       | `-XX:+UseParallelOldGC`          | 标记-整理算法     | 多线程 |                  | 适用于多核服务器，目标是提**高吞吐量**                       |
|                | ~~CMS~~            | `-XX:+UseConcMarkSweepGC`        | 并发标记-清除算法 | 多线程 | 并发             | 适用于追求低延迟的应用，但可能产生较多的碎片。**内存小低延迟** |
| 7 (9默认)      | G1                 | `-XX:+UseG1GC`                   | 标记-整理算法     | 多线程 | 并发             | 适用于大内存的应用，目标是在保持吞吐量的同时降低停顿时间。**内存大低延时** |
| 11(21支持分代) | 分代ZGC            | `-XX:+UseZGC -XX:+ZGenerational` | 标记-整理算法     | 多线程 | 并发             | 低延迟                                                       |

`-XX:+UseAdaptiveSizePolicy`：自适应策略，将设置堆大小的任务交给虚拟机



#### CMS

JDK9 中被弃用

##### Young GC/Minor GC

Minor GC 是针对 GC 堆中年轻代的垃圾回收操作。Eden Region 空间不足触发，将活跃对象 Copy 到 survivor Region 或者直接晋升到 old region 中，空闲的 Region 进入空闲列表等待下一次使用。



**常用的调优参数**

`-XX:NewRatio`：设置老年代和年轻代的大小比例，默认 2:1

`-X:SurvivorRatio`：设置 Eden 区和一个 Survivor 区 的比值，默认 8:1

`-XX: MaxTenuringThreshold` 默认15，若对象超过年龄阈值则晋升到老年代

`-XX:+PretenureSizeThreshold` 若新创建的对象超过这个阈值则直接晋升到老年代



**回收垃圾的算法：**

使用标记-复制算法



###### 触发条件

Eden 区空间不足



###### 晋升条件

从年轻代晋升到老年代

* 经历 MaxTenuringThreshold 次的 Minor GC 后依然存活的对象 `--XX:TenuringThreshold`
* 新创建的超过 PretenuerSizeThreshold 大小的大对象
* Survivor 区中存放不下的对象



##### Full GC/Major GC

Full GC 是针对整个 GC 堆中新生代和老年代都包括的垃圾回收操作。



###### 触发条件

* **老年代空间不足** (在进行一次 Minor GC 时，发现老年代的剩余空间大小小于历次晋升对象的平均大小)
* **显式调用 `System.gc()` 方法**
* **在使用 CMS 垃圾收集器时，出现GC失败** (promotion failed 晋升失败，concurrent mode failure 无法在规定时间内完成收集工作)
* JDK7 的永久代空间不足 (JDK8 的元空间使用本地内存，不会因空间不足而触发 Full GC)
* 在使用 RMI(Remote Method Invocation) 远程方法调用来进行 RPC 或管理的 DK(Distributed Key-Value) 分布式键值对应用时，每小时执行1次 Full GC



**回收垃圾的算法：**

在年轻代使用标记-复制算法

在老年代使用标记-整理算法



**Full GC 和 Major GC 的区别：**

* Major GC 可以指 Full GC、也可以指老年代 GC (面试需问清楚 Major GC 指什么)
* Full GC 比 Minor GC 慢，但执行频率低



###### STW 阶段停顿时间

>  STW暂停 Stop-the-World
>
>  在 Full GC 运行时，会在所有应用程序线程到达各自**安全点**后暂停执行，以确保对象在垃圾回收期间不会被并发修改或访问，直到垃圾回收完成。这个暂停的时间被称为**停顿时间**。(在 Minor GC 运行时，只需要停止正在年轻代区域上的所有应用程序线程，不需要安全点)
>
>  停顿时间越长，应用程序的响应性就越差，用户可能会感觉到应用程序的卡顿或延迟。

只有每个线程都到达安全点后 JVM 才执行 Full GC 操作
Safepoint (安全点) 是应用程序线程暂停的位置，暂停应用程序线程后对象引用关系不会发生变化，分析结果具有确定性。

产生安全点的地方：方法调用指令、循环跳转指令、异常跳转指令、主动式安全点等处

安全点数量太少会导致 Full GC 停顿事件太长，安全点数量太多会频繁检测导致程序负荷大。



#### 分代ZGC

[JEP333 ZGC](https://openjdk.org/jeps/333)	|	[JEP439 分代ZGC](https://openjdk.org/jeps/439)	|	[OpenJDK Wiki#ZGC的配置与调优](https://wiki.openjdk.org/display/zgc#Main-Configuration&Tuning)

不分代的问题：OS内存占用(记录卡表)、CPU占用过高

目标 ：
**低延迟(暂停时间不超过1ms)**
自动调优auto-tuning(不需要手动配置`--Xmn`(空闲内存对任何一代可用)、分代规模、线程数`--XX:ConcGCThreads`、对象停留在年轻代的时间)
可扩展(可处理不同大小的堆 [8MB, 16TB])

使用：`java -XX:+UseZGC -XX:+ZGenerational...`
`--Xmx`
`-Xlog:gc` 基础日志
`-Xlog:gc*` 详细日志

自动调优：
动态代大小，不需要指定 `--Xmn`
动态晋升阈值(经过多少次Young GC年轻代中的对象晋升到老年代)，不需要指定 `--XX:TenuringThreshold`
初始化堆占用百分比，不需要指定 `--XX:InitiatingHeapOccupancyPercent`
动态线程数量，不需要指定 `-XX:ConcGCThreads`

核心技术：
ZGC 通过染色指针(colored pointer)、读屏障(load barriers)、存储屏障(store barriers)，为GC线程、并发的应用线程提供一致的对象视图(object graph view)，实现了GC线程与应用程序线程的并发

**染色指针**(colored pointer)：64位的指针指向堆中对象，提供**对象地址和元数据位**(remapped, marked0, marked1, Finalizable)，是一种多映射内存技术(Multi-mapped memory)，未分代ZGC将多个虚拟地址指向同一块物理内存
优点：
一个 Region 分区中所有对象重定位后内存空间可以立即被回收和重用，在修复指针(指向被回收/被重用区域)之前，这有助于降低堆开销
减少内存屏障(用于修正指针)的使用数量(降低运行时开销)
具备强大扩展性

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting2/img/2023_1027_124923.png" alt="image-20231027124923174" style="zoom:60%;" />

读屏障(load barriers)：是被JIT(即时编译)注入到应用程序的机械指令，更新时指针指向重定位(relocated)对象(重定位时指针不会被急更新，而是当应用遇到该指针时懒更新)

写屏障(store barriers)：保证共享变量的修改可以被其他线程及时感知，帮助标记可达对象(标记loaded对象为alive)

记忆集(remembered-set)：指从老年代到年轻代的指针(old-to-young generation pointers)
卡表标记(card table marking)：通过记忆集合技术，用于跟踪代际间指针(inter-generational pointers)



ZGC阶段：

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting2/img/2023_1027_115136.png" alt="image-20231027115136375" style="zoom: 45%;" />

同步点：指同步状态发生切换的点，是GC暂停的唯一来源

<table>
    <tr>
    	<td><center><img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting2/img/2023_1027_115047.png" alt="image-20231027115045462" style="zoom:55%;" /></center></td>
    	<td><center><img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting2/img/2023_1027_115233.png" alt="image-20231027115233605" style="zoom:55%;" /></center></td>
    </tr>
    <tr>
    	<td><center></center></td>
    	<td><center><img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting2/img/2023_1027_115251.png" alt="image-20231027115251210" style="zoom:55%;" /></center></td>
    </tr>
    <tr>
    	<td><center><img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting2/img/2023_1027_115357.png" alt="image-20231027115357552" style="zoom:55%;" /></center></td>
    	<td><center><img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting2/img/2023_1027_115428.png" alt="image-20231027115428409" style="zoom:55%;" /></center></td>
    </tr>
    <tr>
    	<td><center><img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting2/img/2023_1027_122101.png" alt="image-20231027122101547" style="zoom:55%;" /></center></td>
    	<td><center><img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting2/img/2023_1027_115542.png" alt="image-20231027115542004" style="zoom:55%;" /></center></td>
    </tr>
    <tr>
    	<td><center></center></td>
    	<td><center><img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting2/img/2023_1027_115552.png" alt="image-20231027115552010" style="zoom:55%;" /></center></td>
    </tr>
</table>



#### 年轻代收集器

##### 串行垃圾收集器

Serial Garbage Collector 标记-复制算法

说明：它为单线程环境设计，只使用一个单独的线程进行垃圾回收。是 Client 模式默认的年轻代收集器。

使用：通过 JVM 参数 `-XX:+UseSerialGC` 可以使用串行垃圾回收器。

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_05_19_19_57.png" alt="image-20230519195542293" style="zoom:50%;" />



##### 并行垃圾收集器

Parallel Scavenge /ˈskævɪndʒ/ 标记-复制算法

说明：使用多线程进行垃圾回收，也会暂停用户线程。

使用：可用 `-XX:+UseParallelGC` 来强制指定，用 `-XX:ParallelGCThreads=4`  来指定线程数。

适用于：多CPU，对暂停时间要求暂短的应用

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_05_19_19_55.png" alt="image-20230519195507076" style="zoom:50%;" />

* 比起关注用户线程停顿时间，**更关注系统的吞吐量**，适合后台运算不需要交互的程序
* 在多核下执行才有优势，Server 模式下默认的年轻代收集器

(吞吐量=运行用户代码时间/（运行用户代码时间+垃圾收集时间)



##### 新并发垃圾收集器

ParNew Garbage Collector  标记-复制算法

说明：并发标记垃圾回收使用多线程扫描堆内存，标记需要清理的实例并且清理被标记过的实例。ParNew GC 只会在下面两种情况持有应用程序所有线程。
第一，当标记的引用对象在 Tenured 区域。
第二，在进行垃圾回收的时候堆内存的数据发生改变
相比于Parallel GC，ParNew GC 使用更多的 CPU 来扫描程序，如果能分配为了程序性能更多的 CPU 那么 ParNew 比 Parallel 更好

使用：通过 JVM 参数 `XX:+USeParNewGC` 打开并发标记扫描垃圾回收器。

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_05_19_19_55.png" alt="image-20230519195507076" style="zoom:50%;" />

* 多线程收集，其余的行为、特点和 Serial 收集器一样
* 单核执行效率不如Serial，在多核下执行才有优势，
* 能与 CMS 配合工作，是 Server 模式下的首选年轻代收集器  
* **以减少 Stop-the-World 为出发点，关注用户线程停顿时间**，适合与用户交互的程序



#### 老年代收集器

##### Serial Old

(启动参数：`-XX:+UseSerialOldGC`，标记-整理算法)

* 单线程收集，进行垃圾收集时，必须暂停所有工作线程
* 简单高效，Client模式下默认的老年代收集器



##### Parallel Old

(启动参数：`-XX:+UseParallelOldGC`，标记-整理算法)

* 多线程，**吞吐量优先**



##### 并发标记扫描垃圾收集器

CMS Concurrent Mark-Sweep 并发标记清除

(启动参数：`-XX:+UseConcMarkSweepGC`，标记-清除算法)

![image-20230520162923360](https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_05_20_16_29.png)

* **关注用户线程停顿时间**

* **初始标记**：stop-the-world
* 并发标记：并发追溯标记，程序不会停顿
* 并发预清理：查找执行并发标记阶段从年轻代晋升到老年代的对象
* **重新标记**：stop-the-world 暂停虚拟机，扫描CMS堆中的剩余对象
* 并发清理：清理垃圾对象，程序不会停顿，标记-清除算法
* 并发重置：重置CMS收集器的数据结构

因为 Parallel Scavenge，G1 是独立实现的，其余的共用了框架代码，所以 CMS 不能与 Parallel Scavenge、G1 共用



##### G1 垃圾收集器

Garbage First 标记-复制+标记-整理算法

说明：G1 收集器是当今收集器技术发展最前沿的成果，它是一款面向服务端应用的收集器它能充分利用多CPU、多核环境。因此它是一款并行与并发收集器，并且它能建立可预测的停顿时间模型。**JDK[9+ 成为默认的收集器**

使用：通过 JVM 参数 `-XX:+UseG1GC` 使用G1垃圾回收器，`--XX:G1HeapRegionSize` 指定Region大小(1,2,4,8,16,32M)。

适用于：堆内存很大的情况

* 将整个 Java 堆内存划分成多个大小相等的区域，并发地进行垃圾回收，在回收内存之后对剩余堆内存空间进行压缩
* 年轻代和老年代不再物理隔离

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_09_11_01_36.png" alt="image-20230911013623502" style="zoom:80%;" />

Garbage First 收集器的特点：并行和并发、分代收集、碎片整理压缩、可预测的停顿(可设置停顿时间，通过启发式算法设定收集集合)



G1 的三种 GC 模式：

* young gc

* mixed gc
  = young + old gc

触发条件：老年代达到整个堆的一个阈值，避免堆内存耗尽

* full gc

单线程执行，暂停时间长，需要不断调优避免 full gc

触发条件：对象分配过快 mixed gc 来不及回收导致老年代被填满




### GC 调优

[JDK21 GC调优指南(每个版本都有)](https://docs.oracle.com/en/java/javase/21/gctuning/introduction-garbage-collection-tuning.html#GUID-326EB4CF-8C8C-4267-8355-21AB04F0D304)



### GC相关的面试题

#### finalize() 方法的作用

JDK9 中被标记为@Deprecated，JDK11中被弃用

* **与C++的析构函数不同，析构函数调用是确定的，而它的是不确定的**
* 将未被引用的对象放置于 F-Queue 队列
* **方法执行随时可能会被终止**
* 给予对象最后一次重生的机会

```java
public class Finalization {
    public static Finalization finalization;
    @Override
    protected void finalize(){
        System.out.println("Finalized");
        finalization = this;  // 重生
    }
}
```

由于 finalize() 运行的不确定性较大，无法各对象的保证顺序，运行代价高，因此不建议使用 finalize() 方法



#### Java的4种引用

在 java.lang.ref 包下的引用类型

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_06_02_13_42.png" alt="image-20230602134245031" style="zoom: 70%;" />

使用这些引用类型可能会造成内存泄漏或意外的对象回收，在大多数情况下还是用强引用



##### 总结

值不可变(immutable)

| 引用类型 | 回收时机(可达性不同) | 用途                                               |
| -------- | -------------------- | -------------------------------------------------- |
| 强引用   | 不会                 | 对象的一般状态                                     |
| 软引用   | OOM之前，内存紧张时  | 对象缓存                                           |
| 弱引用   | GC 时                | 对象缓存                                           |
| 虚引用   | Unknown              | 标记、哨兵                                         |
| 引用队列 |                      | 注册引用到引用队列，请求被通知当对象可达性的变化时 |



##### 强引用Strong

* **最普遍的**引用：`Object obj = new Object()`，obj 是引用对象(reference object)，new Object是引用实例(referent)
* 回收时机：即使抛出 OOM，GC 也不会回收被强引用的关联着的对象
* 通过将对象设置为 null 来弱化引用，使其被回收



##### 软引用Soft

* 对象处在有用但非必须的状态
* 回收时机：只有快要 OOM 之前之前，GC 才会回收被软引用的关联着的对象
* 可以**用来实现高速缓存**

示例：

```java
String str = new String("abc");  // 强引用
SoftReference<string>softRef = new SoftReference<string>(str);  // 包装为软引用
```



##### 弱引用Weak 

* 非必须的对象，比软引用更弱一些
* 回收时机：无论内存是否紧缺，GC 时都会回收被弱引用的关联着的对象
* 被回收的概率也不大，因为 GC 线程优先级比较低
* 适用于引用偶尔被使用且不影响垃圾收集的对象

**问题案例：**
利用软引用和弱引用解决 OOM 问题：假如有一个应用需要读取大量的本地图片，如果每次读取图片都从硬盘读取，则会严重影响性能，但是如果全部加载到内存当中，又有可能造成内存溢出，此时使用软引用可以解决这个问题。

设计思路是：用一个 HashMap 来保存图片的路径和相应图片对象关联的软引用之间的映射关系，在内存不足时，JVM 会自动回收这些缓存图片对象所占用的空间，从而有效地避免了 OOM 的问题。

示例1：

```java
public class NormalObjectWeakReference extends WeakReference<NormalObject> {
    public String name;
    // 构造函数用来初始化成员变量
    public NormalObjectWeakReference(NormalObject normalObject, ReferenceQueue<NormalObject> rq) {
        super(normalObject, rq);
        this.name = normalObject.name;
}	}
```

示例2：

```java
NormalObject nobj = new NormalObject("abc");  // 强引用
WeakReference<string>abcWeakRef = new WeakReference<string>(nobj);  // 包装为弱引用
```



##### 虚引用Phantom

* 不会决定对象的生命周期
* 任何时候都可能被垃圾收集器回收
* 跟踪对象被垃圾收集器回收的活动，起**哨兵**作用
* 必须和引用队列 Reference Queue 联合使用

示例：

```java
String str = new String("abc");  // 强引用
PhantomReference ref = new PhantomReference(str,queue);  // 包装为虚引用
```



##### 引用队列ReferenceQueue

* 无实际存储结构，存储逻辑依赖于内部节点之间的关系来表达
* **GC 完成后，ReferenceQueue 会存储，与被回收的软、弱、虚引用实例 referent 相关联的 reference object 引用对象**

如果实例被回收，则 reference 对象会被放进 ReferenceQueue。倘若没有 ReferenceQueue，则只能轮询 reference 对象再通过 get 是否返回空判断实例是否被回收。

示例：

```java
public class ReferenceQueueTest {
    private static ReferenceQueue<NormalObject> rq = new ReferenceQueue<NormalObject>();  // 引用队列

    /**
     * 遍历引用队列rq，检查rq中是否有对象，如果有，打印出来
     */
    private static void checkQueue(){
        Reference<NormalObject> ref = null;
        while ((ref = (Reference<NormalObject>)rq.poll()) != null){
            if (ref != null){
                System.out.println("In queue: " + ((NormalObjectWeakReference)(ref)).name);
                System.out.println("reference object:" + ref.get());  // 返回引用对象所指向的引用实例
            }
        }
    }

    public static void main(String[] args) {
        // 创建3个弱引用放入ArrayList容器
        ArrayList<WeakReference<NormalObject>> weakList = new ArrayList<WeakReference<NormalObject>>();
        for (int i =0; i < 3 ; i++){
            weakList.add(new NormalObjectWeakReference(new NormalObject("Weak " + i),rq));
            System.out.println("Created weak:" + weakList.get(i));
        }
		// 第一次，遍历引用队列rq
        System.out.println("first time");
        checkQueue();  // 说明此时 rq 为空
        // 回收 NormalObjectWeakReference 弱引用
        System.gc();
        try {
            Thread.currentThread().sleep(1000);
        } catch (InterruptedException e){
            e.printStackTrace();
        }
        // 第二次，遍历引用队列rq
        System.out.println("second time");
        checkQueue();
    }
}
```



## Java工具

| 以前的 Java 工具 | 命令文档                                                     | 描述                                           |
| ---------------- | ------------------------------------------------------------ | ---------------------------------------------- |
| jps              | [文档](https://docs.oracle.com/en/java/javase/20/docs/specs/man/jps.html) | 列出目标系统上已检测的 JVM，列出PID            |
| jinfo            | [文档](https://docs.oracle.com/en/java/javase/20/docs/specs/man/jinfo.html) | 生成指定 Java 进程的 Java 配置信息             |
| jmap             | [文档](https://docs.oracle.com/en/java/javase/20/docs/specs/man/jmap.html) | 打印指定进程的详细信息                         |
| jstat            | [文档](https://docs.oracle.com/en/java/javase/20/docs/specs/man/jstat.html) | 监控 JVM 统计信息                              |
| jstack           | [文档](https://docs.oracle.com/en/java/javase/20/docs/specs/man/jstack.html) | 打印指定 Java 进程的 Java 线程的 Java 堆栈跟踪 |



### javac

[文档](https://docs.oracle.com/en/java/javase/20/docs/specs/man/javac.html)

读取 Java 源代码中类和接口定义，编译为 Class 字节码文件，其中的 class 文件常量池中含有字面量和符号引用。添加静态常量成员变量 `Classname.class`，用于指向该类的Class对象，同时添加无参构造方法

```bash
javac -encoding utf-8 <options> <source files>
```



### java

[文档](https://docs.oracle.com/en/java/javase/20/docs/specs/man/java.html)	|	[VM 选项](https://chriswhocodes.com/hotspot_options_openjdk21.html)	|	[Java 命令行检查(用于优化 Java 命令行)](https://jacoline.dev/inspect)

```bash
$ java [options] <主类> [args...]  # 执行类，主类为全类名(不需要后缀)
$ java [options] -jar <jar 文件> [args...]  # 模块运行
```

```bash
java -javaagent:<jar 路径>[=<选项>]  # 加载 Java 代理/探针程序(java.lang.instrument实现)到 JVM 中，用于性能分析、字节码增强、动态代码注入
```



#### JVM 的运行模式

Client 模式：启动快，程序运行慢，轻量级VM，**JDK9 被弃用**

```bash
$ java -client YourApp
```

Server 模式：启动慢，程序运行快，重型VM

```bash
$ java -server YourApp
```



### javap

对Class文件反汇编，用于查看字节码

```bash
javap -verbose <classes>
```

```java
public class ByteCodeSample {
    public static void main(String[] args) {
        int i=1,j=5;
        i++;
        ++j;
        System.out.println(i);
        System.out.println(j);
    }
}
```

```java
$ javap -c ByteCodeSample.class  
Compiled from "ByteCodeSample.java"
public class com.interview.javabasic.bytecode.ByteCodeSample {
  public com.interview.javabasic.bytecode.ByteCodeSample();  // 无参构造方法
    Code:  
       0: aload_0  // 加载句柄
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V  // 调用父类构造初始化
       4: return

  public static void main(java.lang.String[]);
    Code:  // Java虚指令
       0: iconst_1  // 常量1压入->操作数栈
       1: istore_1  // 操作数栈pop->局部变量表[1]
       2: iconst_5  // 常量5压入->操作数栈
       3: istore_2  // 操作数栈pop->局部变量表[2]
       4: iinc          1, 1
       7: iinc          2, 1
      10: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;  // 获取静态域
      13: iload_1  // 局部变量表[1]压入->操作数栈
      14: invokevirtual #3                  // Method java/io/PrintStream.println:(I)V  // 调用println
      17: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
      20: iload_2  // 局部变量表[2]压入->操作数栈
      21: invokevirtual #3                  // Method java/io/PrintStream.println:(I)V
      24: return
}
```



### JConsole

Java Monitoring and Management Console，有 GUI 界面



### arthas

[github](https://github.com/alibaba/arthas/blob/master/README_CN.md)



### mat

堆内存分析工具



