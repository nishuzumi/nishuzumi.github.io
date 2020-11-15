---
title: module.exports,exports,export小型指南🧭
date: 2020-11-15 15:48:56
tags:
- Node
- Browser
catagories:
- [JS]
- [前端]
- [All In Code]
---
## 前言 🎤
本文将会讲述`module.exports`,`exports`,`export`的完全区别和理解。

<!--more-->
## Node exports
Nodejs中使用的是`CommandJS`规范，是Nodejs原生实现的一种模块导入方式。因此，有一些Nodejs环境下的函数和常量，这里稍微举例一些：  
- require —— require不单单是一个函数，它其中还含有很多静态方法和属性，比如`require.main`,`require.resolve()`。
- __dirname,__filename —— 家喻户晓的两个常量
- module对象 —— Node模块加载的心脏。

Node下最常见最常见的问题，就是`module.exports`和`exports`有什么区别？  
简单来说`exports`是一个'别名'，至少文档是这么说的。实际上，他们两个是指向了同一个对象的变量。请注意是变量，而不是常量。所以说是可以被修改的。
这里就产生了第二个问题，到底谁主谁次？改了一个会怎么样？他们互相影响吗？  
还记得我刚刚说了什么吗，他们两个指向了同一个对象，而且`exports`是别名，那么就表示，结果将会以`module.exports`为准，在一般情况下，你对这两个属性进行操作，都会同步，因为他们是同一个对象。只有当你对其中任意一个进行了一次覆盖，就会让他们两个的联系断开。比如：  
```js
module.exports.name = 'box'
exports.name === 'box' //true
module.exports === exports // true

exports = {name:'apple'}
module.exports.name !== 'apple' // true
module.exports === exports // false
```
非常的简单易懂，如果你希望加深影响，可以看看这一段代码。
```js
function __webpack_require__(moduleId) {
    // 缓存检测
    if(__webpack_module_cache__[moduleId]) {
        return __webpack_module_cache__[moduleId].exports;
    }
    // 创建一个标准的module模型
    var module = __webpack_module_cache__[moduleId] = {
        exports: {}
    };

    // 执行IIFE函数
    __webpack_modules__[moduleId](module, module.exports, __webpack_require__);

    // 返回结果
    return module.exports;
}
```
这是一段Webpack对Node模块进行打包前端化的代码中的一部分，你可能会注意到两行。`__webpack_modules__[moduleId](module, module.exports, __webpack_require__);`和`return module.exports;`。这下明白他们的关系了吗？真没那么多花里胡哨的😄。

## ES6 Module
浏览器使用的是一种新的，和Node有着非常大差别的模块加载方式。最常见的是两个关键词`import`，`export`。不过请注意，他们不表示任何对象，这里和Node的区别就非常大了。  
在Node中，如果你想要导入一个文件，那么你需要进行运行文件后，你才能获得这个文件的导出对象`module.exports`。而ES6设计思想是尽量的**静态化**，也就是，在编译的时候就确定这个模块的导出内容，而不是运行之后。   
同时要注意的一点是，ES6模块强制开启严格模式，这里不多叙述。  
同时，ES6中的内容是一个对值的引用，并不是一个拷贝。这里的原理将稍后讲述。
我们可以总结一些特征：
- 导出的内容可以被静态分析并且确定
- 导出的内容不是一个拷贝，而是一个引用。
- export default每个模块只能使用一次（你可以声明多个default导出，但是只会最后一个生效，而且将无法通过webpack的打包📦）
- 你不能导出任何运算后的结果

如果想看一个简单的ES6 Module的polyfill可以看下面的代码。
```js
// Source
export default function(){
    return 10
}
export var foo = 'bar'
export function test(){
    return 'test'
}
// polyfill
// 类似webpack打包后部分结果
let result = (function () {
    var foo = 'bar'
    function test() {
        return 'test'
    }
    return {
        "default": function () {
            return 10
        },
        "foo": () => foo,
        "test": test
    }
})()
```
export的导出非常严格，常见的可用的导出为
```js
// 来自MDN
// 导出单个特性
export let name1, name2, …, nameN; // also var, const
export let name1 = …, name2 = …, …, nameN; // also var, const
export function FunctionName(){...}
export class ClassName {...}

// 导出列表
export { name1, name2, …, nameN };

// 重命名导出
export { variable1 as name1, variable2 as name2, …, nameN };

// 解构导出并重命名
export const { name1, name2: bar } = o;

// 默认导出
export default expression;
export default function (…) { … } // also class, function*
export default function name1(…) { … } // also class, function*
export { name1 as default, … };

// 导出模块合集
export * from …; // does not set the default export
export * as name1 from …; // Draft ECMAScript® 2O21
export { name1, name2, …, nameN } from …;
export { import1 as name1, import2 as name2, …, nameN } from …;
export { default } from …;
```
## 区别总结 👨‍🏫
- `module.exports`和`exports`**默认**指向一个空对象，但是可以被替换修改。
- `export`为一个关键词，并非实际对象。
- 通过Node导出的内容为运算结果，可以认为是一份值拷贝。对于值类型来说，修改内容无法影响导出结果。// Note 1
- 通过ES6 Module导出的是对内容的一个引用，而不是直接进行拷贝，你可以通过上面的polyfill进行理解。

// Note 1
```js
// A.js
let foo = 'bar'
exports.name = foo
setTimeout(()=>{
    foo = 'foo'
    console.log(exports.name) // bar
},1)

// B.js
let foo = 'bar'
exports.name = foo
setTimetou(()=>{
    exports.name = 'foo'
    console.log(exports.name) // foo
})
```