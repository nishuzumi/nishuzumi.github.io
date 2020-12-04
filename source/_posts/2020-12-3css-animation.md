---
title: CSS动画
date: 2020-12-03 13:56:09
tags:
- CSS
- 浏览器
---
# 前言 🎤
CSS动画是CSS中一个重要特性，它不但让浏览器中有着更好的样式效果，也丰富了画面的观赏性。
CSS中我理解为有两种动画，一种是插值动画，一种是插帧动画。
<!--more-->
# 动画（插帧动画）
所谓的插帧动画其实就是序列动画，在CSS中如果你需要创建动画序列，你需要使用`animation`和其子属性，该属性允许你配置动画时间，时长和其他动画细节，但是这些属性不能配置动画的实际表现，实际表现是由`@keyframes`来决定。

`animation`的子属性有：
- animation-delay 设置延迟，指元素加载完成到动画运行的这段时间
- animation-direction 用于设置动画在运行结束后是反向运行还是直接重置
- animation-duration 设置一个动画的周期时长
- animation-iteration-count 设置动画的重复次数，可以指定infinite无限次重复
- animation-name 指定由`@keyframe`描述的关键帧名称
- animation-play-state 是否允许暂停和恢复
- animation-timing-function 设置动画速度，也就是设置加速曲线
- animation-fill-mode 指定动画前后如何设定样式

## 使用keyframes定义动画序列
动画序列中，可以使用`from`,`%0`来表示动画的开始。用`to`,`%100`表示动画结束的时机。

## 动画事件
在CSS动画的运行过程中，可以在JS中监听到他们事件的触发。
```js
const e = document.getElementById('#button')
e.addEventListener("animationstart",()=>{},false)
e.addEventListener("animationend",()=>{},false)
e.addEventListener("animationiteration",()=>{},false)

e.className = 'i am keyframe name'
```
在代码最后才设置class的原因很简单，因为添加监听器的时候，动画可能已经开始了。

# 插值动画（transitions）
其实这个动画的感觉很想Flash的补间动画，浏览器会通过计算开始和结束时候的状态来给中间内容进行插值，从而形成动画。  
transactions属性用来决定什么属性会发生动画效果，并且何时开始，持续多久，以及动画的变速曲线。  
不过要注意的是，可以进行动画的CSS属性是一个有限集合，但是大部分的CSS属性都能支持动画，不过仍然要注意的是，一些值为`auto`的动画可能不和预期一样，请小心使用。

## 事件
在transitions完成的时候会触发一个事件，在标准浏览器下，这个事件是`transitionend`，在WebKit下是`webkitTransitionEnd`。`transitionend`提供两个属性：
- propertyName 表示以及完成过渡的属性
- elapsedTime 表示已经运行了的时间

## 当属性缺失时
如果遇到下面场景
```css
div{
    transition-property: opacity,left,top,height;
    transition-duration: 3s,5s;
}
```
会自动变成
```css
div{
    transition-property: opcity,left,top,height;
    transition-duration: 3s,5s,3s,5s;
}
```
但是如果是属性缺失，那么`duration`会被截断。

## ALL
当你想一劳永逸的时候，可以使用`transition:all <duration>`。