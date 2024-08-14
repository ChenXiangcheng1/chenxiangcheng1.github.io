# ECMAScript

>历史：
>1996 年 11 月，JavaScript 的创造者 Netscape 公司(倒闭已被收购)，决定将 JavaScript 提交给标准化组织 ECMA，希望这种语言能够成为国际标准。次年，ECMA 发布 262 号标准文件（ECMA-262）的第一版，规定了浏览器脚本语言的标准，并将这种语言称为 ECMAScript，这个版本就是 1.0 版。JavaScript 是语言，ECMAScript 是标准化组织规定的**标准**。

ECMAScript 的新特性由 ECMA TC39 委员会进行组织工作
Proposals提案：[tc39/proposals](https://github.com/tc39/proposals)

>- Stage 0/Strawperson： 潜在的可能被纳入规范的一些想法。
>- Stage 1/Proposal：为该想法设想一些适用场景，可能的 case。提出解决实现方案以及可能的变更。
>- Stage 2/Draft：经过上一步验证讨论后，这一阶段开始起草语言层面的语义语法，准备正式的规范文档。
>- Stage 3/Candidate：提案进入到了候选阶段。开始接收一些反馈对提案进行完善。
>- Stage 4/Finished：可以被纳入到正式的 ECMAScript 语言规范中了。



## ECMAScript2015 (ES6)

### 对象

对象由多组键值对组成，其中key可以是数字，value可以是function

```javascript
let obj1 = {
    param1: "111",
    myfunc: () => {  // function value
        console.log(this.param1);
        setTimeout(function() {
            console.log(this.param1)
        }, 1000);
    },
    num: 5,
    str: "123",
    1: 12  // 数字键
}
console.log(obj1[1]); // 输出数字键要使用[]
obj1.myfunc();  // 其他键使用.
```



### 修饰符

|       | ~~不用修饰符~~ | ~~var~~                                                      | let            | const                        |
| ----- | -------------- | ------------------------------------------------------------ | -------------- | ---------------------------- |
| 语义  |                |                                                              |                | 声明只读常量，必须初始化赋值 |
| 机制  |                | 变量提升                                                     | 不存在变量提升 | 不存在变量提升               |
| scope | 全局作用域     | 在函数外声明为全局作用域(会变量泄露)<br />在函数内声明为局部作用域 | 块级作用域     |                              |
|       |                |                                                              |                |                              |

> **变量提升 (提前声明)**：将当前作用域 (全局、函数作用域) 的所有变量的声明提升到程序的顶部，值为 undefined
>
> **变量泄露**：
>
> - 在for(var i=0; i<1; i++)循环中 var 声明的变量在全局范围内都生效，所有的 变量i 都指向同一个 全局变量i ，调用方法时输出的值就是最后一轮 i 的值
> - 而let声明的变量在仅在块级作用域内生效，所以每一次for循环都生成了一个新的变量i，调用方法时输出的值是不同对象的值



### 解构赋值

对数组解构赋值

```ts
let [x = 100, y, z] = [200, 123, ];
```

对对象解构赋值

```ts
let {k1:othername, k2="default"} = {k1: "v1"};
```

对参数应用解构赋值

```ts
function foo({name="zhangsan", age="18"} = {}) {
    console.log(name + "  " + age);
}
foo({name: "lisi", age: "20"});
```



### ...展开

```javascript
function foo(a, b, ...param) {  // rest参数(剩余参数)
    console.log(a);
    console.log(b);
    console.log(...param);
}
foo(1, 2, 3, 4, 5);
```



### `==`和`===`的区别

| ==     | ===      |
| ------ | -------- |
| 比较值 | 比较地址 |



### 可选链运算符

```ts
obj?.a  // obj空不报错
obj ?? b  // obj空返回b
```



### 拓展了字符串

```javascript
includes()
startsWith()
endsWith()
```



### 问题

#### this指向问题

| 环境                         | windows      | node   |
| ---------------------------- | ------------ | ------ |
| setTimeout的顶层对象(调用者) | dom的 window | global |

问题：setTimeout()方法中回调函数中的this不是指向定义的对象本身，而是指向方法调用者
原因：this 指向的是方法的调用者
解决：在setTimeout() 外部将要引用的对象this进行共享(保存this再调用)

```ts
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

let obj1 = {
    param1: "111",
    myfunc: () => {
        console.log(this.param1);
        setTimeout(() => {
            console.log(this.param1)  // this指向obj1
        }, 1000);
}	}
obj1.myfunc();
```



#### 回调地狱问题

问题：回调代码很长、需要链式传递参数、需要链式回调
原因：函数是无状态的，对象是有状态的
解决：使用Promise对象

回调地狱问题示例1：

```javascript
doSomething(function(result) {
  doSomethingElse(result, function(newResult) {
    doThirdThing(newResult, function(finalResult) {
      console.log('Got the final result: ' + finalResult);
    }, failureCallback);
  }, failureCallback);
}, failureCallback);
```

解决：通过Promise对象

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



回调地狱问题示例2：

```javascript
// 回调地狱
/***
   第一步：找到北京的id
   第二步：根据北京的id -> 找到北京公司的id
   第三步：根据北京公司的id -> 找到北京公司的详情
   目的：模拟链式调用、回调地狱
 ***/
 // 请求第一个API: 地址在北京的公司的id
 $.ajax({
   url: 'https://www.easy-mock.com/mock/5a52256ad408383e0e3868d7/lagou/city',
   success (resCity) {
     let findCityId = resCity.filter(item => {
       if (item.id == 'c1') {
         return item
       }
     })[0].id
     //  请求第二个API: 根据上一个返回的在北京公司的id “findCityId”，找到北京公司的第一家公司的id
     $.ajax({       
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
 })}})} }) }
```

解决：通过Promise对象

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



### timer

| 执行异步回调的三种方法签名     | 释义                                |
| ------------------------------ | ----------------------------------- |
| `setTimeout(function, delay)`  | 设置一个timer，到期后执行(延迟执行) |
| `setInterval(function, delay)` | 循环timer，返回 intervalID          |
| `clearInterval(intervalID)`    | 根据intervalID取消定时器            |

```javascript
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



### Promise对象

Promise对象表示一个异步操作

```ts
new Promise( // new时，不用调用，Promise中的异步函数就已经执行了
  function (resolve, reject) {
    resolve('')  // 当异步函数执行成功，Promise状态由pending变为fulfilled，并将异步操作的结果作为参数传递出去
    reject('数据处理出错')  // 当异步函数执行失败，Promise状态由pending变为rejected，并将异步操作抛出的错误作为参数传递出去
  }
).then(
  (res) => {console.log(res)},  // 成功，参数是resolve返回的结果
  (err) => {console.log(err)} // 失败，参数是resolve返回的结果
)
```



### async await(重要)

协程Coroutine(协同程序)

与多线程的共同点：协程类似多线程，调用顺序不明确

与多线程的区别：
协程相比多线程更轻量，因为协程是由程序自身控制的一个线程(意味着不需要锁)，没有线程切换开销



async 用于**定义异步函数**，该函数一定会**返回Promise**
await 用于**等待一个Promise 对象**。它只能在异步函数中使用



Promise 写法：

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
}	});
```

async await 写法：

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



### 闭包概念

闭包是一个函数，是一个用于读取其他函数内部变量的函数

因为在 JS 中，只有函数内部的子函数才能读取局部变量
所以在 JS 中，在**函数内部的子函数**就是闭包

```ts
function fun(){
  var n = 999;
  function subfun(){  // 这就是闭包
    console.log(n);
  }
  return subfun;
}

var result = fun();
result(); // 打印：999
```


