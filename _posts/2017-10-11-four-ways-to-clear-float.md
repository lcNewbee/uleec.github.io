---
layout: post
title: "清除浮动的四种方式及其原理理解"
date: 2017-10-10
---

## 一、为什么要清除浮动

在讲清除浮动的方法之前，我们先来了解一下为什么要清除浮动，清除浮动的目的是什么，即，要解决什么样的问题。来看一个浮动的例子(略去了文字内容)：

```
<div class="topDiv">
	<div class="floatDiv">float left</div>
	<div class="textDiv">...</div>
</div>
<div class="bottomDiv">...</div>
```

其样式为：

```
.topDiv {
	width: 500px;
	border: 2px solid black;
}
.floatDiv {
	width: 100px;
	height: 100px;
	border: 2px dotted red;
	color: red;
	margin: 4px;
	float: left;
}
.bottomDiv {
	width: 500px;
	height: 100px;
	margin: 5px 0;
	border: 2px dotted black;
}
.textDiv {
	color: blue;
	border: 2px solid blue;
}
```

在chrome中渲染的效果如下图所示：

![float example](/imgs/2017-10-10-float-example.png)

这肯定不是我们想要的渲染效果，它可能存在如下问题：

1. 文字围绕浮动元素排版，但我们可能希望文字（textDiv）排列在浮动元素下方，或者，我们并不希望textDiv两边有浮动元素存在。
2. 浮动元素排版超出了其父级元素（topDiv），父元素的高度出现了塌缩，若没有文字高度的支撑，不考虑边框，父级元素高度会塌缩成零。
3. 浮动元素甚至影响到了其父元素的兄弟元素（bottomDiv）排版。因为浮动元素脱离了文档流，bottomDiv在计算元素位置的时候会忽略其影响，紧接着上一个元素的位置继续排列。

解决第一个问题，需要清除textDiv周围的浮动，而解决第二个问题，因为父元素的兄弟元素位置只受父元素位置的影响，就需要一种方法将父级元素的高度撑起来，将浮动元素包裹在其中，避免浮动元素影响父元素外部的元素排列。

接下来开始介绍清除浮动的方法。

## 二、清除浮动的方法

### 1. 利用clear样式

还是开篇的例子，我们给需要清除浮动的元素添加如下样式：

```
.textDiv {
	color: blue;
	border: 2px solid blue;

	clear: left;
}
```

清除浮动后的渲染效果如下：

![clear method](/imgs/2017-10-10-clear-method.png)

解释一下：

通过上面的样式，textDiv告诉浏览器，我的左边不允许有浮动的元素存在，请清除掉我左边的浮动元素。然而，因为浮动元素（floatDiv）位置已经确定，浏览器在计算textDiv的位置时，为满足其需求，将textDiv渲染在浮动元素下方，保证了textDiv左边没有浮动元素。同时可以看出，父元素的高度也被撑起来了，其兄弟元素的渲染也不再受到浮动的影响，这是因为textDiv仍然在文档流中，它必须在父元素的边界内，父元素只有增加其高度才能达到此目的，可以说是一个意外收获。(clear的值为both也有相同的效果，通俗理解就是，哪边不允许有浮动元素，clear就是对应方向的值，两边都不允许就是both)

但是，如果我们把HTML中的floatDiv和textDiv交换一下位置呢？

```
<div class="topDiv">
	<div class="textDiv">...</div>
	<div class="floatDiv">float left</div>
</div>
<div class="bottomDiv">...</div>
```

无论textDiv是否应用清除浮动，情况都是下面的样子：

![exchange div](/imgs/2017-10-10-clear-exchange-div.png)

textDiv的位置先确定了，于是浮动元素就紧接着textDiv下方渲染在父元素的左侧。然而，父元素的高度并没有被撑起来，没有将浮动影响“内化”，导致浮动影响到了接下来的元素排版。

看来，为达到撑起父元素高度的目的，使用clear清除浮动的方法还是有适用范围的。我们需要更加通用和可靠的方法。

（这里澄清一下，单从元素清除浮动的角度，clear完全已经达到了目的，它已经使得textDiv特定的方向上不再有浮动元素，清除浮动其实仅仅针对需要清除浮动的元素本身而言，只关注自身需求是否达到，和外界没有什么关系，它不关注浮动是否超出父元素，以及浮动是否影响到后续元素排列。我们只是利用了浮动的一些特性达到某些目的，但这不是清除浮动关心的问题，只不过，相对于清除浮动，我们可能更加关心这些特性能为我们做些什么而已。我的理解是，清除浮动和撑起父元素高度其实是两个不同的问题，在这里，可以简单地理解为工具和目的之间的关系，接下来要讨论的两个方法都是在利用清除浮动这个工具在解决问题，它并不是清除浮动这个工具本身。不过，我们经常将两者混为一谈。sorry,有点啰嗦。。。）

