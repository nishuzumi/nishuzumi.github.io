---
title: 页面📃的生命周期小记
date: 2020-11-18 14:27:45
tags:
- 浏览器
catagories:
- [JS]
- [前端]
---
## 前言 🎤
整个页面加载是用户体验界面最重要的一部分，当你对浏览器对页面生命周期更为了解后，可以更好优化很多场景下的问题。

<!--more-->
## 三个事件 DOMContentLoaded load beforceunload/unload
一个页面由三个重要事件构成
- DOMContentLoaded 表示浏览器已经完全加载完HTML并且构建完成DOM树🌲，但是此时一些外部资源并不一定完成，比如（样式表，图像，iframe等等）。  
- load 表示浏览器已经彻底的加载完成了所有的内容
- unload 当用户离开页面时
- beforceunload 当用户离开页面前
同时，`DOMContentLoaded`为document上的事件，而其他的为`window`上。
### DOMContentLoaded
监听：
```js
document.addEventListener("DOMContentLoaded",()=>{})
```
要注意的是，文档中任何`<script>`都会在`DOMContentLoaded`触发之前执行完成。
#### Script
要注意的是，当浏览器在处理页面的时候，遇到了任何`<script>`，都会让脚本先执行完成。  
除了动态插入的脚本或者具有`async`的脚本。

#### Style
`DOMContentLoaded`一般不会被样式表堵塞，但是如果在`<link>`标签之后有任何的`<script>`标签出现，那么就会强制等待样式表加载完成才会触发`DOMContentLoaded`。  
**请注意，是任何的`<script>`标签**

#### 自动填充
浏览器会尝试自动填充表单，而且是在触发`DOMContentLoaded`的时候，如果说页面被某个奇怪的东西堵塞了，那么自动填充也不会被触发。

### onload
当整个页面上一切的东西动被加载完成的时候，便会执行`window.onload`回调。在这个阶段，任何内容（除了动态插入的）都已经准备就绪。

### onbeforceunload/onunload
beforceunload和unload比较特殊，在这两个事件中，任何`alert,confirm`都会被浏览器阻止。但是如果在`beforeunload`的回调中`return false`，浏览器则会提醒用户是否要离开此页面，当然`unload`中是无法做到此类操作的，因为在unload中，已成定局。

## 时机不对
如果你在页面加载完成之后才去监听`DOMContentLoaded`，那你永远也不会得到任何回应。不过浏览器中可以通过读取状态标识来判断是否加载完成。比如可以通过`document.readyState`进行判断。它的值有三种情况：   
- loading 页面正在加载
- interactive 页面已经加载了，但是外部资源的状况还不明朗
- complete 这个页面上的任何东西都已经被加载了

## async defer
既然提到了，那么也就顺势的讲一下。
`async和defer`都是让script脚本进行并行加载的关键词,他们的区别其实在于执行时间点。同时，如果两个同时存在，那么会优先`async`。  
关于defer
- 即使加载完成，也会按照出现顺序执行。
- 只会在页面解析完成后执行，但是在`DOMContentLoaded`之前执行。
关于async
- 加载完成立刻执行，没有先后顺序

但是，如果`<script>`标签中有脚本内容，这两个属性都无效。
{% img /images/page-life-2020-11-18-19-32-37.png %} 

## 总结 👨‍🏫
页面在加载时有三层状态
1. 完全什么都不知道的情况
2. DOM元素已经加载完成了，但是外部资源的情况不太清楚
3. 全部的东西都加载完成了，包括外部资源

而`DOMContentLoaded`处于（2）的状态。`onLoad`处于（3）的状态。但是两者都可以获取到DOM元素。

