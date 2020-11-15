---
title: JS中的经典继承实现方法与原型的深入探讨
date: 2020-11-12 12:25:04
tags:
catagories:
- [JS]
- [前端]
- [All In Code]
---
## 前言 🎤
JS在很久很久之前，其实是并没有类这种概念的，也没有明确的继承这个概念，所以聪明的人们就创造出了很多种办法，来模拟出和继承类似的效果，本文就来讲一下继承的原理和常见的继承方法（不包含Class语法糖）
<!--more-->
## 简单的定义一下主角
```js
function Tank(){
    this.speed = 10
}

Task.prototype.fire = function(){
    console.log('fire')
}

function Tiger(){
    this.armar = 10
}

Tiger.prototype.SpeedUp = function(){
    console.log('up!')
}
```

## 寄生组合式继承
```js
function Tank(){
    this.speed = 10
}

Tank.prototype.fire = function(){
    console.log('fire')
}

function Tiger(){
    Tank.call(this) // 调用Tank构造函数中的内容
    this.armar = 10
}

(function(){
    let empty = function(){}
    empty.prototype = Tank.prototype
    //复制原型 并且传递给一个新的临时函数对象
    Tiger.prototype = new empty()
})()

//子元素的prototype内容需要在上面的IIFE之后调用
Tiger.prototype.speedUp = function(){
    console.log('up!')
}

let tank = new Tiger()
console.log(tank)
tank.fire()
tank.speedUp()
/*
Tank { speed: 10, armar: 10 }
fire
up!
*/
```
这个方法非常好，没有过多的空间浪费，没有多余的调用，一步到位堪称完美。  
但是我们可以稍微升级一下

## 升级后的方法
```js
function Tank(){
    this.speed = 10
}

Tank.prototype.fire = function(){
    console.log('fire')
}

// function Tiger(){
//     this.armar = 10
// }

function Tiger(){
    Tank.call(this)
    this.armar = 10
}

// modify
Tiger.prototype = Object.create(Tank.prototype)
Tiger.prototype.constructor = Tiger

Tiger.prototype.speedUp = function(){
    console.log('up!')
}

let tank = new Tiger()
console.log(tank)
tank.fire()
tank.speedUp()
```
改动非常少，但是更加简洁了，而且更加的清晰易懂。其实原理是一样的，通过原型函数创建新的对象，然后将其作为基础进行修改。同时修改构造函数为子类。

## Object.create
什么是`Object.create`?它是一个处于ES5规范中的，用于通过原型创建对象的函数。也就是说，新对象的__proto__将会指向传入的参数原型。  
但是，为了更加升入的理解，所以我们需要实现一个Object.create来加深理解程度。

```js
Object.breate = function(p){
    if(typeof p != 'object' && typeof p != 'function'){
        throw new Error("....!%$") //加密语言
    }
    function C(){}
    C.prototype = p
    return new C()
}
```
哈哈，是不是似成相识？效果其实是完全一样的。只不过要注意一点⚠️`Object.create`的参数可以传递两个，而并非只能一个，你可以去MDN上详细阅读它的使用方法。这里不进行展开。  
有人可能认为`xx.prototype = new YY()`是错误的方法，而只有`Object.create`才是唯一的正确答案。其实这种想法非常过激，我们对错误的定义有时候太过于狭隘。如果它能得到正确的答案，它的运行状态正常而且稳定，它的运行结果并无任何偏差，那么我们怎么能说他是一种错误的方式？

## Prototype
其实在阅读上面的代码的时候，你一定会产生很多疑问🤔️，什么是**prototype**？，为什么可以传入一个对象作为prototype？为什么函数可以直接new？等等之类的问题。接下来，我将讲述我的见解。

在我研究JS的时候，我就一直在思考这些问题，随着我的不断深入，我感觉我对prototype的形象越发的了解，虽然我现在并不能100%的肯定，我理解的prototype就一定是正确的，但是我觉得八九不离十。

