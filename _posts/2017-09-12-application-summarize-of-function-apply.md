---
layout: post
title: "利用apply提高编程效率的方法总结"
date: 2017-09-12
---

合理的使用apply/call函数能够帮助我们极大地简化代码，高效地解决问题。本文尝试总结apply方法的几种用法，并发现一点规律，以便在需要时能够想到该方法。因apply和call方法的用法几乎相同，差别仅在于参数的传入方法，故本文仅以apply方法为例。

### 方法介绍

apply是函数对象原型的一个方法（Function.prototype.apply），它能够改变函数在运行时的this指向，即，能够改变函数运行时的执行环境。该函数最多接受两个参数，第一个参数指定函数运行时的this指向，决定了执行环境，第二个参数为，将要传入函数的参数组成的数组或者类数组对象（如果是call函数，则参数直接传入）。

下面是一个常见的例子：

    var dog = {
        sound: 'wang',
        makeSound: function() {
            console.log(this.sound);
        }
    }

    var cat = {
        sound: 'miao',
    }

    dog.makeSound.apply(cat);

例子中分别定义了一个dog对象和一个cat对象，它们都有sound变量，但是dog对象有一个makeSound方法，而cat没有，如果这里要求cat也要能够发声（makeSound），有两种直观的解决方案：

第一，给cat直接赋予一个makeSound方法

    var cat = {
        sound: 'miao',
        makeSound: function() {
            console.log(this.sound);
        }
    }

第二，利用原型继承

    var Pets = function(sound) { this.sound = sound; }
    Pets.prototype.makeSound = function() { console.log(this.sound); }

    var dog = new Pets('wang');
    var cat = new Pets('miao');
    cat.makeSound(); // miao

但是，有了apply，一切都变得很简单:

    dog.makeSound.apply(cat);// miao

cat既不需要有自己的makeSound方法，也不需要和dog继承自同一个父类，只要makeSound自己能用，那就可以拿来用。
以下的所有用法，其实都是针对apply方法两个方面的特性而来的：
1. 可以传入this，改变函数运行时的执行环境。
2. apply的第二个参数是函数参数组成的数组或者类数组，且被借用函数是以散列形式传参。

### **几种用途**

1. **类数组借用数组方法**

    类数组虽然能和数组一样使用下标索引，但是它不具有数组拥有的内置方法，无法直接使用这些方法，幸运的是，类数组的特性决定了数组的方法也能在其对象上使用，这就给apply方法发挥的空间：

        // 错误例子
        var addOne = function() {
            return argument.map(function(a) {return a+1;});
        }
        var arr = addOne(1,2,3,4) // Uncaught TypeError: arguments.map is not a function

        // 正确的例子
        var addOne = function(){
            var arr = Array.prototype.slice.apply(arguments);
            return arr.map(function(a){return a+1;});
        }
        addOne(1,2,3,4); // [2,3,4,5]
        // 这里使用call更加简单
        var addOne = function() {
            return Array.prototype.map.call(arguments, function(a){ return a+1; })
        }

    上面的addOne函数将所有传入的参数分别加1，然后组成数组返回，函数的arguments就是由参数构成的一个类数组，我们无需取出这些参数再一个个加1，再push进数组，而是将类数组先转化为数组，然后使用数组的map方法。第一个错误的示例表明了类数组不具有数组方法，所以报错。

    在dom操作中，如document.getElementsByClassName, document.querySelectAll等方法拿到的对象都是类数组类型的，一般来讲，只要转化成数组类型就会极大地方便我们的操作。

2. **求数组的最大最小值**

    求数组中的最大最小值操作是很常见的，但是，数组并没有为我们实现这样的操作，最直观的方法就是遍历数组，查找最大最小值，无疑，这种方法不仅笨拙低效，而且性能极差。但是，如果给你的不是一个数组，而就是一些数字呢？你可能会立即想到Math.max方法。这两种形式参数的关系恰好就符合我们之前提到的两个特性之二：函数要求以散列形式传参，apply又要求以数组或类数组传参。

        Math.max.apply(null, [1,6,5,3,5]) // 6
        Math.min.apply(null, [1,6,5,3,5] ) // 1

