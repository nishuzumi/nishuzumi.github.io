---
title: 2020-12-8-request-idle
date: 2020-12-08 09:32:48
tags:
---
# 前言 🎤
发现了一个很有趣的函数，`requestIdleCallback`。  
该函数可以让任务在浏览器空闲的时候被执行，并且可以设定超时时间。

# 用法
```
var handle = window.requestIdleCallback(callback[ options])
```
## 返回值
一个id，用于取消任务
## 参数
- callback 回调函数，这个函数的第一个参数会收到一个[`IdleDeadline`](#IdleDeadline)的参数。
- options 其中只包含了一个属性 timeout，如果这个timeout被指定而且有个正值，那么回调就会在超出的情况下，在下一次空闲时强制执行。

# IdleDeadline
这是一个用来判断执行中有关时间的对象，它自身含有一个属性和一个方法。
## didTimeout[Boolean] 只读
如果一个任务在设定的timeout之后才被运行，那么这个属性值会为`true`。
## timeRemaining():DOMHighResTimeStamp
返回一个[`DOMHighResTimeStamp`](#DOMHighResTimeStamp)，用于表示当前闲置周期的空闲时间，如果空闲时间已经结束，那么返回0。并且你需要不断的去调用这个方法来确定是否有更多的空闲时间。

# DOMHighResTimeStamp
它是一个双精度浮点数，可以精确到5ms