---
title: __proto__ 和 prototype
date: 2020-11-08 14:48:51
tags:
catagories:
- [JS]
- [前端]
- [All In Code]
---

## 前言 🎤
`__proto__` 和 `prototype` 两个直接让一般人去世的问题。今天就来好好研究一番。
<!-- more -->
先看两点
- `__proto__`指向构造函数的`prototype`
- `prototype`你可以认为是一个函数模版，用于存放自身的属性以及方法，以及他的上一个原型。

为什么是这样？
首先`__proto__`的作用是创建一个原型链，要通过`__proto__`，你才能不断的找到所谓的父原型。如果用代码来表示，应该是这样。
```js
let prototype = {
    //@type {prototype}
    __proto__
    //some methods or some property
    //etc
    getName:function(){}
}
```
可能还有人会云里雾里。其实你换一个角度想一下，他为什么叫链？举个例子，假设现在让你写一个链表，存放的数据为number，你会怎么写？一般情况下，想出来的内容应该是这样。
```js
let Node = function(val){//prototype
    this.next;// __proto__
    this.val = val;// prototype content
}

let head = new Node(1)
head.next = new Node(2)
console.log(head)
```
是不是非常的有道理？所以，`__proto__`都会指向上一个`prototype`。而`prototype`中的内容，就包含了一个实例中的各种成员以及属性。
当然这个例子并不是说`prototype`的实现方法就是这样。这只是说明了为什么会有`__proto__`和`prototype`而已。

## 关➡️系 
先看一张图
{% img /images/proto-和-prototype-2020-11-08-15-29-52.png %}
[来源](http://www.mollypages.org/tutorials/jsobj_full.jpg)
通过这张图片，我们可以总结几点内容
- 所有`函数对象`的`__proto__`都在`Function.prototype`上。
- 所有`函数对象`的`prototype`都指向`自身的prototype`（废话）。
- 所有`实例对象(被new 或者其他方式)`的`__proto__`都指向`构造实例对象的函数对象的prototype(也就是构造函数的prototype)`
- `Object.prototype`的`__proto__`为null

结合上面链表的例子，你看懂了吗？
如果可以不恰当的比喻，那么就是`__proto__`指向了一个模版，`prototype`的内容就是这个模版的内容。而且模版的内容中，也包含了上一个模版的“指针”--`__proto__`。

当然，这里有个特殊情况。
```js
let f = function(){}
function f(){}
let f = new Funtion()
```
这三种情况所创建的内容，你必须要把他们统一称之为一个**由构造函数为Function所创建都是实例对象，同时也是一个新prototype的构造函数**。而js中，所有的对象的构造函数，一定在Function的链上。同样的`Object`,`Array`也都是这样。
当然，按照我的理解，你可以简单的认为。凡是可以`new`的东西，都是`构造函数`，同时也都在`Function.prototype`的链上。再回去看看那张图，你是不是能获得更多的启发？
同时，你还可以这么理解。
```js
Function.prototype is class{}
Function is class.constructor
```
所以，所有的构造函数，包括`Function`自己，都在`Function.prototype`的原型链上。如果这句话你理解了，那么你可以自动的过滤掉我上面所说的特殊情况，js的原型链，除了`Object.prototype.__proto__ === null`。其他的都遵循着`构造函数`的`__proto__`的链中，一定包含着`Function.prototype`，没有例外。(当然...如果你直接修改`f.__proto__ = null`，我也是没办法拦着你的😄)。

那么现在举个小例子你看你能不能搞明白？
```js
Number instanceOf Number === false
```
如果搞明白的，可以直接看结语，如果搞不明白的，甚至不知道instanceOf是怎么判定的,可以看我分析。
首先实现一个jsInstanceOf函数
```js
/**
* @param {object} o
* @param {Contructor Function} i
*/
function jsInstanceOf(o,i){
    // if typeof o != 'object'.....
    let p = o.__proto__
    while(p != null){
        if(p == i.prototype){
            return true
        }
        p = p.__proto__
    }
    return false
}

// > jsInstanceOf(Number,Number)
// false
// > jsInstanceOf(Number,Function)
// true
// > jsInstanceOf(Object,Function)
// true
// > jsInstanceOf(Function,Object)
// true
```
还记得上面的链表例子吗？
```js
let Node = function(val){//prototype
    this.next;// __proto__
    this.val = val;// prototype content
}
```
你觉得`Node.next === Node`合理吗？

## 结语 👨‍🏫
很多人不能搞清楚为什么`Object instanceOf Function === true`和`Function instance of Object === true`。其实搞明白`Function`和`Object`其实都是`构造函数`后，就没那么多问题了。
所以，能真正决定一个对象功能的，是他指向的`prototype`。而他的`__proto__`则是用来找到`prototype`。