---
layout: post
title: "表单提交方式及其优缺点探究"
date: 2017-10-09
---

本文的组织方式

1. form表单相关基础（method、action、enctype）
2. 传统的form表单提交方法，以及各种表单元素数据的传递形式
	1. GET提交（表单序列化、GET方式的缺点）
	2. POST提交（传统表单提交的缺点）
		* 无文件提交
		* 带文件提交
3. Ajax方式提交form表单
	1. XMLHttpRequest 1级 GET方式提交
	2. XMLHttpRequest 1级 POST方式提交
	3. XMLHttpRequest 2级（FormData类型）

### form表单三个重要属性解释

1. action: 指定表单的提交目标地址
2. method：

	|取值|描述|
	| :----   | :----- |
	|"get"   | 以GET方式提交表单数据，表单项以键值对形式后缀到请求链接。|
	|"post"  | 以POST方式提交表单数据，表单数据被嵌入HTTP请求发送到目的地址。|

3. enctype:

	|取值|描述|
	| :----   | :----- |
	|application/x-www-form-urlencoded|默认。提交数据之前会对所有字符进行编码，空格被转成“+”，特殊字符被转换成十六进制ASCII形式。|
	|multipart/form-data|用于提交文件，数据不会被转码|
	|text/plain|只有空格会被转码成“+”，其他特殊字符不会被转码|

### 表单提交传统方式探究

1. **GET方式提交表单**

	这里主要研究一下，各种输入元素的值，是以什么样的形式传递的，比如输入框，单选框，复选框，多选框等输入值是如何传递的。

	下面是一个包含多种输入元素的表单，注意这个表单没有指定提交方式和编码方式。

	```
	<form action="action.php">
		<div>
			name: <input type="text" name="name" />
		</div>
		<div>
			sex:
			<input type="radio" name="sex" value="female"/>Female
			<input type="radio" name="sex" value="male" />male
		</div>
		<div>
			fruits:
			<input type="checkbox" name="fruit" value="apple">Apple
			<input type="checkbox" name="fruit" value="orange">Orange
			<input type="checkbox" name="fruit" value="peach">Peach
		</div>
		<div>
			hobby:
			<select name="hobby" multiple>
				<option value="basketball">Basketball</option>
				<option value="football">Football</option>
				<option>badminton</option>
			</select>
		</div>
		<div>
			file:
			<input type="file" name="file" />
		</div>
		<input type="submit" value="Submit" />
	</form>
	```

	渲染的表单如下图所示，图中的数据也是我们将要提交的数据：

	![rendered form](/imgs/2017-10-09-form.png)

	点击提交按钮后，在Chrome浏览器的Network栏查看提交请求如下：

	```
	action.php?name=lc%2B%2B&sex=male&fruit=apple&fruit=peach&hobby=basketball&hobby=badminton&file=examplefile.txt
	```

	来看看这个表单提交给出了哪些信息：
	1. 数据是以键值对的形式后缀到链接（以？为分隔，前面是数据传递的目的地，后面是参数，以&符号衔接），发送到处理请求的脚本处。说明HTML提交数据默认使用的是GET方式。
	2. name的值（lc++）经过了编码，说明encType的默认值是application/x-www-form-urlencoded，因为只有这种形式才会对字符编码。
	3. 存在两个fruit键值对，说明多个相同name属性的复选框，是以多个名称相同的键值对发送数据的。
	4. 存在两个hobby键值对，说明多选框也是以多个名称相同的键值对发送数据的。同时，当option有value属性时，键值对的值为value值，当option没有value属性时，键值对的值为该option的文本内容（如hobby=badminton）
	5. 有一点需要注意：表单项必须有name属性，否则即使填写了表单，数据也不会发送到后台。

	但是GET请求有以下几个缺点：
	1. 请求的链接长度受到限制，所以传递的数据量有限
	2. GET方式发送的数据对任何人可见，若含有敏感信息，则有安全隐患
	3. 键值对的形式，无法传递文件内容（上例仅仅传递了文件的名称）

	使用POST方法，可以解决以上问题。

2. **POST方法提交表单**

	使用post方法，因为表单默认提交方式是GET，所以必须在form元素中明确指定method属性的值为“post”

	在没有文件传递的情况下，form元素的设置形式如下

		<form action="action.php" enctype="application/x-www-form-urlencoded" method="POST">

	请求的数据同样以键值对的形式传递，不过，这些数据被嵌入到HTTP请求的主体中，对他人是不可见的。相同name的元素，同样存在多个同名键值对。

	![post without file](/imgs/2017-10-09-post-no-file.png)

	但是如果需要传递文件，则enctype的值必须设置成"multipart/form-data"，而且method也必须是POST。form元素的设置形式如下：

		<form action="action.php" enctype="multipart/form-data" method="POST">
	
	这种方法是将表单内容以二进制的方式传递到了目的地，如下是POST请求payload的部分截图：

	![post with file](/imgs/2017-10-09-post-with-file.png)

	以上便是浏览器默认的表单提交方式，但是这种方式有很多不足，其中最大的问题就是，这种表单的处理同步的，也就是说，当表单提交后，浏览器必须等待处理结果，在此期间，浏览器无法处理其他事务，从而导致网页卡顿，甚至假死的情况。而Ajax则解决了这个问题，那么，自然我们就希望使用Ajax的方式发送数据。

### Ajax方式提交表单

