---
layout: post
title: "理清JS中等于(==)和全等(===)那些纠缠不清的关系"
date: 2017-10-23
---

文章主要梳理了一下，不同类型的值在判断是否相等时，存在的各种特殊关系，并给出判断是否相等的可靠方法。

首先说明两个关系：等于不一定全等，全等则一定等于；不等于则一定不全等，不全等不一定不等于。在文章中，能用全等的地方，等于也是一定成立的；等于不成立的地方，全等也一定不成立，相信大家都能理解，不再作特殊说明。

### 判断0的符号

`0 === -0`，但是它们不完全相同，如何判断？

先看几个关系：
```
Infinity === Infinity // true
Infinity === -Infinity // false
1/-0 === -Infinity // true
1/0 === Infinity // true

```
所以有：
```
1/-0 === 1/-0 // true
1/0 === 1/0 // true
1/-0 === 1/0 //false
```
所以，0其实是有符号的，可以使用以上办法判断。

### 判断`undefined`和`null`

还是先看几组关系：

```
undefined === undefined // true
null === null // true
null == undefined // true
null === undefined // false
```
所以，如果只判断一个变量值是否为`null`或者变量未定义，只需使用“==”即可，但是如果要清楚地区分`null`和`undefined`，那就要进一步比较了。下面是两个判断`null`和`undefined`的方法：

```
Object.prototype.toString.call(null) === "[object, Null]"
Object.prototype.toString.call(undefined) === "[object, Undefined]"

// 还有一个关系注意一下，我看有些面试题会问到：
typeof null === "object"
typeof undefined === "undefined"
```
### 判断正则表达式
两个完全一样的正则表达式其实是不相等的：

```
var a = /[1-9]/;
var b = /[1-9]/;
a == b // false
```
因为a,b其实是两个正则表达式对象，同样是引用类型的：

```
typeof a === "object" // true
typeof b === "object" // true
```
如果我们希望能够比较两个正则表达式内容是否一样，而不关心内存地址，那么只需要比较两个表达式字符串是否相等即可：

```
var a = /[1-9]/;
var b = /[1-9]/;
'' + a === '' + b // true
注：'' + /[1-9]/ === '/[1-9]/'
```
### 字符串的比较
这里需要区分字符串和字符串对象
如果是字符串，则直接使用“`===`”符号判断即可：

```
var a = 'a string';
var b = 'a string';
a === b //true
```
但是，对于字符串对象（引用类型），直接对比时，对比的仍然是内存地址：

```
var a = new String('a string');
var b = new String('a string');
a == b // false
```
如果关注字符串内容是否相同，则可以将字符串对象转化为字符串，再进行比较：

```
var a = new String('a string');
var b = new String('a string');
'' + a == '' + b // true

// 也可以使用toString方法比较：
a.toString() === b.toString() // true
```
所以，判断两个字符串内容是否相同，最可靠的办法是：

```
function isStringEqual(a, b) {
    return '' + a === '' + b;
}
```
### 数字的比较
同样需要区分数值和数值对象：

```
var a = new Number(5);
var b = new Number(5);

// 直接对比时不相等
a == b //false

// 使用+符号，转化成数值的对比
+a === +b //true
```
但是，有一个特殊的值必须特殊对待，即`NaN`，它也是`Number`类型的

```
Object.prototype.toString.call(NaN) // "[object Number]"
typeof NaN // "number"
```
同时，它的如下关系导致了以上判断数值是否相等的方法出现了例外：

```
NaN == NaN //false
+NaN == +NaN // false
注：+NaN还是NaN
```
如何在两个数值都是`NaN`的情况下判断两者是相等的呢？看一个命题：对于任意非`NaN`的数值对象或数值（a），`+a === +a`始终成立，假如该等式不成立，则a即为`NaN`。所以，如果已知a为`NaN`，如何在b也是`NaN`时，希望判断两者是相等的呢？

```
if(+a !== +a) return +b !== +b;
```
解释如下：

假设a为`NaN`，判断条件成立，如果b也是`NaN`，返回语句的表达式成立，返回`true`，表示两者相等（都是`NaN`）；如果b不是`NaN`，返回语句的表达式不成立，返回`false`，表示两者不相等。

将以上判断的逻辑整合为一个判断函数，即：

```
function isNumberEqual(a, b) {
    if (+a !== +a) return +b !== +b; // 处理特殊情况
    return +a === +b;
}
```
### Date对象对比
对象的对比也不能直接使用等号判断，我们还是只关心日期值是否相同，所以，将日期转化为毫秒数值，然后对比数值是否相同

```
var a = new Date('2017-9-7');
var b = new Date('2017-9-7');
a == b //false
+a === +b //true
注：+a的值为1504713600000，即，对应2017.9.7 00:00:00的毫秒数

效果和使用getTime()方法对比一样
a.getTime() === b.getTime() // true
```
### 布尔值对象的对比
对象不能直接对比，故，将布尔值转化为数值，然后对比

```
var a = new Boolean('123');
var b = new Boolean('123');
a == b //false
+a === +b //true
// 注： 布尔值为真，前面加“+”，转化为数值1，为假则转为0
```
文中很多对比的方法，其实都可以找到对应的对象方法实现值的对比，本文最主要的还是提醒各位，在对比时，注意区分对比的是值还是对象，如果目的是对比值是否相等，则要进一步转化。另外，特别注意各类型的值在对比时存在的特殊情况，以及特殊值之间的关系。

以上，希望对你有用。欢迎留言纠错或补充。
