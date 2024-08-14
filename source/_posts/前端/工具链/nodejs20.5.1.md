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

[Node.js ECMAScript2015 兼容性表](https://node.green/#ES2015)

[Node.js ECMAScript2024 兼容性表](https://node.green/#ES2024)



# 未整理

todo：目前了解 npm，有空再看后端 node.js 官网文档



# Node

An asynchronous event driven JavaScript runtime designed to build scalable network applications.

[官网](https://nodejs.org/en)



### Node环境执行JavaScript

| 操作                                     | 释义                                               |
| ---------------------------------------- | -------------------------------------------------- |
| vscode 中：ctrl + `（是键盘上esc下面的） | 或者 查看——>终端打开 终端<br/>最好进入js的那个目录 |
| node .\demo.js                           | 执行                                               |
| node                                     | 进入node环境                                       |
| .exit                                    | 退出node环境                                       |



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
