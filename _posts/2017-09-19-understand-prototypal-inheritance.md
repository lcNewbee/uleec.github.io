---
layout: post
title: "知其然，亦知其所以然——彻底搞懂JS原型继承"
date: 2017-09-19
---

曾经，我写过下面一段代码，我满心欢喜以为得到了JS面向对象编程和原型继承的真谛。

    var pets = {
        sound: '',
        makeSound: function() {
            console.log(this.sound);
        }
    }

    var cat = { sound: 'miao' };

    cat.prototype = pets;
    cat.makeSound();

然后，我将这段代码复制粘贴到浏览器的console调试工具下运行，竟然报出一个错误。我不得不承认，原来我根本就不懂JS的面向对象编程。

![Error Image](/imgs/2017-09-19-console-error.png)

我的目的是，让cat继承pets的makeSound方法，虽然cat没有makeSound方法，但是它可以沿着原型链查找到pets的makeSound方法。但是，很明显，这段代码有错误，无法达到我的预期。我认识到，我根本就没有弄懂过prototype和__proto__属性的关系和作用。

如果你也不知道上面的代码确切的错在哪里，那你也需要补上这一课。如果你知道上面的代码错在哪里，但不知道为什么是这样的安排，这篇文章也能让你有收获，如标题所言，知其然，亦知其所以然。

为了讲清楚这个问题，我们先抛开上面的错误，从构造一个简单对象开始谈起。

### **一个宠物制造工厂**

我们将从一个简单的工厂函数开始：

    var Pets = function(sound) {
        var obj = { sound: sound };
        obj.makeSound = function() {
            console.log(obj.sound);
        }
        obj.bite = function() {
            console.log('bite');
        }
        return obj;
    }

    var dog = Pets('wang');
    dog.makeSound(); // wang

上面定义了一个宠物制造工厂，它生成了一个拥有sound属性的对象，并将接收的参数赋值sound属性。然后在该对象上添加了两个方法，最后将这个对象返回。有了这个函数，我们就可以制造出各种各样的宠物了。(为了前后一致，请忽略函数首字母大写的问题)

### **优化宠物制造工厂：统一管理方法函数**

然而，上面的工厂函数有一个缺点。如果我们想给这个工厂函数添加更多的方法，或者删除多余的方法，那么我们不得不改动这个函数本身的代码。当方法变得越来越多的时候，这个函数就变得难以维护。所以，我们进行如下优化：

    var Pets = function(sound) {
        var obj = { sound: sound };
            extend(obj, Pets.methods); // 注意，这里的extend函数是没有实现的。
            return obj;
    }

    Pets.methods = {
        makeSound: function() {
            console.log(this.sound);
        },
        bite: function() {
            console.log('bite');
        }
    }

    var dog = Pets('wang');
    dog.makeSound() // wang

可以看到，我们给Pets函数添加了一个methods属性，用来统一保存和维护该工厂函数的方法。当使用该函数生成obj对象时，通过一个extend函数将Pets.methods中的方法统统复制到obj中。这时，代码本质上没有改变什么，只是通过形式上的改变，使得代码更容易维护。如果我们想给Pets工厂函数添加新的方法，可以通过下面的方式实现，而不必修改函数：

    Pets.methods.scratch = function() {/*...*/}

### **继续优化：继承而不是复制**

上面的代码，每次调用工厂函数生成的新对象，都有一份对Pets.methods中方法的完全复制。这种创建对象的方式是低效的，既然Pets.methods中的方法是所有由工厂函数创建的对象都拥有的，我们其实并不希望每个对象都保留一份复制，而是希望通过某种方式，让所有的对象共享方法，所以就有了继承的概念。在JS中，Object.create函数可以实现继承的目的，我们将代码改写如下：

    var Pets = function(sound) {
        var obj = Object.create(Pets.methods);
        obj.sound = sound;
        return obj;
    }

    Pets.methods = {
        makeSound: function() {
            console.log(this.sound);
        },
        bite: function() {
            console.log('bite');
        }
    }

    var dog = Pets('wang');
    dog.makeSound(); // wang
    dog.bite(); // bite

Object.create构建了一个继承关系，即obj继承了Pets.methods的方法。obj内部有一个[[Prototype]] 指针，指向了Pets.methods，Pets.methods也就成了该对象的原型对象。[[Prototype]]指针是一个内部属性， 脚本中没有标准的方式访问它，但是在Chrome、 Safari、Firefox中支持一个属性__proto__,而在其他浏览器实现中，这个属性都是完全不可见的。在Chrome的调试窗口打印dog：

![Dog Print](/imgs/2017-09-19-dog-print.png)