### 2. 父元素结束标签之前插入清除浮动的块级元素

HTML结构如下，在有浮动的父级元素的末尾插入了一个没有内容的块级元素div：

```
<div class="topDiv">
	<div class="textDiv">...</div>
	<div class="floatDiv">float left</div>
	<div class="blankDiv"></div>
</div>
<div class="bottomDiv">...</div>
```

应用样式：

```
.topDiv {
	width: 500px;
	border: 2px solid black;
}
.floatDiv {
	width: 100px;
	height: 100px;
	border: 2px dotted red;
	color: red;
	margin: 4px;
	float: left;
}
.bottomDiv {
	width: 500px;
	height: 100px;
	margin: 5px 0;
	border: 2px dotted black;
}
.textDiv {
	color: blue;
	border: 2px solid blue;
}
// 区别在这里
.blankDiv {
	clear: both; // or left
}
```

渲染效果如下：

![add children div](/imgs/2017-10-10-clear-add-div.png)

原理无需多讲，和第一个例子里textDiv应用clear清除浮动，撑起父级元素高度的原理完全一样。这里强调一点，即，在父级元素末尾添加的元素必须是一个**块级元素**，否则无法撑起父级元素高度。

### 3. 利用伪元素（clearfix）

HTML结构如下，为了惯例相符，在topDiv的div上再添加一个clearfix类：

```
<div class="topDiv clearfix">
	<div class="textDiv">...</div>
	<div class="floatDiv">float left</div>
</div>
<div class="bottomDiv">...</div>
```

样式应用如下：

```
// 省略基本的样式
// 区别在这里
.clearfix:after {
	content: '.';
	height: 0;
	display: block;
	clear: both;
}
```

该样式在clearfix，即父级元素的最后，添加了一个:after伪元素，通过清除伪元素的浮动，达到撑起父元素高度的目的。注意到该伪元素的display值为block，即，它是一个不可见的块级元素（有的地方使用table，因为table也是一个块级元素）。你可能已经意识到，这也只不过是前一种清除浮动方法（添加空白div）的另一种变形，其底层逻辑也是完全一样的。前面的三种方法，其本质上是一样的。

### 4. 利用overflow清除浮动

首先直观地看看，overflow是如何清除浮动的。

HTML结构如下：

```
<div class="topDiv">
	<div class="floatDiv">float left</div>
	<div class="textDiv">...</div>
</div>
<div class="bottomDiv">...</div>
```

样式应用如下：

```
.topDiv {
	width: 500px;
	padding: 4px;
	border: 2px solid black;

	// 区别在这里
	overflow: auto;
}
.floatDiv {
	width: 100px;
	height: 100px;
	border: 2px dotted red;
	color: red;
	margin: 4px;
	float: left;
}
.bottomDiv {
	width: 500px;
	height: 100px;
	margin: 5px 0;
	border: 2px dotted black;
	clear: both;
}
.textDiv {
	color: blue;
	border: 2px solid blue;
}
```

不应用上面标识出来的CSS时，渲染结果和本文开始的第一个图形效果相同，应用CSS后的渲染效果如下：

![clear method](/imgs/2017-10-10-clear-overflow.png)

仅仅只在父级元素上添加了一个值为auto的overflow属性，父元素的高度立即被撑起，将浮动元素包裹在内。看起来，浮动被清除了，浮动不再会影响到后续元素的渲染（严格讲，这和清除浮动没有一点关系，因为不存在哪个元素的浮动被清除，不纠结这个问题）。其实，这里的overflow值，还可以是除了"visible"之外的任何有效值，它们都能达到撑起父元素高度，清除浮动的目的。不过，有的值可能会带来副作用，比如，scroll值会导致滚动条始终可见，hidden会使得超出边框部分不可见等。那它们是如何做到浮动清除的呢？

要讲清楚这个解决方案的原理，有一个概念始终是绕不过去，那就是[块格式化上下文(BFC)](https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Block_formatting_context),然而这又是一个非常抽象的概念，如果要清楚地把这个概念讲出来，恐怕需要非常大的篇幅，这里仅提及和理解该问题相关的内容。

