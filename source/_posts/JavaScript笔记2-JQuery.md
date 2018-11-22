---
title: JavaScript笔记2-JQuery
date: 2018-11-19 23:29:52
tags: ["JavaScript", "JQuery"]
categories: JavaScript
---

在[上一节](https://mitrecx.github.io/2018/11/15/JavaScript%E7%AC%94%E8%AE%B0-1/)说到，web developers 必修的三门语言：  
1. HTML  
2. CSS  
3. JavaScript  

学习 jQuery 必须对上面 3 门语言有一定的了解。  

为什么要学习 jQuery：  

1. jQuery 是一个优秀的、**轻量级的 JS 库**。极大的简化了 JS 编程。   

2. jQuery 封装了 JS、 CSS、 DOM， 提供了一致的简洁的 API。  

3. 使用户更方便的处理 HTML、 Events， 实现动画效果。

4. 方便地为网站提供 AJAX 交互。  

**jQuery 的使用步骤：**  

1. 引入 [jQuery 库](https://code.jquery.com/jquery-3.3.1.min.js)。   

2. 使用 **选择器** 定位要操作的节点。  

3. 调用 jQuery 方法 进行操作。  

**<font color=red>jQuery 语法： $(selector).action()</font>**   
美元符号定义 jQuery  
选择器（selector）"查询" HTML 元素  
action() 执行对元素的操作  

# 1 jQuery 对象  

**DOM对象**： 用传统的方法(javascript)获得的对象。    
**jQuery对象**： 通过 jQuery选择器 获得的对象 是 jQuery对象，如<code>$(".class1")</code>, <code>$("#id1")</code>。  

```JavaScript
 //DOM对象
var domObj = document.getElementById("id");

 //jQuery对象;
var $obj = $("#id");
```

**jQuery对象** 是jQuery独有的。如果一个对象是jQuery对象，那么就可以使用jQuery里的方法，例:  
```JavaScript
//获取id为foo的元素内的html代码，html()是jQuery特有的方法
$("#foo").html();
```
以上代码等价于：  
```JavaScript
// innerHTML 是 DOM对象 的方法
document.getElementById("foo").innerHTML;
```

**<font color=red>注：在jQuery对象中无法使用DOM对象的任何方法。</font>**    
当然，DOM对象也无法使用jQuery对象的任何方法。    

**重点：<font color=green> jQuery对象 的本质是 DOM对象 数组</font>**    

**jQuery对象和DOM对象的互相转换：**  

jquery提供了两种方法将一个jquery对象转换成一个dom对象，即 **[index]** 和 **get(index)**。    

```JavaScript
// 通过id选择器 获取 jquery对象
var $cr=$("#cr");
// jQuery 对象 转化为 DOM对象
var domObj = $cr[0]; //也可写成 var domObj=$cr.get(0);
```

dom对象转换成jquery对象： 只需要用$()把dom对象包装起来，就可以获得一个jquery对象。    
方法为 $(dom对象)。  

```JavaScript
//dom对象
var domObj=document.getElementById("cr");
// DOM对象 转换成 jquery对象
var $cr = $(domObj);
```

平时用到的jquery对象都是通过$()函数制造出来的，$()函数就是一个jQuery对象的制造工厂。   

建议:   
**如果获取的对象是 jQuery对象，那么在变量前面加上$** , 这样方便容易识别出哪些是jQuery对象。  
例如:   
<code> var $variable = jQuery对象;</code>  
如果获取的是dom对象，则定义如下:  
<code> var variable = dom对象;</code>  

# 2. jQuery 选择器

jQuery选择器 和 CSS选择器 类似，但是比 CSS选择器 更丰富。  

CSS选择器：选择元素，施加样式。  
jQuery选择器：选择元素，施加方法。  

1 基本选择器
```JavaScript
$("#id")            //id选择器, 根据属性id名 定位元素
$("div")            //元素选择器， 根据 标签名 定位元素
$(".classname")     //类选择器 ，根据 属性class名 定位元素
$(".classname,.classname1,#id1")     //组合选择器，定位一组选择器所对应的 所有元素
```

2 层次选择器
```JavaScript
$("select1 select2")  //在 select1元素下， 选中所有满足 select2 的 子孙元素
$("select1 > select2")//在 select1元素下， 选中所有满足 select2 的 子元素  (只是子元素，不是子孙元素)

$("#id .classname ")    //后代元素选择器
$("#id > .classname ")    //子元素选择器
$("#id + .classname ")    //紧邻下一个元素选择器（下一个兄弟）
$("#id ~ .classname ")    //兄弟元素选择器（所有兄弟）
```

3 过滤选择器(重点)
3.1 基本过滤选择器 ( 用于表格和列表，可以在一组同样的元素中，选第一个，最后一个，奇数位等等 )
```JavaScript
$("li:first")    //第一个li
$("li:last")     //最后一个li
$("li:even")     //挑选下标为偶数的li
$("li:odd")      //挑选下标为奇数的li

$("li:eq(4)")    //下标等于 4 的li(第五个 li 元素)
$("li:gt(2)")    //下标大于 2 的li
$("li:lt(2)")    //下标小于 2 的li
$("li:not(#runoob)") //挑选除 id="runoob" 以外的所有li
```
3.2 内容过滤选择器
```JavaScript
$("div:contains('Runob')")    // 包含 Runob文本的元素
$("td:empty")                 //不包含子元素或者文本的空元素

$("div:has(selector)")        //含有选择器所匹配的元素
$("td:parent")                //含有子元素或者文本的元素
```
3.3 可见性过滤选择器
```JavaScript
$("li:hidden")       //匹配所有不可见元素，或type为hidden的元素
$("li:visible")      //匹配所有可见元素
```
3.4 **属性过滤选择器** (根据属性 定位元素)
```JavaScript
$("div[id]")        //所有含有 id 属性 的 div 元素
$("div[id='123']")        // id属性值为123的div 元素
$("div[id!='123']")        // id属性值不等于123的div 元素

$("div[id^='qq']")        // id属性值以qq开头的div 元素
$("div[id$='zz']")        // id属性值以zz结尾的div 元素
$("div[id*='bb']")        // id属性值包含bb的div 元素
$("input[id][name$='man']") //多属性选过滤，同时满足两个属性的条件的元素
```
3.5 状态过滤选择器
```JavaScript
$("input:enabled")    // 匹配可用的 input
$("input:disabled")   // 匹配不可用的 input
$("input:checked")    // 匹配选中的 input
$("option:selected")  // 匹配选中的 option
```
4 表单选择器
```JavaScript
$(":input")      //匹配所有 input, textarea, select 和 button 元素
$(":text")       //所有的单行文本框，$(":text") 等价于$("[type=text]")，推荐使用$("input:text")效率更高，下同
$(":password")   //所有密码框
$(":radio")      //所有单选按钮
$(":checkbox")   //所有复选框
$(":submit")     //所有提交按钮
$(":reset")      //所有重置按钮
$(":button")     //所有button按钮
$(":file")       //所有文件域
```

# 3 jQuery 属性和方法
属性：  
<code>$jqObj.length</code>	包含 jQuery 对象中元素的数目  

jQuery对象的 方法 可以分为 4类：  
1. 事件方法  
2. 效果方法
3. HTML/CSS 方法  
4. 遍历节点

## 3.1 事件方法
```JavaScript
blur()	// 添加/触发失去焦点事件
change()	// 添加/触发 change 事件
click()	// 添加/触发 click 事件
dblclick()	// 添加/触发 double click 事件

mousedown()	// 添加/触发 mousedown 事件
mouseup()	// 添加/触发 mouseup 事件

mouseleave()	// 添加/触发 mouseleave 事件
mousemove()	// 添加/触发 mousemove 事件
mouseout()	// 添加/触发 mouseout 事件
mouseover()	// 添加/触发 mouseover 事件
```

## 3.2 效果方法
用于创建动画效果的 jQuery 方法。  
略。  

## 3.3 HTML/CSS 方法

读写节点的 HTML 内容：  
```JavaScript
// 读
$jqObj.html();

// 写
$jqObj.html("<span> hello </span>");
```


读写节点的文本内容：  
```JavaScript
// 读
$jqOjb.text();

// 写
$jqObj.text("hello");
```

读写 **表单或按钮等 <font color=red>value</font> 属性** 值：  
```JavaScript
// 读
$jqObj.val();

// 写
$jqObj.val("hello");
```

读写节点的 **属性值** ：  
```JavaScript
// 读
$jqObj.attr("属性名");

// 写
$jqObj.attr("属性名","新的属性值");
```

增删节点：  

**创建 DOM 节点**：  <code> $("节点内容") </code>  
创建的节点 **会存在 内存里**。 但是页面上看不到，只有把 节点 **追加到已经存在的某个节点下**， 页面上才能看到。  

```JavaScript
// 创建一个 DOM 节点
var pNode = $("<i> hello </i>");

// 插入 DOM 节点
// 在 p元素之前插入 DOM节点 (pNode 作为 p 的上一个兄弟节点)
$("p").before(pNode);
//结果：
//<i> hello </i>
//<p>xxx</p>

// pNode 作为 p 的 下一个兄弟节点
$("p").after(pNode);

// pNode 做为 p 的第一个子节点
$("p").prepend(pNode);
//结果：
//<p><i> hello </i>xxx</p>

// pNode 作为 p 的最后一个子节点
$("p").append(pNode);

// 删除节点
$("p").remove();
// 清空节点： 节点还在，内容清空
$("p").empty();
```

## 3.4 遍历节点
略。

# 4 示例
把 input 元素（type 属性值 为 "password"）的 type 属性值改为 "text"。  

**jQuery 实现**：  
```JavaScript
<html>
<head>
	<script src="jquery-3.3.1.min.js"></script>
	<script type="text/javascript">
		function doclick(){
			$("#password1").attr("type","text");

			alert($("#password1").val());
		}
	</script>
</head>

<body>
	<form>
		<input type="password" id="password1" value="123456" />
		<input type="button" id="button1" onclick="doclick();" value="Show default value" />
	</form>
</body>
</html>
```
也可以写成如下形式：  
```JavaScript
<html>
<head>
	<script src="jquery-3.3.1.min.js"></script>
	<script type="text/javascript">
		 $(document).ready(function(){
		 	$("#button1").click(function(){
		 		$("#password1").attr("type","text");

				alert($("#password1").val());
			})});
	</script>
</head>

<body>
	<form>
	<input type="password" id="password1" value="123456" />
	<input type="button" id="button1"  value="Show default value" />
	</form>
</body>
</html>
```
**ready() 方法：** 在文档加载后激活函数。  
当 DOM（文档对象模型） 已经加载，会发生 ready 事件。  
ready()方法 有三种语法：  
```JavaScript
//语法 1
$(document).ready(function)
//语法 2
$().ready(function)
//语法 3
$(function)
```
注： 语法 3 和 DOM对象转jQuery对象的语法一样 ！！！  


**JS 实现**：
```JavaScript
<html>
<head>
	<script type="text/javascript">
		function doClick () {
			document.getElementById("password1").type="text";

			alert(document.getElementById("password1").value);
		}
	</script>
</head>

<body>
	<form>
		<input type="password" id="password1" value="123456" />
		<input type="button" id="button1" onclick="doClick();" value="Show default value" />
	</form>
</body>
</html>
```


# 参考

1. [DOM对象和JQuery对象的区别](https://www.cnblogs.com/yellowapplemylove/archive/2011/04/19/2021583.html)   

2. [菜鸟教程: jQuery 选择器](https://www.runoob.com/jquery/jquery-ref-selectors.html)

# 附

![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-11/2018-11-20_144025.png)  

上图是在 Chrome 浏览器 的 console 控制台测试的结果。  

**通过 jQuery 的 id 选择器选择的结果 可以调用 DOM对象的value属性**， **<font color=red>但不可以调用 jQuery对象的 val() 方法</font>** 。  

jQuery 选择器返回的结果 为什么 不是 jQuery 对象？
