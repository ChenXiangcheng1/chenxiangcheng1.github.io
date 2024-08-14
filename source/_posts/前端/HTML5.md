# HTML5

[MDN文档](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element)



(以下内容都不重要，有需要看文档就好)

---

H5 = HTML5 + CSS3 + ES6，指基于HTML5的移动端页面



HTML5

localStorage 取代了 Cookie

```html
更多的语义标签：
<nav />  里面放链接，语义作用，导航
<header />
<footer />
<audio />
<video />
<canvas /> 绘制图形和动画
SVG支持
<article />
<section />  用于里面放标题的，语义作用，没有实例效果
```



## DOM DOCTYPE

DOCTYPE是document type的简写。主要用来说明你用的XHTML或者HTML是什么版本。浏览器根据你DOCTYPE定义的DTD(文档类型定义)来解释页面代码

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
```



## head

### meta 

html标签

meta 定义关于 HTML 文档的元信息，一般在head标签中

```html
<meta http-equiv=”Content-Type” content=”text/html; charset=utf-8″/>
```



### link

定义文档与外部资源的关系，一般在head标签中

```html
<link rel="stylesheet" type="text/css" href="styles.css">
```



### title

定义标题

```html
<title></title>
```



## body

### div

```html
<div></div> 定义文档分区，class 用于元素组（类似的元素，或者可以理解为某一类元素），而 id 用于标识单独的唯一的元素
```



### 列表

```html
<ul>  unorder list 无序列表
<ol>  order list 有序列表
```

```html
<dl></dl>  用于创建一个普通的列表
<dt></dt>  用于创建列表中的上层项目
<dd></dd>  用于创建列表中的最下层项目
一般用于图文并排的情况
```



### 文本

```html
<h1></h1> 表示字体大小为一号字的标签

<b> 文本加粗

<strong></strong> 字体加粗

<font></font> 可以设置标签中字体的属性

<br/> 表示换行

<hr/> 表示创建一条水平线

<center> 报错，可能是淘汰了
    
<span></span> 定义文档中的节,属性border：solid设置实线

<p> 会自动在其前后创建一些空白。浏览器会自动添加这些空间，您也可以在样式表中规定。
```



### img

```html
<img/>
表示插入一张图片
src：路径
alt：图片找不到的提示信息字符串
```



### form

```html
<form action="dologin.jsp" name="loginForm" method="提交方式***"></form> 
定义供用户输入的 HTML 表单，定义表单，前端数据通过表单传到后端，属性style="cursor:pointer;"表示鼠标在上面时会出现小手的形状
表单标签，定义在  动作／名称等顺序无所谓
action=url地址	表单提交给哪个动作去处理
name	表单名
method	有两种提交方式
get：用户提交的数据通过URL提交，在URL中可以看到，最多不超过2KB。安全性较低，但效率比post方式高。适合提交数据量小，且安全要求低的数据：比如：搜索、查询等功能。
post：用户提交的信息封装到数据包HTML HEADER内（HTML响应头部）。适合提交数据量大，安全要求高的数据，如：注册、修改、上传等功能。

onSubmit事件
```



### table

```html
<table></table> 定义表格bycolor属性

<tr></tr>  定义表格中的行，属性valign设置垂直对齐方式 align设置水平对齐方式比如center水平居中

<td></td> 定义表格中的单元，属性colspan（数字，跨的列数）有跨列的效果，align="center"表示居中，bgcolor背景色
```



### input输入标签

```html
<input /> 表示输入标签，有三个必须属性type（text、password、submit、checkbox、radio）
属性：
name 是标签名字会被传到客户端URL栏
value 值在标签内会被传到客户端的值上
style="cursor:pointer;"光标在按钮上面会变成小手
type="reset" 取消
type="submit" 提交
type="radio" 单选框
type="checkbox" 复选框

<input type="radio" name="gender" value="Male" title="xxx">woman</input>
圆点表示单选框，，单选框复选框不加value时选中就是value就是on
input使用日历控件--先引入javascript文件，在onclick="new Calendar().show(this);" readonly="readonly"
title属性鼠标悬停提示

type="checkbox"表示复选框
checked="checked"表示预先选定复选框或单选按钮，
使用request.getParameterValues("inputName")来获取同名复选框的字符串数组信息
```

```html
<textarea rows="3" cols="20"></textarea> 定义多行的文本输入控件，属性rows行数，cols可见宽度
```

```html
<lable> 为 input 元素定义标注
```



### select下拉

```html
<Select>multiple设置多选（ctrl可以多选）  size显示的行数
    <Option selected（设置默认选中）value/>或者<Option>字</Option>
</Select>  实现下拉选择表
```



### 超链接

```html
<base> 标签为页面上的所有链接规定默认地址或默认目标

<a> 定义超链接
有属性href（URL）规定链接指向的页面的URL，例：href="login.jsp?username=lisi" 这里？后可以带参数
属性target规定在何处打开链接文档，相当于请求重定向，属于get()请求	_blank在一个新的窗口打开被连接文档	_self当前页面	_parent在父框架集中		_top在整个窗口page中
```



### frameset

```html
<frameset rows="20%,*">						框架行2.8开
    <frame src="top.jsp">					上框架2
    <frameset cols="20%,*">					下框架有分列2.8开8
        <frame src="main_left.jsp">				下左框架2下右框架8
        <frame src="main_right.jsp" name="main_right">			
    </frameset>
</frameset>
```



### 文字轮播

```html
<marquee>文字</marquee>  文字会轮播从右到左
```



## 标签属性

属性class是为了对对象分类，name是为了分对象

标签属性可以设置成#，表示属性暂定

空格：浏览器在解析HTML时，会把连续的空格解析成只有一个空格，所以我们会使用&nbsp; 等这样的占位符，&nbsp;的宽度是浏览器字符的宽度