3. **准确判断对象类型**

    判断对象类型，我们有typeof函数可用，但是它的判断并不可靠，比如，对数组进行typeof操作，返回的却是“object”。而在Object的原型对象上，有一个toString方法，它作用在不同类型的对象上，返回特定的字符串，根据返回值可以准确地判断对象类型。

        Object.prototype.toString.apply([]) // "[object, Array]"
        Object.prototype.toString.apply({}) // "[object, Object]"
        Object.prototype.toString.apply(undefined) // "[object, Undefined]"
        Object.prototype.toString.apply(function(){}) // "[object, Function]"
        Object.prototype.toString.apply(document.getElementsByClassName(div)) // "[object, NodeList ]"

    该判断方法可以支持如下类型的判断：NodeList, Window, Object, String, Infinity, Number(NaN), Function, HTMLDocument, Undefined, Boolean。需要特别注意的是Number类型的判断，NaN和Infinity 也会被识别为Number类型（可用如下规则判断：1/0 === Infinity, 1/-0 === -Infinity, NaN != NaN）。这里因为不涉及第二个参数的问题，所以使用call也完全是可以的。

4. **二维数组的扁平化**

    先来看看数组的concat方法的用法：

        var a = [1,2,3];
        var b = [4,5,6];
        var c = a.concat(7,8,9) // [1,2,3,7,8,9]
        var d = a.concat(b) // [1, 2, 3, 4, 5, 6]
        var f = a.concat(7,8,b,9) //[1, 2, 3, 7, 8, 4, 5, 6, 9]
        a // [1,2,3]
        b // [4,5,6]

    可以发现，concat既可以接受数组也可以接受散列参数，而且最终生成的结果都是一样的，都是一个一维的数组，同时，不改变原来的数组。如果将计算f的参数组成一个数组，那么就是一个二维数组，再结合apply接受数组作为第二个参数的特性，就可以实现一个二维数组的扁平化功能了：

        var twoDemArr = [[1,2,3], [4,5,6], 7,8,9]
        var arr = Array.prototype.concat.apply([],twoDemArr);
        arr // [1,2,3,4,5,6,7,8,9]

    进一步，我们看看那些接受散列参数的数组方法，如果结合apply会有什么样的作用：

    push也接受散列参数，它将参数推入数组，并且改变了原数组，那么它就可以实现，将一个数组的元素推入另外一个数组，并且改变被推入数组：

        var a = [1,2,3]
        var b = [4,5,6]
        Array.prototype.push.apply(a, b);
        a // [1,2,3,4,5,6]

    unshift和push一样，不过是将元素加在数组前面：

        var a = [1,2,3]
        var b = [4,5,6]
        Array.prototype.unshift.apply(a, b);
        a // [4,5,6,1,2,3]

5. **修正内部函数的this指向**

    在函数内部定义的函数，如果直接调用，则该内部函数的this并不指向外层函数的this，而是指向全局执行环境，所以调用内部函数必须指明其this的指向

        document.getElementById('div1').onclick = function() {
            alert(this.id) // div1
            var func = function() {
                alert(this.id);
            }
            func(); // undefined
        }
        document.getElementById('div1').onclick = function() {
            alert(this.id) // div1
            var func = function() {
                alert(this.id);
            }
            func.apply(this); // div1
        }

    当然，在外层保存this变量，然后在内部函数定义中直接使用保存的变量也可以达到相同的效果，但是使用apply的方法相对而言，保持了内部函数的独立性。

6. **给既有方法打补丁**

    下面这个例子来自MDN的apply词条页面

        // 保存原函数
        var originalfoo = someobject.foo;
        someobject.foo = function() {
            // 在这里添加需要在原函数调用前执行的操作
            console.log(arguments);
            // 调用原函数
            originalfoo.apply(this, arguments);
            // 在这里添加需要在原函数调用后执行的操作
        }

    上面的例子，在调用原来的方法之前或者之后，执行了新的操作，补充增强了原来的方法，而且不改变原来的操作。这也是设计模式中装饰者模式的实现思路。
    举一个应用场景：假如你从上一位开发者手中接过了一个项目，你需要在不改动原来功能的基础上开发一个新功能，你找到了这个功能的函数位置，但是因为代码组织很糟糕，你几乎看不懂这段代码做了什么，所以也不敢轻易改动。怎么办？也许上面利用apply打补丁的方法值得一试。只需在调用前后添加新操作，然后按照其调用方法调用，完全不用关心原函数的实现细节如何。

善用apply和call方法，可以大幅提高我们编程的效率，提升程序性能。这里仅仅总结了一部分apply的用法，apply的强大之处肯定远远不止如此，更多的用法还有待我们进一步的发现和总结。