这是从[MDN](https://developer.mozilla.org/en-US/docs/Web/Guide/CSS/Block_formatting_context)上摘下来的BFC定义：

>A block formatting context is a part of a visual CSS rendering of a Web page. It is the region in which the layout of block boxes occurs and in which floats interact with each other.

翻译过来就是：块级格式化上下文是CSS可视化渲染的一部分。它是一块区域，规定了内部块盒的渲染方式，以及浮动相互之间的影响关系。

块格式化上下文（BFC）有下面几个特点：

1. BFC是就像一道屏障，隔离出了BFC内部和外部，内部和外部区域的渲染相互之间不影响。BFC有自己的一套内部子元素渲染的规则，不影响外部渲染，也不受外部渲染影响。
2. BFC的区域不会和外部浮动盒子的外边距区域发生叠加。也就是说，外部任何浮动元素区域和BFC区域是泾渭分明的，不可能重叠。
3. BFC在计算高度的时候，内部浮动元素的高度也要计算在内。也就是说，即使BFC区域内只有一个浮动元素，BFC的高度也不会发生塌缩，高度是大于等于浮动元素的高度的。
4. HTML结构中，当构建BFC区域的元素紧接着一个浮动盒子时，即，是该浮动盒子的兄弟节点，BFC区域会首先尝试在浮动盒子的旁边渲染，但若宽度不够，就在浮动元素的下方渲染。[[看这里]](https://www.w3.org/TR/CSS2/visuren.html#bfc-next-to-float)

有了这几点，就可以尝试解释为什么overflow（值不为visible）可以清除浮动了。

当元素设置了overflow样式，且值不为visible时，该元素就建构了一个BFC(哪些情况下，元素可以建构出BFC，可以看查看CSS文档对[BFC的定义](https://www.w3.org/TR/CSS2/visuren.html#block-formatting))。在我们的例子中，topDiv因设置了值为auto的overflow样式，所以该元素建构出一个BFC，按照第三个特点，BFC的高度是要包括浮动元素的，所以topDiv的高度被撑起来，达到了清除浮动影响的目的。(至于为什么值为visible的overflow不能建构BFC，[这个答案](https://stackoverflow.com/questions/9943503/why-does-css2-1-define-overflow-values-other-than-visible-to-establish-a-new-b?answertab=votes#tab-top)给了一个解释)

其实，这里overflow的作用就是为了构建一个BFC区域，让内部浮动的影响都得以“内化”。如果你看了[BFC的定义](https://www.w3.org/TR/CSS2/visuren.html#block-formatting)，你会发现，构建一个BFC区域的方法有很多种，overflow只是其中的一种，那在这里，我们是否也可以利用其它的方式构建BFC，且同样能达到清除浮动的目的呢？

定义中说，inline-block同样也能构建BFC，那我们就用该样式来试试：

```
.topDiv {
	width: 500px;
	padding: 4px;
	border: 2px solid black;

	// 区别在这里
	display: inline-block;
}
// 其他样式相同，省略
```

渲染效果如下：

![inline-block clear](/imgs/2017-10-10-clear-inline-block.png)

效果完全一样！只要我们理解了原理，就可以灵活演变出不同的清除浮动的方法，而不必死记某种手段。

当然，要说明的是，在实际项目中选择采用哪种方式构建BFC是要具体问题具体分析的，因为要考虑到选用的样式自身的作用和影响。这个例子中，选用inline-block和选用overflow效果完全一样，没有看出有什么副作用，但不代表在其他项目中一样能行得通。甚至对overflow值的选择也要考虑其表现和影响。在各种构建BFC的方式中，overflow方式可能是外部影响更可控的一种，我猜想这也许就是为什么普遍采用overflow来清除浮动的原因吧。

到这里，我要分享的清除浮动的方法已经讲完了。其实，如果在不同的使用场景下，对这几个方法进行拆分组合(其实是对底层原理的拆分组合)，还可以实现其他形式不同的清除浮动的方法，最重要的还是对底层原理的把握。知其然，亦知其所以然才是最有效的学习方式。

下面是文中涉及的链接汇总：

1. [MDN对BFC的定义](https://developer.mozilla.org/en-US/docs/Web/Guide/CSS/Block_formatting_context)
2. [CSS文档对BFC的定义和解释，BFC如何创建，BFC内部渲染规则](https://www.w3.org/TR/CSS2/visuren.html#block-formatting)
3. [为什么overflow的值为visible时，无法清除浮动，而其他值可以，Stack Overflow上的一个回答](https://stackoverflow.com/questions/9943503/why-does-css2-1-define-overflow-values-other-than-visible-to-establish-a-new-b?answertab=votes#tab-top)
4. [BFC紧跟浮动元素时如何渲染](https://www.w3.org/TR/CSS2/visuren.html#bfc-next-to-float)

水平有限，文章中难免会出现不准确，甚至错误的地方，期待你的宝贵意见！