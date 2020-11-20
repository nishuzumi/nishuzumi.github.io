---
title: 细看事件委托
date: 2020-11-18 20:39:00
tags:
catagories:
- [JS]
- [浏览器]
- [前端]
---
## 前言 🎤
>捕获和冒泡允许我们实现一种被称为 事件委托 的强大的事件处理模式。
>这个想法是，如果我们有许多以类似方式处理的元素，那么就不必为每个元素分配一个处理程序 —— 而是将单个处理程序放在它们的共同祖先上。
<!--more-->
## 事件的传递

当一个事件发生在具有父元素当元素上时，有两种阶段：
**捕获阶段**
捕获阶段会从最外层的父元素开始查找，是否注册了相关的事件，比如`onclick`,如果是，则允许。
然后他会找到下一层的父元素，继续执行这个流程，直到到达实际点击的元素。
**冒泡阶段**
冒泡阶段恰好相反，他会先处理自身的`onclick`，然后找到最近的父元素，并且继续执行，然后再是上上层的父元素，直到到达`<html>`为止。  

>豆知识：这两种阶段是微软和网景提出的两种阶段，最后w3c指定了统一标准。并且定义他们的执行顺序为先捕获再冒泡。

同时要注意几点：
- on*系列的执行程序，比如`onclick`均在冒泡上触发。
- addEventListener(type,callback,capture = false) 第三个参数为false（默认）时，注册为冒泡，否则为捕获
- ~~因为微软是老大，所以大家一般默认使用冒泡~~

同时，在DOM事件中，传播分为三个阶段
- 捕获阶段 
- 目标（到达）阶段 （不常使用）
- 冒泡阶段

{% img /images/event-delegation-2020-11-19-12-51-31.png %}   

一般来说，我们默认情况下都是在使用冒泡阶段，但是并不是说捕获阶段就没有任何作用。我们可以在捕获阶段中使用`event.stopPropagation()`来中止捕获或者冒泡阶段的传递。

## 事件的处理
因此，当出现冒泡事件时，最先被触发的，自然是元素本身上面的监听程序。当事件触发时，会向元素上的监听器传递一个`event`对象。`event`中常用的属性为：
- event.target 触发元素
- event.currentTarget 当前执行程序挂载的元素
- event.eventPhase 当前阶段（capturing=1，target=2，bubbling=3）
**并且，程序中的`this`，默认情况下指向`event.currentTarget`。**
比如，你在`body`上设定了`onclick`，当点击`body`中的`button`后，如果冒泡达到`body.onclick`时，event中属性的内容应该分别为
- event.target == button
- event.currentTarget == body
- event.eventPhase == 3

不过，如果在一个元素上注册了多个监听器，那么他们的执行顺序会按照注册顺序来执行。如果你希望停止执行同一个元素其他的监听器并且停止冒泡，那么可以用`event.stopImmediatePropagation()`来强制停止。

### ⚠️注意
不要认为可以优化很多性能而自作主张的在每个监听器上面都阻止冒泡，除非整个程序完全由你掌控。很多函数库或者UI框架或多或少的使用来事件委托功能，如果没有十足的把握，请不要轻易的阻止冒泡或者事件传递。

## 事件委托
事件委托其实就是依赖于事件传递的一种程序设计方式。在处理程序中，用户点击到的元素不一定是真正想要的元素。比如：
```js
// html
<div class='main'>
    <div class='button'>
        <span>😊</span><span>点我！</span>
    </div>
</div>
// js
document.querySelector('.main').addEventListener('click',function(ev){
  alert(this.tagName+ev.target.tagName) // DIVSPAN
})
```
很明显，我们想判断的是它有没有点击按钮，而不是SPAN。  
因此我们可以在代码中进行一些判断，比如:
```js
object.onclick = function(ev){
    let target = ev.target.closest('.button') //判断自身或者上级中是否有可以被选择器选择的元素
    if(!target) return
    if(!object.containts(target)) return; // 如果不是当前元素的后代元素

    // do something
}
```
## 总结
当一个事件触发时，会将💉事件穿透整个🧅层级结构：
- 穿入的情况叫做捕获阶段
- 穿到目标的时候叫做到达阶段
- 过度穿透的时候叫做冒泡阶段。
当你使用`event.stopPropagation()`，会停止继续穿透。  

在使用`addEventListener`时第三个参数用于设定注册在捕获还是冒泡阶段。  
传递到程序中的`event`对象中：
- event.target 表示目标元素
- event.currencyTarget(==this) 表示当前处理元素
- event.eventPhase 表示当前阶段

不要随便的阻止事件传播。