# Python3.12

[Python官方文档](https://docs.python.org/zh-cn/3.12/) | [Python各版本状态](https://devguide.python.org/versions/#versions)

[电子书](https://pythonhowto.readthedocs.io/zh-cn/latest/index.html) | [pandas](https://mlhowto.readthedocs.io/en/latest/pandas.html)

* 编译型语言：在运行前需要一个编译的过程。
   优点：程序运行速度快
   缺点：跨平台性差(依赖编译器)
  **解释型语言**：在运行时先解释再运行。
   优点：跨平台性好、可以逐行执行
   缺点：程序运行速度慢
* 静态类型定义语言：变量类型在使用变量之前静态声明，编译时判断
  **动态类型定义语言**：变量类型在第一次赋值时动态声明，运行时判断
   鸭子类型：是一种设计原则，关注对象的行为(方法和属性)，不关注对象的具体类型
   缺点：程序运行速度慢，所以建议尽量Pythonic地使用内置函数(filter、map)、列表生成式

* 强类型语言(对于类型转换而言)：类型独立，不轻易转换
  弱类型语言：类型不严格区分，不需要明确的类型转换操作

## 解释器

### CPython

Python GIL：同一时刻只有一个线程进行字节码级别的执行，阻止不同的线程并发访问C对象。一个函数中包含多个字节码指令仍存在并发问题
意图：用于引用计数垃圾回收的锁，使GC正常工作、使Python胶水的**C拓展模块线程安全**不会runtime error
劣势：多线程性能低

无锁: 多线程性能提升，单线程性能下降
 由于单线程性能下降太多、多进程也能在多CPU上执行，所以迟迟没有实现

### 没有GIL

Pypy：JIT, 适合对执行性能有要求的
Jython: 基于JVM实现的Python，用于与Java结合的项目
IronPython: 基于.NET, 用于与 .net 结合的项目

## PEP

PEP(Python Enhancement Proposals Python增强提案): 是设计文档，分为Standards Track 新特性或实现、Informational 指导信息、Process 流程  
[PEP8](https://peps.python.org/pep-0008/#public-and-internal-interfaces)  
[PEP20](https://peps.python.org/pep-0020/)  

[Python Discourse#Python论坛](https://discuss.python.org/)  
[The Python Language Reference](https://docs.python.org/3/reference/index.html)
[标准库](https://docs.python.org/3/library/index.html)  
BDFL(Benevolent Dictator For Life): Guido van Rossum  

## 命令行

```bash
python -m pip --version  # python -m 运行库模块
```

## 内置函数

filter、map

列表生成式

## 数据类型

字符串、List、Tuple、dict

## 列表生成式

```python
[num * 2 for num in lst]
```

## 函数

| 参数     | 释义                                                         | 类型 | 示例                      |
| -------- | ------------------------------------------------------------ | ---- | ------------------------- |
| *args    | 表示**位置参数**，位置参数是根据位置顺序传递给函数的参数     | 元组 | foo("Alice",25)           |
| **kwargs | 表示**关键字参数**(keyword arguments)，关键字参数是以关键字形式传递给函数的参数 | 字典 | foo(name="Alice", age=25) |

### decorator装饰器

decorator是对方法增强的语法糖。装饰器方法会隐式传入self参数

```python
def deco(fun):  # 装饰器
    def inner():
        print("before execute")
        fun()
        print("after execute")
    return inner

@deco
def foo():
    print("Hello")

foo()
```

等价

```python
foo = deco(foo)  # return inner
```

等价

```python
def deco(fun):  # 装饰器
    def inner():
        print("before execute")
        fun()
        print("after execute")
    return inner

def foo():
    print("Hello")

deco(foo)()
```

```python
def timeit(iteration):  # 装饰器
    def inner(f):
        def wrapper(*args, **kwargs):
            start = time.time()
            for _ in range(iteration):
                ret = f(*args, **kwargs)
            print(time.time() - start)
            return ret
        return wrapper
    return inner

@timeit(1000)
def double(x):
    return x * 2

double(2)
```

等价

```python
double = timeit(1000)(double) = inner(double) = wrapper(*args, **kwargs)
```

@classmethod、@staticmethod、@property 不是装饰器

## 错误和异常

## 标准库+第三方库

time

# 未整理

脚本语言，解析器在运行时进行解释。

性能与解释器有关，Pythonic的写法可能比逻辑本身更重要

解释器：CPython、PyPy

## 导模块

导方法

```python
from time import time, sleep

sleep(1)
```

导对象

```python
import time

time.sleep(1)
```

## 变量

基础类型或对象的引用

### List []

#### append()

append附加，添加一个对象到 List

### 元组

```python
# 简单表示
xx, yy
```

### 生成表达式 性能远远更好

生成式语法：
[被追加的数据 循环语句 循环或者判断语句]
{被追加的数据 循环语句 循环或者判断语句}

## 运算符

### is

is 比较对象地址，== 比较对象的值

### in

返回 Boolean

```python
k in values
```

## 函数

### isinstance(value, int)

判断是否在该类型下

### len(*args, **kwargs)

返回长度

### all(*args, **kwargs)

### String

#### r'x'x'x'

告诉编译器这个string是个raw string，不要转意反斜杠。

例如，\n 在raw string中，是两个字符，\和n， 而不会转意为换行符。由于正则表达式和 \ 会有冲突，因此，当一个字符串使用了正则表达式后，最好在前面加上’r’。

#### f‘xxx’

```python
f'{last_proof}{proof}'
```

Python3.6引入，表示格式化字符串，可以在{}中添加变量或表达式

性能优于%和format函数，推荐使用

#### encode()

```python
"""
Python encode() 方法以 encoding 指定的编码格式编码字符串。errors参数可以指定不同的错误处理方案。
"""
str.encode(encoding='UTF-8',errors='strict') # 默认为这种格式，utf-8
```

## Clazz类

用 Clazz可以避免关键字(不推荐)  
推荐使用class_

```python
class A(object):
    def m1(self, n):
        print("self:", self)

    @classmethod
    def m2(cls, n):
        print("cls:", cls)

    @staticmethod
    def m3(n):
        pass

a = A()
a.m1(1) # self: <__main__.A object at 0x000001E596E41A90>
A.m2(1) # cls: <class '__main__.A'>
A.m3(1)
```

### 类的生命周期 (调用过程)

* 第一步：系统执行 class 代码，创建一个**类 A 对象**（一切皆对象，类也是对象，属性为方法名，该对象的职责是用于实例化），同时初始化类里面的属性和方法。此时还没创建**实例对象**。
* 第二、三步：执行 a = A() 代码，调用类的构造器，创造出实例对象 a
* 第四步：执行 a.m1(1) 代码，实例方法，内部会隐式地把实例对象传递给 self 参数进行绑定，也就是说， self 和 a 指向的是同一个实例对象。
* 第五步：执行A.m2(1) 代码，类方法，内部会隐式地把类对象传递给 cls 参数进行绑定，也就是说，cls 和 A 指向的是同一个类对象。
* 第六步：执行A.m3(1) 代码，静态方法，

### 权限 (属性或方法的)

私有属性或方法以两个下划线 __开头，在类外部直接访问私有会抛出异常，可使用\_Class\__arr访问

```python
class A:
    def __init__(self):
        self.__name = "A"
 
    def fun1(self):
        # 在类内部使用私有成员变量__name
        print(self.__name)
 
a = A()
a.fun1() # OK
 
# 直接访问会抛出异常
print("name of boy:", boy.__name)
# 通过_Class__可以在外部访问私有变量__name
print("name of boy:", a._A__name)
```

### 方法

#### 装饰器

和 Java注解不同，注解就是对类的描述符 / 标记

装饰器：可以是函数，Python 装饰器 == Java 注解标记 + AOP执行逻辑

对方法使用，用于修饰 类与实例的绑定关系

* 实例方法：不带装饰器
* 类方法：**@classmethod**，可以通过类调用方法，有cls参数, 不需要创建类实例
* 静态方法：**@staticmethod**，可以通过类调用方法，无cls参数

对方法使用，将类的方法当做属性使用

* @property
* @[property修饰的方法].setter

##### 实例方法属性与实例方法

```python
# 配合上文类下的代码看

print(A.m1)
# 在py2中类对象调用实例方法属性：<unbound method A.m1>
# 在py3中类对象调用实例方法属性：<function A.m1 at 0x000002BF7FF9A488>

print(a.m1)
# 在py3中实例对象调用实例方法属性：<bound method A.m1 of <__main__.A object at 0x000002BF7FFA2BE0>>


A.m1(a, 1)
# 等价  
a.m1(1)
```

A.m1是还没有绑定实例对象的方法属性，方法调用时需要显示地传入一个实例对象进入

a.m1是已经绑定了实例对象的方法属性，方法调用时隐式传入了实例对象

##### 类方法属性与类方法

**使用场景：类方法可作为工厂方法，用于创建实例对象**

```python
# 内置模块 datetime.date 类
class date:

    def __new__(cls, year, month=None, day=None):
        self = object.__new__(cls)
        self._year = year
        self._month = month
        self._day = day
        return self

    @classmethod
    def fromtimestamp(cls, t):
        y, m, d, hh, mm, ss, weekday, jday, dst = _time.localtime(t)
        return cls(y, m, d)

    @classmethod
    def today(cls):
        t = _time.time()
        return cls.fromtimestamp(t)
```

```python
print(A.m2)
<bound method A.m2 of <class '__main__.A'>>

print(a.m2)
<bound method A.m2 of <class '__main__.A'>>


A.m2(1) 
# 等价
a.m2(1)
```

A.m2和a.m2在方法调用时都会隐式地传入类对象

##### 静态方法属性与静态方法

**使用场景：在方法中不需要访问任何实例方法和属性，纯粹地通过传入参数并返回数据的功能性方法**

类中的静态方法和类外的普通函数没区别。
类与实例没有所谓的绑定关系。

```python
print(A.m2)  # 等价于 print(a.m2)
<function A.m3 at 0x000002544FE0F9D8>

A.m3(1) 
# 等价
a.m3(1)
```

##### Getter/Setter方法

@property：表示 Getter方法

@[property修饰的方法].setter：表示 Setter方法

```python
class Cat(object):

    def __init__(self, name, age):
        self.name = name
        self.__age = age

    # @property 将类的方法当做属性使用，类似 getter方法需要返回一个值(默认为None)，只不过是属性的形式
    @property
    def age(self):
        return self.__age

    # 类似 setter方法的属性形式，需要在getter基础之上即@age
    @age.setter
    def age(self, value):
        if not isinstance(value, int):
            print('年龄只能是整数')
            return -1
        if value < 0 or value > 100:
            print('年龄不能太大或太小')
            return -1
        self.__age = value

    @property
    def show_info(self):
        print('我是名字是：{0}，年龄：{1}'.format(self.name, self.__age))

    def __str__(self):
        return '信息'


if __name__ == '__main__':
    black = Cat('小黑', 2)
    print(Cat)  # Cat return <class '__main__.Cat'>
    print(black)  # return function __str__ or <__main__.Cat object at 0x00000223B9E01C08>
    black.show_info  # return <bound method Cat.show_info of <__main__.Cat object at 0x000001D19CAA1B88>>
                     # object address
                    # black.show_info会执行函数逻辑并返回None
    black.age = -1
    black.show_info

打印：
<class '__main__.Cat'>
信息
我是名字是：小黑，年龄：2
年龄不能太大或太小
我是名字是：小黑，年龄：2
```

#### 类的魔法方法

##### def \__str__(self)

类似 Java中的 toStrin();，直接返回实例对象时转为 String

## 库

### Json序列化

```python
import json

block_string = json.dumps(block, sort_keys=True).encode()
```

### 时间

```python
from time import time

time()  # >> 1621494970.4989505
```

## python 操作 CSV数据

Comma-Separated Values 逗号分隔值，其文件以纯文本形式存储**表格数据**，每个记录由一个或多个字段组成，用逗号分隔。

也可以用 excel 打开

```python
import pandas as pd
dataset = pd.read_csv('./xxx.csv')  # dataframe类型 二维表格类型 有索引
dataset.head()

shape属性
columns属性
info()
describe()  # 查看数据集统计特征 (常用)

1 导入库
2 导入数据集，查看数据集
3 提取特征。提取特征 提取标签
feature_columns = ['Hours']
label_column = ['Scores']
4 建立模型
用训练集的数据进行训练，对测试集进行预测
```

## 以下未整理

函数
type

变量
每次赋值都是声明和定义的过程，赋值会改变变量的类型
基本数据类型int、float、str、bool
type

基本运算符、
/浮点数除法10/2=5.0
//底板除
**幂

列表

字典
