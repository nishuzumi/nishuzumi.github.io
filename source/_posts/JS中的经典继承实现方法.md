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

### Prototype可以类似的理解成一个实例对象
- prototype可以被对象随意覆盖
- 可以随时修改prototype的属性
- prototype中的修改会影响到所有继承当前prototype的对象。
- prototype可以当成一个对象来进行操作

但是！！请注意，你在对原型赋值的时候，并不是在你想的prototype进行赋值。

这里其实有个非常非常神奇的特性。

### Prototype != Prototype
当你进行对某个构造函数赋值对象时，真正获得目标参数的，是__proto__，而不是它真正意义上的prototype。
```js
function A(){}
A.prototype.x = 1

function B(){}
B.prototype = new A() //Object A
// B.prototype.__proto__ === Object A
```
但是如果你对他进行直接赋值prototype，情况又回不一样。
```js
function A(){}
A.prototype.x = 1

function B(){}
B.prototype = A.prototype
B.prototype.x = 2
// A.prototype.x === 2
// B.prototype.x === 2
```
此时的双方共享一个prototype，即使B修改prototype，A也会收到影响。

所以，我认为，在对prototype进行赋值的时候，如果赋值内容为一个prototype，那么他们会共享一个prototype内容，如果赋值的内容为`new Object`，那么赋值的时候，`__proto__`会被赋值为`(new Object)`。
有人会疑惑，为什么我不写成：赋值的对象为一个实例对象？
因为，如果你赋值一个实例对象的时候，结果又会发生变化。
```js
function A(){}
A.prototype.x = 1

function B(){}
let c = new A()
B.prototype = c
B.prototype.x = 2
console.log(A.prototype.x,B.prototype.x) // 1 2
console.log(c.x) // 2
```
当然，还有人会想，如果我直接赋值一个构造对象，会怎么样？  
这里的结果和上面其实是一样的，整个prototype的内容都塞满了构造对象中的各种属性。  

自然，有些聪明的小伙伴可能想到了，如果我修改`prototype.__proto__`，岂不是就修改了A的prototype？  
没错，你的猜想完全正确🙆‍♂️。
```js
function A(){}
A.prototype.x = 1

function B(){}
B.prototype = new A() //Object A
B.prototype.x = 2
B.prototype.__proto__.x = 3
console.log(B.prototype.x,A.prototype.x) // 2 3
```

### Prototype结论
通过上面的论述，我们可以发现，prototype的内容赋值可能并不完全由js进行掌控。很明显的是，这部分的内容会随着代码而改变。也就是说，只有当你`xx.prototype = new XX()`或者`xx.prototype = Object.create(XX.prototype)`时，你才能获得正确的结果。否则，都将会变成错误的答案。

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

## 结语 👨‍🏫
prototype真的是js中非常重要的一个部分，我觉得每个前端开发者都需要好好理解掌握prototype中的每一个细节。