可以看到，dog并不拥有makeSound方法，但仍然可以使用该方法，因为它可以沿着_proto_指针指明的方向继续查找makeSound方法，一旦找到同名方法就返回该方法。（任何对象都继承自Object对象，所以方法查找的终点在Object处，假如查找到达Object对象且Object对象也没有该方法，则返回undefined）

上面的改进，通过继承，将对象的公用方法委托给原型对象，每次创建新的对象时，就免去了属性的复制，提高了代码的性能和可维护性。下面，我们对代码进行一点小改动：

    var Pets = function(sound) {
        var obj = Object.create(Pets.prototype);
        obj.sound = sound;
        return obj;
    }

    Pets.prototype.makeSound = function() {
        console.log(this.sound);
    }

    Pets.prototype.bite = function() {
        console.log('bite');
    }

    var dog = Pets('wang');
    dog.makeSound(); // wang
    dog.bite(); // bite

我们把作为原型对象的Pets.methods换了一个名称，叫做Pets.prototype。是不是觉得哪里不对？怎么能这么随意的替换呢？prototype可是JS语言中很特殊的一个属性，有着某种很特别的功能，怎么可能和这里的methods一样呢？没错，这么替换，而不是一开始就使用prototype，就是想说明，其实，prototype属性并没有什么神秘的地方，它的作用和这里的methods几乎是一样的。

### **庐山真面目：构造函数**

上面的这种创建对象，并将对象方法委托到原型对象的方式，在JS编程中是如此的常见，所以语言本身提供了一个方法，将重复的部分自动处理，程序员只需要关注每个对象不相同的部分，这个方法就是，构造函数：

    var Pets = function(sound) {
        this.sound = sound;
    }

    Pets.prototype.makeSound = function() {
        console.log(this.sound);
    }

    Pets.prototype.bite = function() {
        console.log('bite');
    }
    var dog = new Pets('wang');
    dog.makeSound(); // wang
    dog.bite(); // bite
    var cat = new Pets('miao');
    cat.makeSound(); // miao
    cat.bite(); // bite

构造函数的new操作，自动处理了继承和返回操作。可以这么理解new的主要作用：

    var Pets = function(sound) {
        /* this = Object.create(Pets.methods); */
        this.sound = sound;
        /* return this; */
    }

就好像在执行new操作的时候，语言自动处理了注释部分的代码，只需要我们关注将要创建的对象的特殊部分即可。（当然，上面的代码去掉注释是无法运行的，因为this是只读的，不能赋值，浏览器运行会报错。但原理是正确的。）

prototype则为构造函数的一个属性，也是由构造函数所创建对象的原型对象。如果一定要说prototype和前面例子中的methods有什么不同，那就是，prototype有一个默认属性constructor，该属性指向构造函数本身。

    console.log(Pets.prototype.constructor === Pets) // true

顺便，你认为下面的表达式应该打印什么？

    console.log(dog.constructor)

应该是Pets，dog自身没有constructor属性，所以沿着原型链向上查找，找到Pets.prototype，而Pets.prototype是有这个属性的，返回这个属性，该属性指向构造函数Pets，所以打印Pets。

到这里，关于原型继承中涉及到的构造函数、prototype、constructor，[[Prototype]]以及创建出来的对象之间的关系已经全部呈现出来了。来做个总结：


1. prototype是构造函数的一个属性，并没有什么特殊和神秘的性质。

2. prototype是由构造函数所创建对象的原型对象，对象的公共方法和属性可以委托到prototype。

3. 再次强调，构造函数（如Pets）和prototype并不存在继承关系，继承关系存在于构造函数创建的对象和prototype之间。（Object.create建立了对象和原型之间的继承关系，和构造函数没有关系）
4. constructor是语言自动赋予prototype的一个属性，其值为构造函数本身。

5. [[Prototype]]是对象的一个内部属性，是一个指针，指向对象的原型对象，在Safari、Chrome和Firefox下，可以通过__proto__属性访问。

还是使用上面的例子，我们将所有这些关键词之间的相互关系使用一个图示展示出来：

![关系图](/imgs/2017-09-19-relationship.png)

### **错误解析：**

现在回过头去看开头提到的那个错误例子，简直就错的离谱啊。这个错误明显神化了prototype的作用，以为只要使用了prototype属性，然后就如同黑魔法一般，在两个完全不相关的对象之间架起了一座桥梁，也就是继承关系，然后就可以随意使用另外一个对象的方法了。天真！

我的问题在于，首先，神话了prototype的作用。prototype并没有这种黑魔法，它只是一个属性。
其次，没有搞明白继承关系到底存在与哪两个对象之间。（被创建对象和prototype之间）

所以，在错误的代码中，dog对象没有makeSound方法，dog对象继承Object.prototype，而非pets，而Object.prototype上并没有所谓的makeSound方法，返回undefined，所以报错。

以上，希望对你有所帮助。



