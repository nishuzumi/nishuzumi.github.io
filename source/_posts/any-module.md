---
title: 小结AMD，CMD，UMD，CommandJS，ES Module
date: 2020-11-17 09:16:49
tags:
- [JS]
- [前端]
- [All In Code]
---
## 前言 🎤
JS中，最为伟大的，也可说是最为可赞的就是劳动人民的双手，他们让JS原本不带有的功能，用各种各样的神奇方式，把这些功能都实现出来。而模块加载，也是JS进化历程上的一部可圈可点的赞歌。  

<!--more-->
## 鼻祖CommandJS
CommandJS是服务端JS的标准规范，特点是只能用于服务端，因此才有AMD出现的机会。CommandJS核心为`require`和`module.exports`。同时加载方式为同步加载。
### 声明
```js
//math.js
module.exports.add = function(){}
```
在CommandJS中的声明方式就是对`module.exports`或者`exports`进行修改，具体差异不再赘述，可以看我之前文章。
### 使用
```js
// index.js
const math = require('./math')
math.add()
```
使用方法也异常的简单，这是因为Node在底层做了很多其他事情来保证开发者能几乎无痛的使用。
### 循环依赖
Node的循环依赖处理方式是：如果有循环，执行到哪就算哪。简单来说就是，当A模块开始加载时，缓存中会立刻出现A模块的`module.exports`，当`require(b)`，并且b中`require(a)`的时候，b只能获取到循环依赖之前的a。
```js
//a.js
module.exports.max = 10
require(b)
module.exports.max = 20
//b.js
const a = require(a)
console.log(a.max) // 10
```
## AMD
AMD是RequireJS使用的规范，是为了浏览器中的模块加载而实现的。并且AMD是异步加载。而AMD的设计思路，也是参考了一部分CommandJS的。  
AMD是声明和加载方式非常的简单易懂。  
### 声明
```js
define(['jquery', 'underscore'], function ($, _) {
    //    methods
    function a(){};    //    private because it's not returned (see below)
    function b(){};    //    public because it's returned
    function c(){};    //    public because it's returned

    //    exposed public methods
    return {
        b: b,
        c: c
    }
});
```
只有在有依赖的情况下，才需要进行定义依赖，否则可以直接传入内容，比如：
```js
// math.js
define(function(){
    return {
        add:function(){}
    }
})
```
### 使用
使用则是：
```js
require(['math'], function(math) {
    console.log(math.add());
})
```
### 循环依赖
那么AMD是如何解决循环依赖的？其实就是强制忽略了，比如有两个模块A，B。当A依赖B，然后B依赖A的时候，B获取到的A是为未定义的状态。而且总是会把依赖的模块执行完成，也就是说B一定会被先执行完成。

## UMD
UMD是一个为了解决跨平台MD导入问题，它的解决方法就是通过揉合`CommandJS`和`AMD`来解决这个问题，不得不说，思路很清晰。
```js
// if the module has no dependencies, the above pattern can be simplified to
(function (root, factory) {
    if (typeof define === 'function' && define.amd) {
        // AMD. Register as an anonymous module.
        define([], factory);
    } else if (typeof exports === 'object') {
        // Node. Does not work with strict CommonJS, but
        // only CommonJS-like environments that support module.exports,
        // like Node.
        module.exports = factory();
    } else {
        // Browser globals (root is window)
        root.returnExports = factory();
  }
}(this, function () {

    // Just return a value to define the module export.
    // This example returns an object, but the module
    // can return a function as the exported value.
    return {};
}));
```
其实它就是自定义了一种包装方式，并且对每种环境下进行不同处理。如果是有依赖的情况，一样会进行特殊情况处理。  
## CMD
CMD的存在感太低，它的各个方面和AMD都非常像，只不过CMD推崇**依赖就近**  
```js
define(function(require,exports,module){
    const a = require('a')
    a.x()
})
```
但是也是延迟执行，还是要等加载完。判断方式就是正则啦。
```js
Function.prototype.toString.call(function(){
    console.log('i am Magic')
})
```
说白了，其实就是让写法更舒服了。

## 平行世界 ES6 Module
在多维宇宙中，因为一些特殊的，细小的变动，导致在平行宇宙的地球上，ES Module成为了唯一的前后端通用模块加载规范。而2015，在第四次太阳🌞异常活动导致的宇宙波裂变，使得ES Module的灵感传递到了我们所处的地球🌍上来。因为传递时间是2015年，所以这个功能也称为`ES2015 Module`或者`ES6 Module`。——我瞎掰的。  
`ES Module`的诞生可以说是真正意义上的解决了前后端模块导入的问题，不过生不逢时，并没有能彻底的取代Node的CommandJS的地位，但是也是一种可喜可贺的划时代创新。  
### 声明
`ES6 Module`的声明喜闻乐见，这里简单说说
```js
// math.js
export function add(){}

export default {
    add
}

export var name = 'box'
```
### 使用
```js
import {add} from './math'
add()
```
### 循环依赖
`ES6 Module`其实并不关心有没有循环依赖，他并不需要产生结果，他只需要给你一个引用即可，至于是否能取到值，那么就需要开发者自己来保证了。

## 奇点🎆 Webpack
Webpack带来的影响可以说是翻天覆地，你基本不用再关心什么模块导入方式，伟大的Webpack会使用他全知全能的处理方式帮你解决这些问题，你再也不用担心如何在服务端使用`ES Module`,或者在前端使用`CommandJS`(Browserify)。

## 总结 👨‍🏫
如果看不全可以👉滑动
|*|AMD|CommandJS|UMD|CMD|ES Module|
|-|---|---------|---|---|---------|
|使用在浏览器|✅|❌|✅|✅|✅|
|使用在服务端|❌|✅|✅|❌|✅|
|异步加载|✅|❌|✅|✅（允许）|✅（允许）|

同时，只有`ES Module`为传递**符号引用**，其余均为对象。