### Prototype是一个实例对象
- prototype可以被对象随意覆盖
- 可以随时修改prototype的属性
- prototype中的修改会影响到所有继承当前prototype的对象。
- prototype可以当成一个对象来进行操作

但是！！请注意，你在对原型赋值的时候，并不是在你想的prototype进行赋值。

这里其实有个非常非常神奇的特性。

### Prototype ?= Prototype
请注意⚠️，构造对象，也就是Function，在搞乱我们对原型链对理解这件事上面，占有很大一笔责任。  
一个普通对构造对象，它同时有着两个属性,`__proto__`表示继承的上一个原型,`prototype`表示当前构造函数中的原型。
而当他被构造后，会产生新的情况。最重要的是**构造函数中的prototype会转移到实例对象的__proto__之中**  
因此，如果你直接赋值一个构造好的实例到子类的prototype时，一切都恰恰刚好。  
而如果你在父类对象的构造函数中赋值了实例属性，那么这些属性也会成为子类prototype的一部分。
```js
//伪代码
//构造对象结构
let A = {
    prototype:{
        __proto__:{x:1},
        y:2,
        constructor:function(){}
    }
}
//实例对象结构
let newA = new A()
newA == {
    __proto__:{
        __proto__:{x:1},
        y:2,
        constructor:function(){}
    }
}
//继承了A的B
let B = {
    prototype:{
        __proto__:{
            __proto__:{x:1},
            y:2,
            constructor:function(){}
        }
    },
}
//实例化B
let newB = new B()
newB == {
    __proto__:{
        __proto__:{
            __proto__:{x:1},
            y:2,
            constructor:function(){}
        }
    }
}
```
prototype的谜题解开了。
### Prototype结论
经过多次研究发现，实际上的prototype机制异常的简单。重点就两个。
- 实例对象中不存在prototype
- 在构造对象中，如果这个对象被构造，那么构造对象的`prototype`将会变成实例对象的`__proto__`
```js
function A(){}
let a = new A()
console.log(a.__proto__ === A.prototype)
```
- 

还记得我们一开始是怎么实现继承的吗？
```js
    let empty = function(){}
    empty.prototype = Tank.prototype
    //复制原型 并且传递给一个新的临时函数对象
    Tiger.prototype = new empty()
```
结合一张图片看看
{% img /images/JS中的经典继承实现方法-2020-11-12-18-17-11.png %}  
现在明白为什么需要创建一个新函数了吗？

不过，其实这种继承有着少量的缺点，下面看一看Babel是如何实现继承的。
## Babel
```js
function _inherits(subClass, superClass) {
    // 喜闻乐见的Object.create
    subClass.prototype = Object.create(superClass.prototype, {
        //第二个参数的说明详细见MDN，这里的作用类似于 subClass.prototype.constructor = subClass
        constructor: {
            value: subClass,
            enumerable: false,
            writable: true,
            configurable: true;
        }
    })
    if (superClass) Object.setPrototypeOf ? Object.setPrototypeOf(subClass, superClass) : subClass.__proto__ = superClass; 
    // 设置subClass中__proto__属性，将其设置为superClass的构造函数
    // 这个举动的作用是将赋值于构造函数上的方法进行继承，在es6中是class{static xx(){}}中的static
}

function A(){}
function B(){}
A.StaticMethod = ()=>{} // 可以认为是静态方法
A.prototype.hello = ()=>{} // 可以认为是对象实例化后的方法
```
一个小例子
```js
function A(){}
function B(){}
B.C = function(){
    console.log('static')
}
console.log(A instanceof B)
_inherits(A,B)
let a = new A()
console.log(a instanceof B)
A.C()
/*
false
true
static
*/
```
这里其实也反应了JS原型链的机制，当你在一个对象上面调用方法时，无论是实例对象还是构造对象，JS都会从其`__proto__`中查找方法，如果找不到就在`__proto__.__proto__`中继续查找，直到为`null`。
## 结语 👨‍🏫
prototype真的是js中非常重要的一个部分，我觉得每个前端开发者都需要好好理解掌握prototype中的每一个细节。