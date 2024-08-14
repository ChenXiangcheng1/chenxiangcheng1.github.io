# Vue.js2

开箱即用的解决方案：[Vue Antd Admin文档](https://iczer.gitee.io/vue-antd-admin-docs/start/use.html#%E5%AE%89%E8%A3%85)

### Vue简介

Vue.js 与React.js 都是前端的主流框架

提升开发效率经过了哪几个阶段：
原生js  ==>   jquery类库  ==>   模板引擎  ==>  框架(软件的半成品)
document.getElementByID  ==》 $("#app")   ==> 将数据和页面分离

原生js：跑在不同的浏览器引擎中，有兼容问题
jquery类库：解决了兼容问题且简化了代码
模板引擎：将数据和页面分离，实现解耦   高内聚 低耦合
框架： 开发人员不用再担心数据的渲染，不用再操作DOM元素了  (Vue指令实现)

vue 编程范式：声明式
响应式编程范式：数据改变，页面也改变

### vue.js安装

方式1：CDN（Content Delivery Network）内容分发网络引入，从最近的服务器、缓存中获取

```html
<script src="http://xxx"></script>
```

方式2：下载js文件引入
方式3：NPM安装，通过webpack和CLI使用

`cnpm i vue-cli -g --save`

### MVVM思想

开发的分层思想
Model-View-ViewModel
【M模型】 后端传递的数据
【V视图】 所看到的页面视图
【VM视图模型】 MVVM的核心  是连接View，Model的桥梁    观察者
    模型=>视图   将数据转换成我们所能看到的页面   	**数据绑定**
    视图=>模型   将页面转换成后台的数据          			**事件监听**
这让程序员的关注点从如何操作DOM变成了如何更新JavaScript对象的状态



### Vue的基本使用

```html
<!-- 1.导入Vue的包 -->
// 当我们导入Vue的包  在浏览器的内存中 就会多一个Vue的构造函数，生命周期
<script src="lib/vue-2.4.0.js"></script>
</head>
<body>
    <!-- DIV属于View层，用于将后台数据渲染到前台页面视图  -->
    <div id="app">
        <!-- Vue的指令（插值表达式）把数据渲染到页面上，不需要再去操作dom元素了 -->
        {{ text}}
    </div>
</body>

<script>
// 2. 创建Vue实例对象
var vm = new Vue({  		// 该**构造函数**需要传入一个Object
    el: '#app',     		// **Vue实例对象所要控制的页面视图区域**
    data: {         		// data属于Model层，用于存放el中所需的数据        			
        text: '今天天气真好' 	// 这些数据可以是我们定义的，也可以是来自网络，从服务器加载的
    }
});
// var app = document.getElem....  	// 原始的写法
// app.innerText = '今天天气真好'
</script>
```


### Vue生命周期

```vue
<div id ="app"> 
    {{ msg }}
    <button @click="foo1">执行foo1方法</button>
</div>

<script>
// 1. new Vue() 新建Vue实例，内存创建了空间，但还未初始化
 var vm = new Vue({
    // 4. 判断是否指定"el"元素选项    
    el: '#app',
    data: {
        msg: '生命周期钩子函数',
    },
    methods: {
        foo1() {
            console.log("foo1函数执行。。");
            this.msg = "haha";
        },
    },
    // 2. 初始化默认事件，
    beforeCreate() {
        console.log("beforeCreate: " + this.msg);
    },
    // 3. 注入实例的属性数据，vue实例初始化完成
    created() {
        console.log("created: " + this.msg);
    },
    // 5. 指定了el元素时，当调用vm.$mount(el)挂载函数时，判断是否指定了"template"模板选项。是，将template编译到render渲染函数中；否，将el外部的HTML作为template编译
    beforeMount() {
        // 表示模板已经渲染好，但是vue模板还未替换掉原始模板  因此通过DOM得到文本内容还是原始的文本
        var msg = document.getElementById("app").innerHTML;
        console.log("beforeMount: " + msg);
    },
    // 6. 创建vm.$el并用其替换"el"
    mounted() {
        // 表示模板已经渲染好，且vue已经替换调原始模板了  因此通过DOM得到文本内容就是替换好了的文本
        var msg = document.getElementById("app").innerHTML;
        console.log("beforeMount: " + msg);
    },
    // 7. 当data被修改时,beforeUpdate 虚拟DOM重新渲染
    beforeUpdate() {
        // 表示data变化，view要更新之前   此时数据还没替换上去
        var msg = document.getElementById("app").innerHTML;
        console.log("beforeMount: " + msg);
    },
    // 8. updated应用更新
    updated() {
        // 表示data变化，view更新完成后
        var msg = document.getElementById("app").innerHTML;
        console.log("beforeMount: " + msg);
    },
    // 9. 当调用vm.$destroy()函数时,beforeDestroy 解除绑定销毁子组件以及事件监听器
    // 10. 销毁完毕destroyed 如果使用构造生成文件 （例如构造单文件组件），模板编译将提前执行
 });
</script>
```



### Vue中插值表达式与基础指令

Vue指令的使用必须放在标签里 

#### 插值表达式

`{{Vue对象的data属性中的键名}}`

Vue通过插值表达式把数据渲染到页面上，不需要再去操作dom元素了



插值表达式的闪烁问题：因为浏览器引擎先加载并渲染msg文本，再下载vue.js，再创建Vue实例对象后才能调用插值表达式渲染替换msg，所以为出现表达式的闪烁

##### vue过滤器

过滤器要定义在上面，优先执行，对插值表达式中的原始字符串进行过滤

```vue
<div id ="app"> 
    {{ text | filter01('~') }}<br/>
    {{ text | filter01('~') | filter02('!') }}
</div>

<script>
    
// 创建 Vue 实例之前全局定义过滤器
// Vue.filter() 定义一个过滤器 参数1：过滤器名字，参数2：回调函数
// 回调函数 参数msg：原始字符串  参数arg：表示传递的参数
    
Vue.filter('filter01', function(msg, arg) {
    // str.replace(old, new) new替换掉str中的old
    return msg.replace(/\?/, arg);
});
Vue.filter('filter02', function(msg, arg) {
    // str.replace(old, new) new替换掉str中的old
    return msg.replace(/~/, arg);
});
    
var vm = new Vue({
    el: '#app',
    data: {
        'text': 'hello?',
    },
 });
</script>
```

##### 计算属性computed

使用计算属性 == 使用带缓存机制的有返回值的方法

```vue
 <div id ="app"> 
    {{ name.substring(0, 1).toUpperCase() + name.substring(1) }}
    <hr/>
    计算属性{{ upperName }}
    <hr/>
    方法{{ upperName1() }}
    <hr/>
    计算属性{{ upperName }}
    <hr/>
    方法{{ upperName1() }}
</div>

<script>
 var vm = new Vue({
    el: '#app',
    data: {
        name: 'michael jordan',
    },
    methods: {
        upperName1() {
            console.log("我是方法");
            return this.name.substring(0, 1).toUpperCase() + this.name.substring(1);
        }
    },

    // 定义一个计算属性
    // 必须要有返回值
    // **缓存机制**     计算属性会将结果存入内存中，下次进行计算时如果没有改变  就用内存中缓存值
    computed: {
        upperName() {
            console.log("我是计算属性");
            return this.name.substring(0, 1).toUpperCase() + this.name.substring(1);
        }
    },
 });
</script>
```

#### v-cloak指令

cloak：披风，遮盖物



当数据还未渲染到页面上时，属性存在在标签上；当数据渲染到页面上时，属性消失。可以根据这个特性 去改变样式 去解决插值表达式闪烁的问题

```vue
    <script src="./lib/vue-2.4.0.js"></script>

    <!-- 指定css样式 -->
    <style>
    [v-cloak] {
      display: none;
    }
    </style>
</head>
<body>
    <div id ="app">
        {{msg}}        
        <p v-cloak>{{msg}}</p>
    </div>
    <script>
    setTimeout(function() {
        var vm = new Vue({
            el: '#app',  	// 这里是css选择器中的id选择器
            data: {         // 存放用到的数据
                msg: 'hello',
            },
            methods: {      // 存放用到的方法
                fun1: function() {
                    alert("hello");
                    this.msg = "haha";  // 要用this访问自己对象的字段
                }
            }
        });
    }, 5000);
    </script>
```

#### v-text指令

​    引号内**作为变量进行解析**，**覆盖**掉元素中原本的**文本内容**；使用插值表达式不会覆盖，只会替换调自己的**占位符**
​     v-text=“msg”

#### v-html指令

字符串**作为 html进行解析**，会创建子标签

v-html="msg"

#### v-bind指令

**数据的单向绑定** Modle => View，在绑定的字符串**作为 JS 表达式解析** 

v-bind:标签属性="dataModel + 'str'" 或者 :标签属性="dataModel + 'str'"

##### Vue操作样式

###### 操作class

标签的class属性：指向预设置的类样式，可以合单向绑定的 JS表达式，样式扩展性强，可以进行逻辑判断

```vue
<style>
    .red {
        background-color: red;
    }
    .green {
        background-color: green;
    }
    .size {
        width: 200px;
        height: 200px;
    }
</style>

// 三元表达式，控制选择类名
<div :class="['size', flag?'green':'red']"></div>
// {}，控制显示不显示
<div :class="['size', {'green': flag}]"></div>
// 使用vm.data对象直接操作样式
<div :class="obj"></div>

data: {
    flag: true,
    obj: {'size': true, 'red': true},
},
```

###### 操作style

style：直接写在标签内部，想要引用多个style  将它们写在数组中

原生方式 行内样式

```html
<div style="background-color: red; width: 300px; height: 300px;"></div>
```

vue 方式

```vue
<div :style="[obj1, obj2]">123</div>
```

想要引用多个style  将它们写在数组中

```vue
data: {
    obj1: {
        "background-color": "green",
        "width": "300px",
        "height": "300px",
    },
    obj2: {
        "color": "white",
    }
},
```

#### vue条件渲染标签

控制元素的显示或隐藏

##### v-if

通过渲染不渲染控制，dom操作

true：每次都会创建元素

false：每次都会删除元素

##### v-show

通过样式，无dom操作，性能更好

添加 display: none 样式，不会进行删除元素的操作

```vue
<div id ="app"> 
    <button @click="fun1">切换</button>
    <div id="box1" v-if="flag">我是v-if指令</div>
    <div id="box2" v-show="flag">我是v-show指令</div>
</div>

<script>
 var vm = new Vue({
    el: '#app',
    data: {
        flag: true,
    },
    methods: {
        fun1() {
            this.flag = !this.flag;
        }
    },
 });
</script>
```

#### v-on指令

**事件绑定**

`v-on:click="fun1"` 或者 `@click="fun1"` 或者`@click[.事件修饰符] = "fun1"`

##### DOM 事件流

经历了3个阶段
**事件捕获**阶段(从外部到内部)  ==>  **目标阶段**(触发点击事件)  ==>  **事件冒泡**阶段(从内到外  从点击元素，按事件触发顺序    子元素->父元素)

v-on指令可以结合Dom事件流使用

`@click[.事件修饰符] = "fun1"`

事件修饰符
stop        	   阻止事件冒泡，从根源上阻止了冒泡行为的触发
prevent         阻止事件默认行为，如阻止a标签的超链接默认跳转行为
capture         实现事件捕获机制，会捕获传递触发事件，事件触发顺序    父元素->子元素
self        	     当前元素触发事件，阻止自己身上的冒泡行为的触发   只能阻止本身不受冒泡行为的波及
once        	  只触发一次事件，如只执行一次阻止默认行为，@click.prevent.once="fun1"

#### v-model指令

**数据的双向绑定**

```vue
<input type="text" v-model="msg">
```

可以实现数据的双向绑定   必须结合表单元素 (或可以输入值的情况, input, )

#### v-for

**遍历数组，对象，数字**，能够生成子元素

```vue
<div id ="app"> 
    <p v-for="(item, index) in arr">{{item}}:{{index}}</p>
    <p v-for="(value, item, index) in obj">{{value}}:{{item}}:{{index}}</p>
    <p v-for="index in 5">{{index}}</p>
</div>

<script>
 var vm = new Vue({
    el: '#app',
    data: {
        arr: ['11', '22', '33'],
        obj: {
            1: '11',
            2: '22',
            3: '33',
        }
    }
 });
</script>
```
#####  单选框就地复用原则

就地复用原则：为了提高渲染效率，使数据和单选框分离，单选框的选中情况不随数据改变而改变
解决：key="item"，绑定子元素和数据，key不能是index，因为index会变化，key要是唯一

```vue
// list添加、删除
<div id ="app"> 
    <input type="text" v-model="text"/>
    <button @click="add">添加</button>
    <ul>
        <li v-for="(item, index) in list" :key='item'>
            {{ item }}:{{ index }}
            <button @click="del(index)">删除</button>
            <input type="checkbox"/>
        </li>
    </ul>
</div>

<script>
 var vm = new Vue({
    el: '#app',
    data: {
        list: [],
        text: 'hello',
    },
    methods: {
        add() {
            this.list.push(this.text);
        },
        del(index) {
            this.list.splice(index, 1);
// [] splice方法 API 介绍：   
// let arrDeletedItems = array .splice（start [，deleteCount [，item1 [，item2 [，...]]]]）
            // splice() 方法向/从数组中添加/删除项目，然后返回被删除的项目。
            // index   必需。整数，规定添加/删除项目的位置，使用负数可从数组结尾处规定位置。
            // howmany 必需。要删除的项目数量。如果设置为 0，则不会删除项目。
            // item1, ..., itemX   可选。向数组添加的新项目。
        },
    }
 });
</script>
```

### eval() 方法

将字符串解析成表达式

### 监听属性Watch

```vue
<div id ="app"> 
    单价：<input type="text" v-model="price"/>
    数量：<input type="text" v-model="num"/>
    总价：{{ sum }}
</div>

<script>
 var vm = new Vue({
    el: '#app',
    data: {
        price: 0,
        num: 0,
        sum: 0,
    },
    
    watch: {
        // 要监听谁就写哪个属性的方法
        price() {
            console.log("price改变了", this.price);
            this.sum = this.price * this.num;
        },
        num() {
            console.log("num改变了", this.num);
            this.sum = this.price * this.num;
        }
    }
 });
</script>
```

利用监听属性，实现筛选list列表

```vue
  <div id ="app"> 
    <input type="text" v-model="name">
    <ul>
      <li v-for="data in list">
        {{ data }}
      </li>
    </ul>
  </div>

  <script>
   var vm = new Vue({
    el: '#app',
    data: {
      list:["张三","张无忌","张三丰","李四","李斯","李世民"],
      name: ""
    },
    watch: {
      // 实现监听name，list包含name的返回，否则不返回
      name() {
        // list.filter(回调函数)方法，过滤数组元素, 数组元素执行回调函数返回为true 会返回该元素
        var newList = this.list.filter(item => {
          console.log(this)
          return item.indexOf(this.name) > -1;
        })
        this.list = newList;
      }
    }
   });
  </script>
```

使用回调函数解决监听器中函数的this的指向问题

function 指向 Windows
箭头函数 指向 Vue$3



handler				// 处理器属性
deep: true  		// **深度监听**，可以监听到对象内部属性的改变，可以针对 docData.doc_id 对象的某个属性监听
immediate: true  // 在最初绑定值的时候执行 handler 函数

# 以下还未整理

## vue自定义指令

全局注册：在任何组件中都可以使用全局注册的自定义指令
局部注册（私有指令）：只有在当前组件中才能使用

使用
把需要传递的值放在‘=’号后面传递过去
指令:click=”handClick” `为指令传递参数’click’
可以使用指令.修饰符 使用修饰符
可以什么都不传递。

```vue
<div id ="app"> 
    <input type="text" v-dir="'hello'" v-model="text"/>
    <button @click="text='111'">改变text属性</button>
</div>

<script>
// 指令的定义   参数1：指令名字     参数2：对象，内部存放钩子函数
// v-dir
Vue.directive('dir', {
    // 每当指令绑定元素上时，就会立即执行这个bind函数 只执行一次 在这里可以进行一次性的初始化设置 元素样式。
    // el 表示当前指令绑定的元素，binding 表示指令传递的值, vnode 表示虚拟节点内部存放一系列相关信息
    bind(el, binding, vnode) {
        console.log(el, binding, vnode);
    },
    // 被绑定元素插入父节点时调用 (仅保证父节点存在，但不一定已被插入文档中)。 插入节点后用于DOM操作
    inserted(el, binding, vnode) {
        console.log('此时DOM节点创建', el, binding, vnode);
    },
    // 所在组件的 VNode 更新时调用，但是可能发生在其子 VNode 更新之前。指令的值可能发生了改变，也可能没有。但是你可以通过比较更新前后的值来忽略不必要的模板更新 
    update() {
        console.log("绑定的状态改变");
    },
});
     var vm = new Vue({
    el: '#app',
    data: {
        text: "hi",
    },
 });
</script>
```


====================================================================================

综合案例，设计list的遍历方法，label标签的设置方法，事件监听机制

```vue
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <script src="../lib/vue-2.4.0.js"></script>
</head>
<body>
    <div id ="app"> 
        <div>
            <h3>书籍管理</h3>
        </div>
        <div>
            <label>
                ID: <input type="text" v-model="id"/>
            </label>
            <!-- label标签的第二种方法 -->
            <!-- <label for="name">书名: </label>
            <input id="name" type="text"> </input> -->
            <label>
                &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
                书名: <input type="text" v-model="name"/>
            </label>
       		<button @click="add">添加</button>
            <label>
                &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
                搜过关键字: <input type="text" v-model="keywords"/>
            </label>
            <table>
                <!-- 表头thead table head -->
                <thead>
                    <tr>
                        <th>ID</th>
                        <th>Name</th>
                        <th>CTime</th>
                        <th>Operation</th>
                    </tr>
                </thead>
                <tbody>
                    <!-- <tr v-for="item in search(keywords)" :key="item.id"> -->
                    <tr v-for="item in newArr" :key="item.id">
                        <td>{{ item.id }}</td>
                        <td>{{ item.name }}</td>
                        <td>{{ item.ctime | dateFormat }}</td>
                        <td>
                            <a href="" @click.prevent="del(item.id)">删除</a>
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
    </div>

    <script>
    // 定义全局过滤器dateFomat， 用于展示正常的时间格式
    Vue.filter("dateFormat", function(msg) {
        // Sat Apr 18 2020 09:51:49 GMT+0800 (中国标准时间)   ==>  2020-04-18 9:51:20
        var year = msg.getFullYear();
        // dateobj.getMonth() 返回月份-1
        var month = msg.getMonth() + 1;
        var day = msg.getDate();
        return year + "-" + month + "-" + day;
    });

     var vm = new Vue({
        el: '#app',
        data: {
            list: [
                {id: 1, name: "CSS权威指南", ctime: new Date()},
                {id: 2, name: "JAVA核心技术卷", ctime: new Date()},
            ],
            id: '',
            name: '',
            keywords: '',
            newArr: []
        },
        methods: {
            add() {
                this.list.push({id: this.id, name: this.name, ctime: new Date()});
                this.id = '';
                this.name = '';
            },
            del(id) {
                // 第一种方法：原始for循环遍历数组
                // for (var i = 0; i < this.list.length; i++) {
                //     if (id == this.list[i].id) {
                //         this.list.splice(i, 1);
                //     }
                // }

                var index = this.list.findIndex(item => {
                    if (item.id == id) {
                        return true;
                    }
                });
                this.list.splice(index, 1);
            },
            search(keywords) {
                var newArr = [];
                this.list.forEach(item => {
                    if (item.name.indexOf(keywords) > -1) {
                        newArr.push(item);
                    }
                });
                return newArr;
            },
        },
        watch: {
            keywords: {
                //
                handler: function() {
                    console.log(this.keywords)
                    this.newArr = [];
                    this.list.forEach(item => {
                        console.log(item.name.indexOf(this.keywords))
                        // 如果keywords为空，该方法会返回0
                        if (item.name.indexOf(this.keywords) > -1) {
                            this.newArr.push(item);
                        }
                    });
                },
                // immediate设为true    监听方法会在创建的时候  执行handler里的方法
                immediate: true         
            }
        }
     });
    </script>
</body>

</html>
```

## vue组件化开发

将公共的代码抽离出来，封装成一个组件，方便后期复用

组件化开发从UI(User Interface)角度封装
模块化开发从代码逻辑角度封装

```vue
<div id ="app"> 
    <test1></test1>
    <test1/>
</div>
<!-- vue提供的标签 -->
<!-- template标签，必须有且只有一个根标签!!!! -->

<template id="temp1">
    <!-- 组件模板 -->
    <h2>我是一个组件，可以被复用</h2>
</template>

<script>
/*
    组件的定义
        1.定义模板
        2.创建组件
*/
// Vue.component()方法，参数1：组件名，参数2：对象，存放组件的相关信息   
Vue.component("test1", {
    // 模板数据
    template: "#temp1",
});
<script>
/*
    组件的定义
        1.定义模板
        2.创建组件
*/
// Vue.component()方法，参数1：组件名，参数2：对象，存放组件的相关信息   
Vue.component("test1", {
    // 模板数据
    template: "#temp1",
});
 var vm = new Vue({
    el: '#app',
 });
</script>
```

组件的data为何是必须是一个函数？？

```vue
Vue.component('com1', {
    template: '#tem1',
    data: function() {
        // 组件的data，必须的返回一个对象，而不是一个对象的引用
        // 组件是可以复用，因此希望组件的data是独立的，互不影响，所以需要每次复用组件都有新的data对象
        // 因此data是一个方法，每次复用就调用一次
        return {
            num: 0,
        };
    },
});
```

私有组件

```vue
var vm1 = new Vue({
    el: '#app1',
    // 私有组件
    components: {
        com02: {
            template: '#temp2'
        }
    }
 });
```

## 组件的切换

方式1

```vue
<a href="" @click.prevent="flag=true">登录</a>
<a href="" @click.prevent="flag=false">注册</a>
<login v-if="flag"></login>
<reg v-else="!flag"></reg>
```

方式2
```vue
<div id ="app"> 
    <a href="" @click.prevent="name='com1'">组件1</a>
    <a href="" @click.prevent="name='com2'">组件2</a>
    <a href="" @click.prevent="name='com3'">组件3</a>
    <!-- component就是一个占位符，:单向绑定，name展示的组件的名称 -->
    <component :is="name"></component>
</div>

<template id="temp01">
    <div>
        <h2>组件1</h2>
    </div>
</template>
<template id="temp02">
    <div>
        <h2>组件2</h2>
    </div>
</template>
<template id="temp03">
    <div>
        <h2>组件3</h2>
    </div>
</template>
<script>
Vue.component('com1', {
    template: "#temp01",
})
Vue.component('com2', {
    template: "#temp02",
})
Vue.component('com3', {
    template: "#temp03",
})
 var vm = new Vue({
    el: '#app',
    data: {
        name: "",
    },
 });
</script>
```



## 父组件向子组件传值

通过Pass Props
父组件可以使用 props 把数据传给子组件

```vue
<com1 :parentmsg="msg"></com1>
```




组件里面：

```vue
props: {
    parentmsg: {
        type: String,
    },
},

<!-- 
    父组件的数据，子组件不能直接引用
    不管什么组件，组件的作用域都是独立的
 -->
<div id ="app"> 
    <!-- 将要转递给子组件的值，绑定在标签内 -->
	<!-- 传递常量 -->
	<!-- <com1 parentmsg="123"></com1> -->
	<!-- 传递变量 -->
	<com1 :parentmsg="msg"></com1>
</div>

<script>
 var vm = new Vue({
    el: '#app',
    data: {
        msg: "父组件的msg属性",
    },
    methods: {
        foo() {
            console.log("父组件的foo方法");
        },
    },
    components: {
    com1: {
        // 当子组件中data没有值，所用的值是从父组件传递而来的
        data() {
            return {

            }
        },
        // 组件的作用域都是独立的，父组件的数据子组件不能引用，所以msg不能使用
        // template: "<div><h2>这是子组件{{ msg }}</h2></div>",

        // template 也可以定义在component中
        // foo方法还是不能使用
        template: "<div><h2>这是子组件</h2>{{ parentmsg }}<br/><button @click='foo'>点击</button></div>",

        // 子组件需要显示声明预期的数据，prop中的数据是只读  无法重新赋值的
        props: {
            parentmsg: {
                type: String,
            },
        },
    },
},
      });
</script>
```



## 子组件向父组件传值

Emit Events（广播机制，通过事件），可以通过

```vue
$emit/$on/watch
```

子组件可以使用

```vue
 $emit 触发父组件的自定义事件
vm.$emit( event, arg ) //触发当前实例上的事件
vm.$on( event, fn );//监听event事件后运行 fn； 
```



```vue


<!-- 这是父组件的view -->
<div id ="app"> 
    <!-- 父组件接收数据，也是通过事件绑定，监听子组件的方法 -->
    // 在父组件中注册子组件并在子组件标签上绑定对自定义事件的监听
    <child @foo="childValue"></child>
    <hr/>
    <h2>父组件</h2>
    <span>{{ msg }}</span>
</div>
<!-- 这是子组件的view -->
<template id="child01">
    <div>
        <h2>子组件</h2>
        <span>{{ childValue }}</span>
        <!-- 1.子组件中定义一个点击事件，通过事件向父组件传值 -->
        <input type="button" value="点击传值" @click="childClick"/>
    </div>
</template>
<script>
Vue.component("child", {
    template: "#child01",
    data() {
        return {
            childValue: "我是子组件的数据",
        };
    },
    methods: {
        childClick() {
            // 2.在点击事件中，使用$emit去广播数据
            // 子组件向父组件传递值，需要方法广播，参数1：数据的名字，key，也是父组件接收数据事件的方法名，参数2：值，value
            this.$emit("foo", this.childValue);
        },
    },
});
var vm = new Vue({
    el: '#app',
    data: {
        msg: "我是父组件的数据",
    },
    methods: {
        // 3.在组件中定义一个方法，@自定义方法='传递数据的名字'
        childValue(value) {
            this.msg = value;
        },
    },
 });
</script>
```

## vue路由

vue.js除了拥有组件开发体系之外，还有自己的路由vue-router。
没有使用路由前  通过a标签进行跳转页面   ==>  Vue  spa(single page application单页面应用)
路由模块，路由的功能就是 在同一页面 通过不同的url 展示出不同组件

```vue
<!-- 1.引入路由模块 -->
<script src="../lib/vue-router-3.0.1.js"></script>
<div id ="app"> 
    <!-- 路由的功能就是 在同一页面 通过不同的url 展示出不同组件 -->
    <!-- 路由入口 -->
<!-- <a href="#/comm1">Go to com01</a>
<a href="#/comm2">Go to com02</a> -->        
<hr/>
<!-- <a href="#/comm1" class="router-link-exact-active router-link-active">Go to com01</a> -->
<!-- router-link 有class属性的切换，可以利用加样式 -->
<router-link to="/comm1">Go to com01</router-link>
<router-link to="/comm2">Go to com02</router-link>

<!-- 路由出口 路由匹配到的组件都会展示在router-view中 -->
<router-view></router-view>
    </div>

<script>
// 2.定义组件模板对象
var com01 = {template: '<div>路由模板1</div>' };
var com02 = {template: '<div>路由模板2</div>' };

// 3. 定义路由规则 route 内部可以存放多条路由规则 因此是一个数组
var routes = [
    // 路由入口和路由模板连接
    {path: "/comm1", component: com01},
    {path: "/comm2", component: com02},
];

// 4.创建路由实例 路由对象，并导出
var router = new VueRouter({
    // 4.1 引入路由规则
    // routes: routes,  // routes 路由规则属性
    routes,  // ES6新语法 当属性值和属性名相同时，可以简写
});

 var vm = new Vue({
    el: '#app',
    // 5. 将路由实例挂载到路由vm实例中
    router,
 });
</script>    
```

路由传参的两种方式
- 在url中通过`?` 传递参数   此时接受参数  使用$route.query获取参数

```vue
  <router-link to="/comm1?id=1&name=zhangsan">组件1</router-link>
  template: "<div>comm1{{ $route.query }}</div>",
  var vm = new Vue({
    el: '#app',
    router,
    created() {
        // 当vm实例创建好之后，打印路由信息对象
        console.log(this.$route)
        /*
            $route 路由信息对象
                承载着当前路由的状态信息
                params
                query 一个键值对对象 存放URI 查询数据
        */
    }
  });
```

- 在url中通过`/` 传递参数   此时接受参数   需要修改路由规则 使之形成一个完整的键值对

```vue
  <router-link to="/comm1/1/zhangsan">组件1</router-link>
  template: "<div>comm1{{ $route.params }}</div>",
  routes: [
    {path: "/comm1/:id/:name", component: com01},
    {path: "/comm2", component: com02}
  ]
```


## 子路由

在实际开发中  时常会碰到组件中匹配子路由情况

```vue
<div id ="app"> 
      <router-link to="/user">user组件</router-link>
    <router-view></router-view>
</div>
<template id="user">
    <div>
        <!-- 路由入口 -->
        <h2>User组件</h2>
        <router-link to="/user/comm1">组件1</router-link>
        <router-link to="/user/comm2">组件2</router-link>
         <!-- 路由出口 -->
        <router-view></router-view>
    </div>
</template>
<script>
// 定义组件
var user = { template: "#user"}
var com01 = { template: "<div>组件1</div>"}
var com02 = { template: "<div>组件2</div>"}


// 定义路由对象
var router = new VueRouter({
    // 此写法不可取，h2标签会消失
    // routes: [
    //     {path: "/user", component: user},
    //     {path: "/user/comm1", component: com01},
    //     {path: "/user/comm2", component: com02}
    // ]
    routes: [
        {
            path: "/user",
            component: user,
            children: [
                // 注意这里不能写/ 写了会跳回根目录
                {path: "comm1", component: com01},
                {path: "comm2", component: com02}
            ]
        }
    ]
})

var vm = new Vue({
    el: '#app',
    router,
});
</script> 
```

## Vue-cli   Command-Line Interface命令行界面

vue-cli作为一款mvvm框架语言(vue)的脚手架，集成了webpack环境及主要依赖，对于项目开发环境的搭建、打包、维护管理等都非常方便快捷。
代码目录结构、项目结构、部署、热加载、代码单元测试

安装vue-cli2
cnpm i vue-cli -g

最新版vue-cli3
npm install -g @vue/cli
桥接工具，可使用init功能
npm install -g @vue/cli-init

使用
`vue` 查看脚手架工具的功能
`vue init`  根据模板 创建一个项目
`vue list`  查看当前可用的官方模板

创建项目 vue-cli2
vue init webpack pro01`   webpack 表示模板名字   pro01 表示项目的名字，不可以有大写字母
VueBuild   Runtime+ Compile  编译加部署，使用template，在客户端即new vue中使用template
           Runtime-only      只部署，使用.vue文件夹开发
Install vue-router   yes
ESlint : no  规范代码风格
nightWatch  no
使用npm构建  no
cd到项目目录下   `cd pro01
cnpm install` 安装项目所有依赖
npm run dev`   编译项目  通过  指定域名去访问创建好的项目   默认就是 localhost:8080

创建项目 vue/cli3
vue create 项目名

## Vue-cli 项目目录

build：webpack相关配置文件（webpack，用于配置开发生产环境，打包脚本）
    build.js： 生产环境的配置
config：vue的基本配置文件
    index.js 配置的端口，打包输出的东西
    dev.env.js： 开发环境的配置
    prod.env.js： 生产环境的配置
node_module：项目的相关依赖
src：项目的核心文件
    assets：静态资源，css文件，less、sass文件（是为css添加逻辑处理的东西）
    component：公共组件
    router：路由，配置项目的路由，安装路由模块，导出路由
    App.vue：项目的根组件！
    main.js：入口文件
static：静态资源，图片，不会被压缩，assets中会被压缩
.babelrc：babel的编译参数，将ES6的语法降为ES5，使更多浏览器支持
.editorconfig：代码格式
.gitignore：配置上传git时需要忽略的配置
.postcssrc.js：转换css的工具
index.html：主页
package.json：项目的相关信息，脚本
README.md：项目描述说明

## vue框架执行流程

在执行npm run dev时，webpack会执行入口文件main.js
main.js是一个Vue实例
    import Vue from 'vue'           // 引入Vue框架
    import App from './App'         // 导入App.vue文件，webpack配置了省略有的后缀名
    import router from './router'   // 引进router文件，index.js

    Vue.config.productionTip = false
    
    /* eslint-disable no-new */
    new Vue({
      el: '#app',                   // el挂载要管理的元素  // 把app实例传给Vue实例，Vue实例负责管理app实例  // vm层管理v层
      router,                       // router挂载路由，router对象指向router文件，其中有index.js文件，该文件暴露了一个路由对象export default new Router
                                        // 路由对象包含了路由规则列表routes，routes又包含path、component键值对字典
                                        // <router-link to='/'>首页</router-link>需要在组件中定义路由入口，也要<router-view/>在组件中定义路由出口
      components: { App },          // 挂载组件App，这里是语法糖，App: App，App对象指向App.vue文件
      template: '<App/>'            // 表示使用App组件的模板
    })


组件树，多个模板被一个vue实例所引用

====================================================================================

ES6中模块的导入和导出
export let num = 1;
export function a() {
    console.log("hello")
}
export default {
    num: 123,
    a() {
        console.log("hi")
    }
}

index.js文件中
import {num, a} from '@/demo01'
console.log(num)
a()
import demo from '@/demo01'
console.log(demo)


====================================================================================

// 解析post请求需要的中间件

// 导入中间件，body-parser，用于post请求解析，是内置中间件不需要下载了
const bodyParser = require("body-parser")
// 原生不能解析body，bodyParser要用中间件解析
app.use(bodyParser.json())
app.use(bodyParser.urlencoded({extended: true}))

====================================================================================

后端开发（node）
Express 框架
    基于Node.js平台，快速、开放、极简的Web开发框架

// app.js入口文件中
const express = require("express")
const app = express()

// 加载路由对象router
app.use("/api/user", user)
// express的get方法和use方法的区别
// get参数可以传递路由对象
// use参数为回调函数

// 路由对象中
// express的路由中间件
const router = express.Router()
/*
    @url /test
    @desc 测试接口
*/
router.get("/test", (req, res) => {
    // res.send() 是发送普通字符串
    // res.json() 是发送json字符串
    res.json({
        msg: "success"
    }) 
})


====================================================================================

npm run server
    "server": "nodemon app.js"

npm run dev
    "dev": "webpack-dev-server --inline --progress --config build/webpack.dev.conf.js",

npm run build：用于打包项目
    移除之前的打包文件，再去执行webpack
    "build": "node build/build.js"



## bcrypt

​    是一个跨平台的文件加密工具。由它加密的文件可在所有支持的操作系统和处理器上进行转移。它的口令必须是8至56个字符，并将在内部被转化为448位的密钥。
除了对您的数据进行加密，默认情况下，bcrypt 在删除数据之前将使用随机数据三次覆盖原始输入文件，以阻挠可能会获得您的计算机数据的人恢复数据的尝试。如果您不想使用此功能，可设定禁用此功能。

bcrypt 的使用步骤
1. 安装模块   `cnpm i bcrypt --save`
2. 在需要加密的模块引用他   `const bcrypt = require("bcrypt")`
3. 设置加密强度  `let salt = bcrypt.genSaltSync(10)`
4. 使用加密算法，生成hash 加密后的密码  `bcrypt.hashSync(req.body.password, salt)`

// 加密算法是单向的，要比较两边都加密
var flag = bcrypt.compareSync(req.body.password, result[0].password)



====================================================================================

## 前端Vue

concurrently
    用于前后台项目连载
1. 安装工具 concurrently  `cnpm i concurrently --S`
2. 在`package.json`文件中编辑命令   
   1. 前台项目命令`"client": "npm run start --prefix client",`  // prefix client表示在client目录中执行
   2. 后台项目命令`"server": "nodemon app.js",`
   3. 连载前后台项目指令  `"begin": "concurrently \"npm run server\" \"npm run client\""`
3. 执行`npm run begin`指令

====================================================================================

## element-ui

​    是饿了么开发的前端ui框架

安装element-ui  
1. `cnpm i element-ui --S`
2. 在入口文件 main.js 
    // 在main入口中引入element-ui框架
    import ElementUI from 'element-ui'
    // 在引入组件库的同时还要引入样式
    import 'element-ui/lib/theme-chalk/index.css'


====================================================================================



====================================================================================

====================================================================================



====================================================================================

扩展
    ORM 框架

====================================================================================d

