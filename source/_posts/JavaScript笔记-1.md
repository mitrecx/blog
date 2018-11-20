---
title: JavaScript笔记1-JS基础
date: 2018-11-15 21:54:46
tags: JavaScript
categories: JavaScript
---
# 0 前言

为什么要学习 JavaScript ？ [引用于w3schools](https://www.w3schools.com/Js/) :  

JavaScript is one of the 3 languages all web developers must learn:

   1. HTML to define **the content of web pages**  

   2. CSS to specify **the layout of web pages**  

   3. **<font color="red">JavaScript</font>** to program **the behavior of web pages**  

JavaScript 是一种轻量级的脚本语言，由浏览器执行。  

**与 JS 相关的技术**：  
1. HTML DOM 定义了访问和操作 HTML 文档的标准方法。  
2. **jQuery库** 极大地简化了 JavaScript 编程。  
3. **AJAX** = Asynchronous JavaScript and XML（**异步的 JavaScript 和 XML**）。    
4. **JSON** ：JavaScript 对象表示法（JavaScript Object Notation）。  

JS 代码可以写在 HTML 中，也可以写在 js 文件中：  
**1、** HTML 中的 JavaScript:   
HTML 中的 JS脚本 必须位于 &lt;script&gt; 与 &lt;/script&gt; 标签之间。  
JS脚本 可被放置在 HTML 页面的 &lt;body&gt;元素 和 &lt;head&gt;元素 中。  

**JavaScript** 是所有现代浏览器以及 HTML5 中的 **默认脚本语言**。  在 &lt;script&gt; 标签中使用 <code>type="text/javascript"</code> 不是必要的。  

**2、** 外部的 JavaScript:
外部 JavaScript 文件的文件扩展名是 .js。  
引用外部 JS 文件，需要在 &lt;script&gt; 标签的 "src" 属性中设置该 .js 文件：  
```html
<!DOCTYPE html>
<html>
  <head>
    <script src="myScript.js"></script>
  </head>
</html>
```

----

我对 JavaScript 的理解 ：  

1. 学习简单： UE + Chrome + W3School 即可学习开发    

2. 面向对象，回调函数    

3. 有点 Java 的影子 ，其 控制语句 和 运算符 与Java的语法几乎一致  

4. 在 JS 中，**变量没有类型，统一用 var 声明。但变量所引用的数据是有类型的**   

5. JS 函数是 **由事件驱动的** 或者当它被调用时执行的 **可重复使用的代码块**。  

![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-11/js%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B.png)  

**<font color=red>在 JS 中，一切皆对象。</font>**

----




# 1 JavaScript 内置对象
1. [String](#string)  
2. [Number](#number)  
3. [Array 数组](#array)  
4. [Math](#math)  
5. [Date 时间](#date)  
6. [RegExp 正则表达式](#regexp)
7. [Function](#function)  
8. Boolean (不做说明)  

注：  
**typeof(varName)** 可以查看变量的 数据类型。   
在浏览器中打开控制台( F12 ) ，可以方便的进行测试或学习。

<h2 id="string"> 1.1 String 对象</h2>

定义一个变量，引用的数据类型为 **String** 类型：  
```JavaScript
var str="hello world.";
```

String 的属性：  
**length属性**，str.length 获取字符串长度。  


String 的函数：  
1. **子串**， **str.substring**(startIndex, [endIndex]) 和java一样 "含头不含尾"   

2. **获取子串下标**， **str.indexOf**(findStr, [startIndex]),   **str.lastIndexOf**(findStr, [inverseStartIndex])  找到返回索引，找不到返回 -1   

3. **获取对应下标的字符**，**str.charAt(index)**  

4. **替换 replace**， str.replace(findStr, newStr) , [参考例子](#e1)   

5. **字符串切割**， **str.split**(findStr, [limit])  。limit 取负 和足够大一样。 但 limit 不够大，会忽略后面的段。

6. 大小写 toUpperCase(), toLowerCase()   

<h2 id="number"> 1.2 Number 对象  </h2>

引用的数据类型是 **Number** 类型：  
```javaScript
var num=5;
var num2=num/2; // num2 结果为 2.5
// javaScript 中 数字类型 包含了 "整型" 和 "浮点型"  
var n=0.5;
```

JavaScript 中，Number 对象 包括 整数和小数。  

<h2 id="array"> 1.3 Array 对象</h2>

创建 **数组对象**：    
```javaScript
var a1 = [1, 2, 3];
```
也可以 new 数组对象：  
```javaScript
var a2 = new Array();

//new 时 指定长度
var a3 = new Array(10);
```
**<font color=red>在 JS 中，数组的长度是可变的。</font> 因此，指定数组长度是没有意义的。**

为数组赋值：  
```javaScript
// 1. 直接 指定下标赋值
a1.[0]=100;

// 2. push 赋值，总是在数组的 最后 添加元素
a1.push(4);
```

数组元素 **排序**：  
```javaScript
// 按字符 顺序排： 100 < 2 < 3 < 4
a1.sort();
```
如果想要按 数字大小 排序，需要 **自定义排序方法** ：    
```javaScript
fuction compare(i,j){
  return i-j;
}

// 按照 数字大小排序： 2 < 3 < 4 < 100
a1.sort(compare);
```


**数组中的数组**：  
```JavaScript
var aa = [ [1, 2, 3], [6, 7, 8] ];

// 数组中 还是 数组， 各个数组 的类型可以不一致
var var aa=[[1,2,3],["hello","world"],["你好"]];
```

<h2 id="math"> 1.4 Math 对象</h2>  

无需创建， 直接使用 Math 对象的 属性和方法。  

比如：  

属性：   
Math.PI 圆周率 , Math.E 自然对数  等。  

方法：  
1. Math.round(x) 四舍五入   

2. **Math. abs(x)** 绝对值，**absolute value**    

3. Math.pow(x,n) x 的 n 次方， **power**   

4. Math.sqrt(x) 平方根， **square root**   

5. Math.random() 返回 0 ~ 1 之间的一个随机数    

6. Math.max(...value)  返回最大值  

7. 三角函数等等。

<h2 id="date"> 1.5 Date 对象</h2>  

创建 Date 对象：  
```javaScript
// d1 的值为 当前系统时间
var d1 = new Date();  

// 指定 d2 的值
var d2 = new Date("2017-3-20 11:23");
```

获取 毫秒 数：  
```javaScript
d1.getTime();
```

读写 时间分量：  
```javaScript
//年
d1.getFullYear();
// 月
d1.getMonth();
// 日
d1.getDate();
// 星期几
d1.getDay();
// 等等...

//写， set 对应的分量
```

Date 转 字符串：  
```javaScript
d1.toString();
// "Sat Nov 17 2018 19:07:53 GMT+0800 (China Standard Time)"

d1.toLocaleString();
// "11/17/2018, 7:07:53 PM"

d1.toLocaleDateString();
// "11/17/2018"
```

<h2 id="regexp"> 1.6 RegExp 正则表达式 对象</h2>

**<font color=orange>创建 RegExp 对象</font>**：  
```javaScript
var re = /pattern/[flags] ;

// 或者
var re = new RegExp("pattern", ["flags"]);
```
**flags** 标识有两个:  

1.  **g**, 全局模式  

2.  **i**, 忽略大小写  

RegExp 对象有一个 **test()** 方法，用于 测试字符串 是否匹配 正则表达式。如果匹配，**返回true**。 否则 **false**。  
```javaScript
re.test(str);
```
注：RegExp 还有一个 **exec()** 方法，这个方法不返回 boolean 类型，而是 **返回匹配到的字符串**。  


**<font color=green>举两个例子</font>**：  

匹配字符串，包含且仅包含 [3, 6] 位数字：    
```javaScript
var re = /^\d{3,6}$/;

re.test("123"); //true

re.test("12"); //false
re.test("12a"); //false
```

匹配字符串，是否包含 3 位 数字：  
```javaScript
var re2=/\d{3}/;

re2.test("mitre123"); //true
re2.test("mitre1234"); //true 包含3位数字

re2.test("mitre12"); //false
```

**<font color=orange>[String](#string) 和 RegExp</font>** :    
<i id="e1">举个例子</i>：  
```javaScript
var str="JavaScript 和 Java 语法有点类似, 但是 JavaScript 不是 Java";

var re=/Java/g;

str.match(re); //返回匹配到的字符串的 数组
// (4) ["Java", "Java", "Java", "Java"]

str.replace("Java","java"); //不用正则， 只改变第一个
// "javaScript 和 Java 语法有点类似, 但是 JavaScript 不是 Java"
str.replace(re,"java");
// "javaScript 和 java 语法有点类似, 但是 javaScript 不是 java"

str.search(re); //返回匹配到的第一个 下标， 如果没有匹配到返回 -1
// 0
```

登录案例：  
JS 验证用户名和密码是否符合要求。  
```HTML
<html>
	<head>
		<title>登录</title>
		<link rel="stylesheet" type="text/css" href="style.css">

		<script>
			//校验用户名
			function checkName () {

				//获取账号文本框
				var input=document.getElementById("name");
				//获取账号提示span
				var span=document.getElementById("nameMsg");

				//获取文本框的值
				var name = input.value;
				var re = /^\w{3,20}$/;
				//若不匹配，把span样式改一下
				if(!re.test(name)){
					//className 等价于元素的 class属性
					span.className="errorMsg";
					return false;
				}else{
					span.className=""; //匹配上需要移除错误样式
					return true;
				}
			}
		</script>
	</head>

	<body>
		<!-- onsubmit 是表单提交事件，点击提交时触发。提交表单前先调用 onsubmit 中的方法，只有方法返回 true 才提交表单-->
		<form action="mitrecx.cn" onsubmit="return checkName()&&checkPassword();">
			<h1>登录</h1>
			<p>
				账号：<input id="name" type="text" onblur="checkName();"/>
				<span id="nameMsg">3-20 位 字母、数字、下划线</span>
			</p>
			<p>
				密码：<input type="password" onblur="checkPassword();" />
				<span>6-20 位 字母、数字、下划线</span>
			</p>
			<div>
				<input type="submit" value="登录"/>
			</div>
		</form>

	</body>
</html>
```

style.css 文件：  
```CSS
form{
	width: 500px;
	margin: 150px auto;
	border: 1px solid #00ff00;
}
form h1{
	text-align:center;
	margin: 0;
	padding: 5px;
	border: 1px solid #ccc;
}
form p{
	margin: 0;
	padding: 20px;
	border: 1px solid #ccc;
}
form div{
	text-align:center;
	margin: 0;
	padding:10px;
	border: 1px solid #ccc;
}

.errorMsg{
	border: 1px solid red;
	color:red;
}
```
结果：  
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-11/2018-11-17_222205.png)  

<h2 id="function"> 1.7 Function 对象</h2>  

在 JS 中，函数就是 Function 对象。  
函数名就是 指向 Function 对象的引用。  

**JS 函数定义**：  
```JavaScript
function functionName([参数]){
  函数体;
  [return 返回值;]
}
```
**函数的返回值**：  

* 默认返回 undefined  

* 可以使用 return 返回具体的值  

**函数的参数**：  

1. JS 函数没有重载。调用时 只要函数名相同，无论传入多少个参数，调用的都是同一个函数。    

2. 没有收到 **实参** 的参数值都是 **undefined** 类型。  

3. 所有的参数都传递给 JS内部的 **<font color=red>arguments数组</font>** 对象。  
function 函数定义时，不论有没有参数，**<font color=red>arguments数组</font>** 都是存在的。  

arguments 数组：  
* arguments 是一种特殊的对象，在函数中，表示函数的 **参数数组** 。  

* 在函数中，可以使用 arguments 访问所有的参数：  
arguments.length : 函数 **实参** 个数  
arguments[i] : 第 i 个参数  

有了 arguments ， 在 JS 函数中声明与不声明 **形参** 都可以。这使得 JS 的函数定义变得很灵活。  
一般情况下，最好用到几个参数，就声明几个参数，这样符合编程习惯。  

例如，定义一个 sum() 函数，求所有参数的和：  
```JavaScript
function sum(){
    var sum=0;
    for(var i=0;i<arguments.length;++i){
        sum+=arguments[i];
    }
    return sum;
}
```
F12，在浏览器控制台测试结果：  
```
sum(1,2);
3
sum(1,2,3,4,5);
15
```

使用 Function 创建函数 (几乎没人这样写)：  
```JavaScript
var add=new Function("x","y","return(x+y)");
//效果等价于 var add(x,y){return x+y;}
```
结果：
```
add(1,2);
3
```
**alert(add)**，把用 new Function 创建的函数打印出来，结果：  
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-11/2018-11-18_004942.png)   
这个函数名是 anonymous (匿名的)。  

**匿名函数**    
创建匿名函数：  
```JavaScript
var func = function(arg1, ... , argn){
  func_body;
  return returnValue;
}
```
匿名函数应用：  
```JavaScript
var a=[ 8, 7, 1, 2, 10, 33, 5];

a.sort();
//结果：(7) [1, 10, 2, 33, 5, 7, 8]

//使用匿名函数
a.sort( function(a,b){return a-b;} );
//结果：(7) [1, 2, 5, 7, 8, 10, 33]
```

**全局函数**  
所谓[全局函数](http://www.w3school.com.cn/jsref/jsref_obj_global.asp)，就是不用创建对象，拿来就能用的函数。  

JS常用的全局函数：  
```JavaScript
parseInt(str);
parseFloat(str);

//是数字类型 返回 false
isNaN(variable);

//把 str 当做 JS表达式 或 JS代码 执行
eval(str);

// 把字符串编码为 URI
encodeURI(URIstring)
// 返回 URIstring 的副本，其中的某些字符将被十六进制的转义序列进行替换

decodeURI()	//解码某个编码的 URI。
```

# 2 外部对象

**外部对象就是 浏览器对象模型 ( Browser Object Model, BOM )。**  

![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-11/2018-11-18_100841.png)  


BOM 是 浏览器提供的一套 API。它使 JS 有了操作浏览器窗口的能力。  

**文档对象模型 ( Document Object Model, DOM )** 定义了访问和操作文档的方法，它属于 BOM，但是 DOM 的内容占据 BOM 的半壁江山。DOM 就是上图的 **document**。  

## 2.1 浏览器对象模型 BOM

window 对象的方法和属性 都可以省略 **<font color=red>window.</font>** 前缀。  
比如，**window.alert(str)** 可以 写成 **alert(str)** 。  

window对象的方法：  

1、 对话框：  
```JavaScript
//提示对话框， 无返回
alert("hello world");

//确认对话框， 确认返回true，取消返回false
confirm("你确认吗? ");
```

2、 定时器：  
定时器通常用于定时刷新页面。  

一次性定时器： setTimeout(functionName,time)  
周期性定时器： setInterval(functionName, time)

周期性定时器：  
```JavaScript
// 每隔 time 时间 触发执行 functionName函数/代码
setInterval(functionName, time);
// time 的单位是毫秒
// 返回 已经启动的定时器对象

// 停止已经启动的定时器对象
clearInterval(tID);
// tID 是启动的定时器对象
```

举个栗子，每隔 3 秒 弹出提示对话框：  
```JavaScript
var tid=setInterval("alert('hello');",3000);

//或者
setInterval(fun,3000);
function fun(){ alert("hello"); }

//或者
setInterval(function (){
  alert("hello");
}, 3000);
```

## 2.2 history 对象

history 对象 **记录了一个窗口访问过的 URL**   

history 的 属性 和 方法：   
1. length属性，记录中 URL 的数量  
2. back() ， 等同于点击 后退 按钮  
3. forward() ，等同于点击 前进 按钮
3. go(n) ，n为正前进 n 次， n 为负后退 n 次  

## 2.3 location 对象

**location 对象 包含了当前 URL 的相关信息**  

location 通常用于 获取和改变当前浏览的 URL  

location 的 属性 和 方法：  
1. href 属性： 若给 href 属性赋值，则会改变当前浏览的 URL。 若不赋值则 返回当前窗口正在浏览的地址。    

2. reload() 方法： 重新加载当前地址。  

## 2.4 screen 和 navigator

**screen 对象包含有关客户端显示屏幕的信息。**  

Screen 对象属性：  
1. availHeight	返回显示屏幕的高度 (除 Windows 任务栏之外)。  
2. availWidth	返回显示屏幕的宽度 (除 Windows 任务栏之外)。  

**navigator 对象包含有关浏览器的信息。**  

navigator 对象 属性：  
1. userAgent	返回由客户机发送服务器的 user-agent 头部的值。

## 2.5 DOM

DOM 提供了如下操作：  
1. 查找节点  
2. 读取节点信息
3. 修改节点信息  
4. 创建新节点  
5. 删除节点  

节点信息：  
nodeName 节点名称，   
nodeType 节点类型。  

节点的内容：  
innerHTML，    
innerText。  

节点的属性：  
getAttribute()  
setAttribute()  
removeAttribute()  

节点的样式：  
className， 类选择器封装的类型  
style  

```JavaScript
//<p id="t1"> hello world.</p>
vat t = document.getElementById("t1");

//改变字体颜色  和 大小
t.style.color = "red";
t.style.fontSize = "30px";

// <a href="xxx.jsp" title="this is a link"  id="a1" > xxx </a>
var a1 = document.getElementById("a1");
//获取 a 元素的 href 属性 值
var href = a1.getAttribute("href");

}
```

# 3 自定义对象

自定义对象 是一种有属性和方法封装而成的 **数据类型**。   

调用对象属性： 对象名.属性名  
调用对象方法： 对象名.方法名  

创建 自定义对象 有三种方法：  

1、 直接创建  
```JavaScript
// 直接 new 一个 Object
var teacher = new Object();
// 自定义属性
teacher.name = "陶行知";
teacher.age = "50";
// 自定义方法
teacher.work = function(){
  alert("我教 JavaScript");
}

// 调用对象
alert("姓名: "+ teacher.name +" 年龄： "+teacher.age);
teacher.work();
```

2、 使用构造器创建  
```JavaScript
// 写一个构造器方法
function Student(name,age){
  this.name=name;
  this.age=age;
  this.work=function(){
    alert("我学 Java");
  }
}

// 创建对象
var s=new Student("小艾",20);
```

3、 **<font color=red>使用 JSON 创建</font>**  
```JavaScript
// 创建一个 programmer对象
var programmer={
  "name":"mitre",
  "age":24;
  "work":function(){
    alert("我写 Java  ");
  }
}

// 可以直接调用 programmer 对象
alert(programmer.name+" "+programmer.age);
programmer.work();
```
