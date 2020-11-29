---
title: 小记addEventListener
date: 2020-11-28 19:41:50
tags:
---
# 前言 🎤
addEventListener，没那么简单。

# 常用情况
一般使用addEventListener的情况基本只会使用到前两个参数
```js
target.addEventListener('click',function(){
    console.log('click')
})
```
可能某些性能要求高的也会往第三个参数传个true，让事件在捕获周期进行。
```js
target.addEventListener('click',console.log,true)
```
# 第三个参数
第三个参数有两种可以传递值`Boolean|Object`。默认情况下为`false`。  
当传入`bool`的时候，第三个参数为`useCaptrue`，表示是否注册到事件捕获周期。  
如果传入`Object`，那么则会有一系列的配置。
```js
{
    capture:false,
    once:false
    passive:false,
}
```
- capture 是否注册到捕获周期
- once 是否只允许单次调用
- passive 是否阻止调用`preventDefault`
# option支持的安全检测
在旧版的DOM规定中，第三参数必定是一个不二值，而在新版本中，第三个参数具有更多的功能，但是为了安全的检测是否支持，需要用以下代码。
```js
let passiveSupported = false
try{
    let options = Object.defineProperty({},'passive',{
        get:function(){
            passiveSupported = true
        }
    })
    window.addEventListener('test',null,options)
}catch(e){}
```
# this
一般情况下，执行时`this`指向当前执行元素。

# IE
在IE下需要使用`attachEvent`

# passive
一般情况下，`Window`，`Document`，`Document.body`，`touchstart`，`touchemove`的`passive`均为默认`true`。这是防止某些监听器中阻止了默认事件导致页面效果异常。

# 对立方法 removeEventListener
用于移除在某个元素上监听对事件，一般情况下和`addEventListener`对参数保持一致即可。