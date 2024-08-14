# JavaScript

[MDN文档#JavaScript](https://developer.mozilla.org/zh-CN/)

所有JS对象都有个内置属性 object: prototype，指向对象原型，由prototype构成原型链，用于属性调用







事件：onSubmit=(e: event)=>{}  onSubmit  onClick

事件操作 e.tartget.value



=== char 好像不是比较地址，也可比较类型



JavaScript 是什么
	解析执行：轻量级解释型的编程语言
		轻量级   语法比较简单，不用编译，调试方便
		解释性
			解释型     源码不需要直接翻译成机器语言 ， 先翻译成中间代码  再交由解释器对中间代码进行解释执行     跨平台性较好
			编译型     将源码一次， 后面的执行就不需要重新编译，直接使用第一次编译的结果就好，执行效率比较高   跨平台型较差
	语言特点：动态，头等函数 (First-class Function)
		又称函数是 JavaScript 中的一等公民
  		函数参数、函数返回值、赋值给变量 具有这三个特点  就可以称作一等公民 （一等公民是方法）
  		在java 中什么是一等公民，  字符串   
	执行环境**：在宿主环境（host environment）下运行，浏览器是最常见的 JavaScript 宿主环境
	浏览器内核  由两部分组成    1. 渲染引擎    2. js引擎
js代码在载入完后，是立即执行的。所以将所有的script标签放到页面底部，也就是body闭合标签之前，这能确保在脚本执行前页面已经完成了DOM树渲染。

=============================================================================================

JavaScript 的组成
	ECMAScript - 语法规范 js的核心 
		简变量、数据类型（number,string,boolean,null,undefined（JavaScript预解析声明变量和函数，这时变量值是undefined未定义）,object）、类型转换(parseInt)、操作符
		流程控制语句：判断、循环语句
		数组、函数、作用域、预解析（预声明）
		对象、属性、方法、简单类型和复杂类型的区别
			简单数据类型 number、string、boolean、null、undefined
			复杂数据类型(引用数据类型) 对象就是一种复杂数据类型
		内置对象：Math、Date、Array，基本包装类型String、Number、Boolean类型String、Number、Boolean
	Web APIs 是Web浏览器提供给JavaScript的API
		BOM 浏览器对象模型 brower object modle
			域名变化
			onload页面加载事件，window顶级对象
			document.body.onload = function;
			定时器
			location前进、后退、刷新
			history历史记录
		DOM 文档对象模型
			获取页面元素，注册事件
			属性操作，样式操作
			节点属性，节点层级
			动态创建元素
			事件：注册事件的方式、事件的三个阶段、事件对象

=============================================================================================

JavaScript 执行过程
JavaScript 运行分为两个阶段：
	预解析
		全局预解析（所有变量和函数声明都会提前；同名的函数和变量函数的优先级高）
		函数内部预解析（所有的变量、函数和形参都会参与预解析）
			函数
			形参
			普通变量
	执行
		先预解析全局作用域，然后执行全局作用域中的代码，
		在执行全局代码的过程中遇到函数调用就会先进行函数预解析，然后再执行函数内代码


=============================================================================================

offsetHeight || offsetWidth = boder + padding + content  可以表示 里面部分 的长宽
	content：文字内容
	padding：内层块
	boder：隔层厚度
	marrgin：外层块	marrgin-top	marrgin-bottom

offsetleft、offsettop 是父元素外层到子元素外层的距离   可以表示到最外层的到浏览器边框的距离  ！常用这个

event.clientX 是目标点(在元素中)距离浏览器可视范围的X轴坐标
event.clientY 是目标点距离浏览器可视范围的Y轴坐标
浏览器的左上角
event.pageX 是目标点距离document最左上角的X轴坐标
event.pageY 是目标点距离document最左上角的Y轴坐标
元素的左上角是原点



| Chrome F12 JS 命令  | 释义                  |
| ------------------- | --------------------- |
| window.localStorage | localstorage 本地存储 |
|                     |                       |
|                     |                       |