1. **XMLHttpRequest 1级 GET方式提交**

	使用get方式提交，有一个问题需要我们手动处理，那就是表单的序列化，即，如何将表单项的值以键值对的形式添加到请求链接的后面。

	下面是一个来自《JS高级程序设计》第14章14.4小节的表单序列化函数
	```
	function serialize(form){
		var parts = [], field = null, i, len, j, optLen, option, optValue;
		for (i=0, len=form.elements.length; i < len; i++){
			field = form.elements[i];
			switch(field.type){
				case "select-one":
				case "select-multiple":
					if (field.name.length){
						for (j=0, optLen = field.options.length; j < optLen; j++){
							option = field.options[j];
							if (option.selected){
								optValue = "";
								if (option.hasAttribute){
									optValue = (option.hasAttribute("value") ? option.value : option.text);
								} else {
									optValue = (option.attributes["value"].specified ? option.value : option.text);
								}
								parts.push(encodeURIComponent(field.name) + "=" + encodeURIComponent(optValue));
							}
						}
					}
					break;
				case undefined: //字段集
				case "file": //文件输入
				case "submit": //提交按钮
				case "reset": //重置按钮
				case "button": //自定义按钮
					break;
				case "radio": //单选按钮
				case "checkbox": //复选框
					if (!field.checked){
							break;
					}
				/* 执行默认操作 */
				default:
					//不包含没有名字的表单字段
					if (field.name.length){
						parts.push(encodeURIComponent(field.name) + "=" + encodeURIComponent(field.value));
					}
			}
		}
		return parts.join("&");
	}
	```

	该函数接收form元素作为参数，然后拿到其中所有的元素，根据元素类型决定是否添加，若添加，则首先键值和键名都需要经过编码。

	数据经过序列化并添加到链接后，利用Ajax发送请求：

	```
	var form = document.form[0];
	var url = "http://192.168.0.1/action";
	url += url.indexOf("?") == -1 ? "?" + serialize(form) : "&" + serialize(form);
	var xhr = new XMLHttpRequest();
	xhr.onreadystatechange = function(){ // 必须在open之前
		if (xhr.readyState == 4){
			if ((xhr.status >= 200 && xhr.status < 300) || xhr.status == 304){
				alert(xhr.responseText);
			} else {
				alert("Request was unsuccessful: " + xhr.status);
			}
		}
	};
	xhr.open("get", url, true);
	xhr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded"); // 必须在open之后
	xhr.send(null); // null是必须的
	```

	请求的链接如下：

	```
	http://192.168.0.1/action?name=lc%2B%2B&sex=male&fruit=apple&fruit=peach&hobby=basketball&hobby=badminton
	```

2. **XMLHttpRequest 1级 POST方式提交**

	```
	var form = document.getElementById('form');
	var url = "http://192.168.0.1/action";
	var xhr = new XMLHttpRequest();
	xhr.onreadystatechange = function(){ // 必须在open之前
		if (xhr.readyState == 4){
			if ((xhr.status >= 200 && xhr.status < 300) || xhr.status == 304) {
				alert(xhr.responseText);
			} else {
				alert("Request was unsuccessful: " + xhr.status);
			}
		}
	};
	xhr.open("post", url, true);
	xhr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded"); // 必须在open之后
	xhr.send(serialize(form)); // 发送的是序列化后的数据
	```

	请求的链接为：

	```
	http://192.168.0.1/action
	```

	下面是POST发送的表单数据（已经由浏览器解析，Chrome下点击view source可以看未经解析的数据）

	![XMLHttpRequest1 post](/imgs/2017-10-09-post-xml2.png)

	以上是XMLHttpRequest 1 级的两种提交表单数据的方式，但有一个明显的缺陷：因为文件无法被序列化，所以无法传递，那么类似文件上传这样的功能就无法实现。这个问题在XMLHttpRequest 2 级得到了解决（XMLHttpRequest 2级并非为解决这个问题才出现的，不过它确实使这个问题得到了解决）

3. **XMLHttpRequest 2级提交表单**

	```
	var button = document.getElementById('submit')
	button.onclick = function(e) {
		e.preventDefault();
		var form = document.getElementById('form');
		var url = "http://192.168.0.1/action"; // Ajax请求链接
		var xhr = new XMLHttpRequest();
		xhr.onreadystatechange = function(){ // 必须在open之前
			if (xhr.readyState == 4){
				if ((xhr.status >= 200 && xhr.status < 300) || xhr.status == 304){
					alert(xhr.responseText);
				} else {
					alert("Request was unsuccessful: " + xhr.status);
				}
			}
		};
		xhr.open("post", url, true);
		// 直接传入表单元素，构造FormData对象，该对象会自动序列化表单内容
		// 利用send方法传递数据，无需设置头部信息，FormData会根据表单内容自动设置合适的格式
		xhr.send(new FormData(form));
	}
	```

	以上代码利用了XMLHttpRequest 2级的FormData类型，其构造函数接收form元素作为参数，构造出一个FormData实例，将该实例传入send函数，即可将经过序列化的表单数据通过Ajax发送出去。FormData对象具有一个append方法，接收键和值作为参数，添加的键值对和序列化的表单数据并无区别。比如：

	```
	var data = new FormData(form);
	data.append("job", "engineer");
	```

	在序列化的表单数据基础上，又添加了一个值为“engineer”的“job”数据。

	因为经过序列化的表单数据必须经过send方法发送，所以，使用FormData对象，就只能使用POST方法发送请求。另外，在版本较低的浏览器中，使用FormData类型可能存在兼容性问题，最好先做能力检测，并做好向后的兼容措施。

	(关于XMLHttpRequest 1级和2级更详细的内容，请参考《Javascript高级程序设计》第21章第一节和第二节)

	以上便是我对表单提交方式的总结，如有错误或遗漏，欢迎纠正补充！





