---
title: Node.js
date: 2023-08-23
update: 2023-08-23

tags:
categories:
  - Node.js
keywords: Node.js,nvm,npm,Hexo
description: 主要是关于Node.js介绍，nvm和npm的使用，以及Hexo博客框架+b=Butterfly主题的使用
top_img:
cover:
---

# Node.js20.5.1

Node.js 是一个开源、跨平台的事件驱动 I/O 的服务端  JavaScript 运行时环境，基于 Google 的 V8 引擎。后端

* js 通过 node 在服务器上运行
* node 提供了大量的工具库 js 可以和操作系统进行交互 (读写文件)
  Node 环境中没有 Web APIs ，和浏览器的上下文环境不同

V8 引擎：执行 Javascript 的速度非常快性能非常好、单线程运行一次只能运行一个任务。

[官网](https://nodejs.org/zh-cn)



## nvm-windows1.1.11

nvm-windows 是 **node 版本管理工具**，目前项目开发组转去开发新工具 [Runtime](https://github.com/coreybutler/nvm-windows/wiki/Runtime) 了。

[用法](https://github.com/coreybutler/nvm-windows#usage)



### 配置

```cmd
nvm node_mirror https://npmmirror.com/mirrors/node/
nvm npm_mirror https://npmmirror.com/mirrors/npm/
```



### 其他Node版本管理工具

[nvs](https://github.com/jasongin/nvs/)



## npm9.8.1

npm 是 **node 包管理工具**，在安装 Node.js 时，会自动安装 npm，但是 npm 的发布比 Node.js 更频繁，因此记得更新 npm。

[npm CLI 文档](https://docs.npmjs.com/cli/v9/commands)	|	[npm 包搜索引擎](https://www.npmjs.com/package/package)



### npm配置

[.npmrc 配置](https://docs.npmjs.com/cli/v9/using-npm/config)

[npm 注册中心镜像](https://npmmirror.com/)



### npm命令

https://biaoyansu.com/20.4

| 常用命令                    | 释义                                                         | 参数                                                         |
| --------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| npm list [package]          | 查看已经安装的包                                             | -g                                                           |
|                             |                                                              |                                                              |
| npm i xxx -S                | **全局安装：**在命令行上运行。安装位置：在 node 根路径下，即 E:\Develop\nvm\v18.14.2\node_modules<br />**局域安装**，需要`require()`。安装位置：当前目录下创建 node_modules 文件夹<br />**最佳实践**：`npm --registry https://registry.npm.taobao.org install` | --save = -S，package.json里面的dependencies：生产环境需要依赖的包<br />--save-dev = -D，packege.json里面的devDependencies，只有开发环境下需要依赖的库，线上生产环境上并不需要他们，如开发中所使用的外部的测试或者文档框架，vue-cli脚手架 |
| `npm install npm@latest -g` | 升级 npm                                                     |                                                              |
|                             |                                                              |                                                              |
| npm cache clean --force     | 下载失败时，清理缓存                                         |                                                              |
| npm init -y (项目目录下)    | **初始化项目**，创建包信息<br />会创建pakeage.json文件记录项目开发中所要用到的包，以及项目的详细信息<br />使版本迭代和项目移植更加方便，防止误删包<br />pakeage.json中不要添加注释 |                                                              |
| npm install / npm i         | npm会根据package.json配置文件中的依赖配置下载安装包。        |                                                              |
| npm key                     | 执行value中的命令（pakeage.json中script）                    |                                                              |
| npm help <command>          | 帮助                                                         |                                                              |
|                             |                                                              |                                                              |



### 第三方包

#### nodemon

用于热编译代码，自动重启项目工程

| 命令                                   | 释义   |
| -------------------------------------- | ------ |
| nodemon ./server.js [localhost] [8080] | 启动js |
| nodemon -help                          | 帮助   |
|                                        |        |



#### es-checker

侦测运行环境支持哪些 ES6 的功能。





# 未整理

todo：目前了解 npm，有空再看后端 node.js 官网文档



# Node



## Node.js是什么？

### Node环境执行JavaScript

| 操作                                     | 释义                                               |
| ---------------------------------------- | -------------------------------------------------- |
| vscode 中：ctrl + `（是键盘上esc下面的） | 或者 查看——>终端打开 终端<br/>最好进入js的那个目录 |
| node .\demo.js                           | 执行                                               |
| node                                     | 进入node环境                                       |
| .exit                                    | 退出node环境                                       |

## ECMAScript6

### JavaScript 和 ECMAScript 的关系

1996 年 11 月，JavaScript 的创造者 Netscape 公司，决定将 JavaScript 提交给标准化组织 ECMA，希
望这种语言能够成为国际标准。次年，ECMA 发布 262 号标准文件（ECMA-262）的第一版，规定了浏
览器脚本语言的标准，并将这种语言称为 ECMAScript，这个版本就是 1.0 版。
JavaScript 是语言，ECMAScript 是标准化组织规定的标准

### ES6语法

#### 对象

对象是由多组 key: value键值对构成的，其中 key可以是数字，value 可以是 方法

```javascript
let obj1 = {
    param1: "111",
    myfunc: function() {
        console.log(this.param1);
        setTimeout(function() {
            console.log(this.param1)  // this指向global
        }, 1000);
    },
    num: 5,
    str: "123",
    1: 12
}
obj1.myfunc();
console.log(obj1[1]); // 输出 数字键 要使用 索引
```

#### 变量声明n

##### 不用修饰符

该变量为全局作用域

##### var

* 变量提升
* 函数作用域、全局作用域（存在变量泄露）

##### let

* 不存在变量提升

* 块级作用域

* 在 相同 作用域内不允许重复声明同一个变量

> 变量提升 (提前声明)：将当前作用域 (全局、函数作用域) 的所有变量的声明提升到程序的顶部，值为 undefined
>
> 变量泄露：
>
> - 在for(var i=0; i<1; i++)循环中 var 声明的变量在全局范围内都生效，所有的 变量i 都指向同一个 全局变量i ，调用方法时输出的值就是最后一轮 i 的值
> - 而let声明的变量在仅在块级作用域内生效，所以每一次for循环都生成了一个新的变量i，调用方法时输出的值是不同对象的值

##### const

* 不存在变量提升

* 在 相同 作用域内不允许重复声明同一个变量

* 声明只读常量，必须初始化赋值

> 如果赋值的是变量，还是可以进行原地变化的，如：
>
> 列表原地变化：改变值，地址并没有变化，只是改变引用对象的值

#### Destructuring (解构赋值)

ES6 允许按照一定模式，从**数组**和**对象**中提取值，对变量进行赋值，这被称为 Destructuring（解构）

#####   数组的解构赋值

`let [x = 100, y, z] = [200, 123, ];`

##### 对象的解构赋值

（还可以使用别名，不过原来的名字foo2就失效了）

`let {foo2:foo3, bar2="default"} = {foo2: "are"};`

##### 内置对象的解构赋值

```javascript
let {sin, cos, random} = Math;
console.log(typeof sin);
```

#### 字符串相关扩展

```javascript
includes()
startsWith()
endsWith()
```

#### 函数相关扩展

##### 参数默认值

##### 参数解构赋值

```javascript
function foo({name="zhangsan", age="18"} = {}) {
    console.log(name + "  " + age);
}
foo({name: "lisi", age: "20"});
```

##### rest参数 剩余参数

```javascript
function foo(a, b, ...param) {
    console.log(a);
    console.log(b);
    console.log(...param);
}
foo(1, 2, 3, 4, 5);
```

##### 箭头函数

只是一个语法糖，匿名函数，省略 function return的写法

一般在一些不需要重复使用的方法上使用箭头函数

```javascript
// 第一种写法
let obj = {
    sum: function(num1, num2) {
        return num1 + num2;
    },
}
console.log(obj.sum1(2,3));

// 第二种写法，省略return
let obj = {
    sum1 : (num1, num2) => (num1 + num2)
}
console.log(obj.sum1(2, 3))
```

#### this 指针

this 指向的是方法的调用者

```javascript
let obj1 = {
    param1: "111",
    myfunc: function() {
        console.log(this.param1);
        setTimeout(function() {
            console.log(this.param1)  // this指向 global
        }, 1000);
    }
}
obj1.myfunc();
// setTimeout是顶层对象的方法 windows的顶层对象是 dom的 window，node是 global (顶层对象可以省略)
```

使 this 指向定义时的对象

```javascript
// 解决方法1 保存对象再调用
let obj2 = {
    param1: "111",
    myfunc: function() {
        self = this;
        console.log(this.param1);
        setTimeout(function() {
            console.log(self.param1)  // self指向obj2
        }, 1000);
    }
}
obj2.myfunc();
```

#### 方法调用的三种方式（重点、常用）

>三种调用方式：
>
>- 同步调用：一种阻塞式调用，**调用方/客户方要等待对方执行完毕才返回**，它是一种单向调用；
>- 回调：一种双向调用模式，也就是说，**被调用方在接口被调用时也会调用对方的接口**；
>- 异步调用：一种类似消息或事件的机制，不过它的调用方向刚好相反，**接口的服务在收到某种讯息或发生某种事件时，会主动通知客户方（即调用客户方的接口）**。
>
>回调和异步调用的关系非常紧密：**使用回调来实现异步消息的注册，通过异步调用来实现消息的通知。**

##### 回调函数

回调函数，解决 this 指向问题，回调函数一般使用箭头函数

此时回调函数中的 this是从她们的语法范围内获取她们的值，不在乎叫什么，不在乎在哪里定义

也就是指向了定义时的对象

```javascript
// 解决方法2 使用回调函数调用this
let obj3 = {
    param1: "111",
    myfunc: function() {
        setTimeout(() => {
            console.log(this.param1)
        }, 1000);
    }
}
obj3.myfunc();
```

###### 定时器

异步回调

Interval：(时间上的) 间隔

- setTimeout(回调函数, 时间) 		一个定时器
- setInterval(回调函数, 时间) 		循环定时器，返回 intervalID
- clearInterval(intervalID) 			根据intervalID取消定时器

##### Promise对象

Promise对象，表示一个异步操作。对象和函数的区别就是对象可以保存状态，函数不可以（闭包除外）并未剥夺函数return的能力，因此无需层层传递callback，进行回调获取数据

```javascript
new Promise( // new时，不用调用，Promise中的函数就已经执行了
  function (resolve, reject) {
    // 一段耗时的异步操作
    resolve('要返回的数据可以任何数据例如接口返回数据') // 数据处理完成
    // reject('失败') // 数据处理出错
  }
).then(
  (res) => {console.log(res)},  // 成功
  (err) => {console.log(err)} // 失败
)
```

Promise有三种状态：pending、fulfilled、rejected

resolve：在异步操作成功时调用，将Promise对象的状态从“pending”变为“fulfilled”，并将异步操作的结果，作为参数传递出去；
reject：在异步操作失败时调用，将Promise对象的状态从“pending”变为“rejected”，并将异步操作报出的错误，作为参数传递出去；

then方法：其中有箭头函数，参数是resolve返回的结果，then和回调函数一个意思

###### 回调和Promise异步比较

```javascript
/***
   第一步：找到北京的id
   第二步：根据北京的id -> 找到北京公司的id
   第三步：根据北京公司的id -> 找到北京公司的详情
   目的：模拟链式调用、回调地狱
 ***/
 
 // 回调地狱
 // 请求第一个API: 地址在北京的公司的id
 $.ajax({
   url: 'https://www.easy-mock.com/mock/5a52256ad408383e0e3868d7/lagou/city',
   success (resCity) {
     let findCityId = resCity.filter(item => {
       if (item.id == 'c1') {
         return item
       }
     })[0].id
     
     $.ajax({
       //  请求第二个API: 根据上一个返回的在北京公司的id “findCityId”，找到北京公司的第一家公司的id
       url: 'https://www.easy-mock.com/mock/5a52256ad408383e0e3868d7/lagou/position-list',
       success (resPosition) {
         let findPostionId = resPosition.filter(item => {
           if(item.cityId == findCityId) {
             return item
           }
         })[0].id
         // 请求第三个API: 根据上一个API的id(findPostionId)找到具体公司，然后返回公司详情
         $.ajax({
           url: 'https://www.easy-mock.com/mock/5a52256ad408383e0e3868d7/lagou/company',
           success (resCom) {
             let comInfo = resCom.filter(item => {
               if (findPostionId == item.id) {
                 return item
               }
             })[0]
             console.log(comInfo)
           }
         })
       }
     })
   }
 })
```

```javascript
doSomething(function(result) {
  doSomethingElse(result, function(newResult) {
    doThirdThing(newResult, function(finalResult) {
      console.log('Got the final result: ' + finalResult);
    }, failureCallback);
  }, failureCallback);
}, failureCallback);
```



```javascript
    // Promise 写法
    // 第一步：获取城市列表
    const cityList = new Promise((resolve, reject) => {
        $.ajax({
            url: 'https://www.easy-mock.com/mock/5a52256ad408383e0e3868d7/lagou/city',
            success (res) {
                resolve(res)
            }
        })
    })

    // 第二步：找到城市是北京的id
    cityList.then(res => {
        let findCityId = res.filter(item => {
            if (item.id == 'c1') {
                return item
            }
        })[0].id
        findCompanyId().then(res => {
            // 第三步（2）：根据北京的id -> 找到北京公司的id
            let findPostionId = res.filter(item => {
                if(item.cityId == findCityId) {
                    return item
                }
            })[0].id
            // 第四步（2）：传入公司的id
            companyInfo(findPostionId)
        })
    })

    // 第三步（1）：根据北京的id -> 找到北京公司的id
    function findCompanyId () {
        let aaa = new Promise((resolve, reject) => {
            $.ajax({
                url: 'https://www.easy-mock.com/mock/5a52256ad408383e0e3868d7/lagou/position-list',
                success (res) {
                    resolve(res)
                }
            })
        })
        return aaa
    }

    // 第四步：根据上一个API的id(findPostionId)找到具体公司，然后返回公司详情
    function companyInfo (id) {
        let companyList = new Promise((resolve, reject) => {
            $.ajax({
                url: 'https://www.easy-mock.com/mock/5a52256ad408383e0e3868d7/lagou/company',
                success (res) {
                    let comInfo = res.filter(item => {
                        if (id == item.id) {
                            return item
                        }
                    })[0]
                    console.log(comInfo)
                }
            })
        }
    }
```

```javascript
// Promise chain 链
doSomething().then(function(result) {
  return doSomethingElse(result);
})
.then(function(newResult) {
  return doThirdThing(newResult);
})
.then(function(finalResult) {
  console.log('Got the final result: ' + finalResult);
})
.catch(failureCallback);
```

##### ES7 async await 协程

async 用于**定义异步函数**，该函数一定会**返回Promise**。

await 操作符用于**等待一个Promise 对象**。它只能在异步函数 async function 中使用。

###### 比较 Promise和 async

```javascript
function getProcessedData(url) {
  return downloadData(url) // returns a promise
    .catch(e => {
      return downloadFallbackData(url)  // returns a promise
        .then(v => {
          return processDataInWorker(v); // returns a promise
        }); 
    })
    .then(v => {
      return processDataInWorker(v); // returns a promise
    });
}
```

```javascript
async function getProcessedData(url) {
  let v;
  try {
    v = await downloadData(url); 
  } catch (e) {
    v = await downloadFallbackData(url);
  }
  return processDataInWorker(v);
}
```

#### \==和===的区别

\==：仅比较值

===：比较地址，即比类型和值

计算每个数字有多少个

```javascript
let arr = [1, 1, 1, 1, 2, 3, 1, 3, 4, 2, 3, 1, 3, 4, 2];
function count(arr) {
    let c = {};
    for (let i = 0; i < arr.length; i++) {
        let a = arr[i];
        c[a] === undefined ? c[a] = 1 : c[a]++;  // 三元表达式  // 对象索引操作
    }
    return c;
}
result = count(arr);
console.log(result)
```

#### 闭包

闭包就是能够读取其他函数内部变量的函数。

由于在Javascript语言中，只有函数内部的子函数才能读取局部变量，因此可以把闭包简单理解成"定义在一个函数内部的函数"。

```javascript
function f1(){
　　　　var n=999;
　　　　function f2(){
　　　　　　alert(n);
　　　　}
　　　　return f2;
　　}
　　var result=f1();
　　result(); // 999
```

### Node 结合 ES6的语法特点

#### Node 的异步操作

V8 引擎是单线程运行的，一次只能运行一个任务。这导致 Node 大量采用异步操作。

>sync 同步(阻塞)
>
>```javascript
>var fs = require("fs");
>var data = fs.readFileSync("./data.txt");
>console.log(data.toString());
>console.log("done");  
>// data 
>// done
>```
>
>async 异步 (非阻塞)
>
>异步操作：asynchronous operation，即任务不是马上执行，而是插在任务队列的尾部，等到前面的任务运行完后再执行。
>
>```javascript
>// 这里有个回调函数作为参数， 回调函数，就是在文件处理完再回过头来调用，配合异步使用的  
>//回调函数 第一个参数是err错误信息，第二个参数的数据
>var fs = require("fs");
>fs.readFile('./data.txt', function(err, data) {
>if(err) {
>console.log(err)
>}
>console.log(data.toString());
>})
>console.log("done");
>// done
>// data 
>```

#### Node 的模块化开发

模块是什么？：一个模块就是一个文件，是一个独立的作用域，对其他的文件都是不可见的

解决什么问题？：**解决js文件之间的复杂依赖关系而导致的 变量名冲突问题**

在 nodejs 中，模块之间相互独立，模块组成包，包组成应用。

##### CommonJS 模块规范

nodejs 采用的 **模块化规范** 是 CommonJS 模块规范

###### 模块的导出

```javascript
console.log(module); // module变量：指向当前模块，标识每个模块的信息
var a = 10;
module.exports.param = a  // exports属性：指向对外暴露的对象
```

我们要在对外暴露的对象上再加上属性来使用，如果不加上属性，只能保存一个变量

exports：指向 module.exports，不能进行赋值操作，因为 exports 是引用，进行赋值操作就指向了其他地方了

###### 模块的导入

```javascript
var demo = require('./demo07.js')  // 加载模块，js后缀可以省略
console.log(demo.param);  // 加载某个模块，加载的就是module.exports实例，这个属性指向要传递的对象
```

#### JS文件 导入导出机制 (模块化规范、重)

exports / module.exports：CommonJS 规范
export default：ES6 规范，只能用一次，导出一个默认模板，可以匿名
export{}：ES6 规范，导入导出变量名要一致
@ 是src路径的别名alias



后端，Node.js 环境采用的模块化规范是 CommonJS 规范
前端，ES6规范，使用的是 import 和 export default

前端导入导出如下：

```
<script>
	import FooterNav from '@/components/footer'
</script>

<script>
name: 'xxx';
export default {
    data() {
        return {
        }
    }
}

// 要在导出中注册私有组件后，才能在导出的模板中使用该组件！！
components: {
    FooterNav
}
</script>
```

##### Node模块分类

* 核心模块
      fs：					文件操作模块
      http：				网络操作模块
      path：				路径操作模块
      url：					解析地址的模块
      querystring：	解析参数字符串的模块
    使用方法：字节通过 require 加载即可
* 自定义模块
  使用方法：使用相对路径加载模块即可
* 第三方模块
      社区或个人提供，如：jquery、mysql
    使用方法：1.npm去下载  2.引用使用

##### HTTP模块的使用

```javascript
// 初步实现服务器功能

// 1.加载http模块
const http = require("http");
console.log(http);

// 2.创建服务器的实例对象  该对象 用来接收请求并做出响应
let server = http.createServer();

// 3.通过事件绑定机制（node是基于事件驱动的） 第一个参数事件，第二个参数回调函数
server.on('request', (req, res) => {
    res.end('hello');  // 在页面上输出
});

// 4. 监听端口 开3000端口
server.listen(3000);

// 使用链式编程，简化代码
const http = require("http");

http.createServer((req, res) => {
    console.log(req);
    res.end("你好");  // 响应
}).listen(3000, "192.168.0.103", () => {  // 相当于server调用listen方法
    console.log("running...");
});

/*
    1.req 请求 http.inComingMessage 的实例对象 它可用于访问响应状态，消息头，以及数据，由http.Server或http.ClientRequest创建，它作为第一个参数传递给request事件和response事件
    2.res 响应 http.serverResponse 的实例对象 此对象由HTTP服务器在内部创建，而不是由用户创建，它作为第二个参数传递给request事件
*/
const http = require("http");

http.createServer((req, res) => {
    // req.url 可以获取url中的路径(端口之后的部分)
    if (req.url.startsWith("/index")) {
        // end 用来结束响应的方法 也可以页面输出一次内容 只能执行一次
        // res.end(req.url);
        res.write("hello");
        res.write('world');  // 向页面响应内容 可以执行多次 需要注意的是 如果响应没有结束则无输出结果
        res.end();  // 响应结束
    }
}).listen(3000, "192.168.0.103", () => {
    console.log("listenting");
});
```

##### MYSQL模块的使用

[Node-MySQL  API文档](https://www.oschina.net/translate/node-mysql-tutorial)

```javascript
// 注意 SQL注入不要有嗷
// node连接数据库
// 1. 加载mysql包
const mysql = require('mysql');

// 2. 创建数据库连接对象 确定数据库配置信息
let conn = mysql.createConnection({
    host: 'localhost',
    post: '3306',
    user: 'root',
    password: '123',
    database: 'icake'
})

// 3.连接数据库
conn.connect();

// 4. 定义sql语句
let sql = 'select * from cake';

// 5.执行查询   query()   第一个参数是sql语句     第二个参数  回调函数
conn.query(sql, (err, data) => {
    if (err) {
        console.log(typeof(err));
        console.log(err);
    } else {
        console.log(data.toString());
        console.log(typeof(data));
    }
});

// 6.关闭连接
conn.end();

升级版 使用连接池！！！并封装
// 对象的创建和销毁十分消耗资源 ==》数据库连接池
// 数据库连接池 始终将连接对象维护在缓存中

const mysql = require("mysql")

// 数据库连接池的创建
const pool = mysql.createPool({
    host: "localhost",
    user: "root",
    password: "123",
    database: "db_test"
})

// 封装一个抽象的方法
// 原因：多次以不同的sql语句访问数据库 select name from test1 where id=?
const query01 = function(sql, sqlParams, callback) {
    // 获取数据库连接
    pool.getConnection((err, conn) => {
        if (err) {
            // 连接错误的回调
            callback(err, null, null)
        } else {
            // 连接成功的回调   
            conn.query(sql, sqlParams, (error, results, fields) => {
                // error: 错误信息 results: 查询结果 fields: 字段信息
                // 释放资源
                conn.release();
                callback(err, results, fields)
            })
        }
    })
}

module.exports = query01

使用封装好的数据库连接池
const query = require("../db.js")

router.post("/register", (req, res) => {
    let sql = "SELECT * FROM USER_INFO WHERE EMAIL = ?"    
    let sqlParams = [
        req.body.email
    ]
    query(sql, sqlParams, (err, result, fields) => {
        if (err) {
            res.json({
                ok: false,
                err: err
            })
        } else { 
            // console.log(err)   
            // console.log(result) 
            // console.log(fields) 
            res.json({
                ok: false,
                msg: "hello hi"
            })
        }
    })
})
```

###### 传统动态网站开发需要的应用

PHP：Apache + php模块
Java：Tomcat + Weblogic
