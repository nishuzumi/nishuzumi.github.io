---
title: 2020-12-2-web-worker
date: 2020-12-02 14:33:10
tags:
---
# 前言 🎤
浏览器中真的并行任务

<!--more-->
# Web Worker
Web Worker是基于浏览器中真的多线程任务，这种任务和传统的任务相比，他最大的特点就是不会堵塞用户的UI渲染。并且通过一些方式来与主线程进行交互。

# API
每个worker在创建的时候都需要指定一个js文件，这个文件将会在一个新的环境中进行处理，并且依然可以访问到window对象，但是这个window对象和主线程的并不相同。

在专用worker下，`DedicatedWorkerGlobalScope`对象代表来worker的上下文（专用workers是指标准worker在仅一个脚本中被使用。共享worker的上下文是`SharedWorkerGlobalScope`）。一个专用worker只能被他首次生成的脚本使用，而共享worker可以被多个脚本一起使用。  
虽然在worker中你可以运行几乎任何东西，但是也存在例外，比如：worker中不能直接操作DOM，也不能使用window的默认方法和属性。但是一些不会影响主视角的其他内容可以被正常使用。比如WebSockets，IndexedDB。

workers和主线程通过传递消息进行通讯。双方都使用`postMessage()`发送各自消息。使用`onmessage`处理函数来响应。注意，传输的数据会被复制，而不是共享，因为本质上还是在进行进程间通讯。

如果worker运行在同源的父页面中，workers则可以一次生成新的workers，并且可以使用XHR进行网络IO，但是XHR的`responseXML`和`channel`总会返回null。
# 专用worker
## 安全检测
```
if (!window.Worker){
    // error !
}
```
## 生成专用worker
如果需要创建一个专用worker指需要调用Worker构造器，然后传入一个脚本的URI就可以了。
```js
const myWorker = new Worker('worker.js')
```
## 向worker中发送消息
创建好worker，你可以在程序之后的任何地方调用。并且需要通过`postMessage()`方式发送消息。通过`onmessage`的方式处理消息。例如：
```js
// main
myWorker.postMessage(['hole',123])
// worker
onmesssage = function(e){
    postMessage(`${e.data[1]}-${e.data[0]}`)
}
// main
myWorker.onmessage = function(e){
    alert(e.data)
}
```
注意，在worker中，worker自身为有效的全局域。
## 终止worker
如果你打算立刻停止一个worker，你可以在主线程中使用`terminate`方法。
```js
myWorker.terminate()
```
当然在worker线程中可以使用`close()`方法关闭
## 处理错误
当worker中的程序出现错误时，它的`onerror`事件会被调用，并且会收到一个拓展了`ErrorEvent`接口名为`error`的事件。
错误事件有三个主要的字段：
- message 良好的错误信息
- filename 发生错误的脚本文件名称
- lineno 发生错误时所在脚本的行号

## subworker
如果worker需要生成worker，那么就会生成所谓的subwoker，并且会托管在同源父页面中，而且在解析URI的时候，是相对与父worker的地址，而不是自身页面的地址。
## 引入脚本和库
有一个很神秘的函数`importScripts()`来引入脚本，该函数接受0或者多个参数来引入资源，以下内容都是合法的。
```js
importScripts()
importScripts('foo.js')
importScripts('foo.js','bar.js')
```
浏览器会加载并且运行每一个脚本，每个脚本中的全局对象都可以被worker使用，如果无法加载，会抛出`NETWORK_ERROR`异常。并且无法执行接下来的内容，但是之前通过`setTimeout`之类方法注册的异步方法仍然可以运行。  
⚠️ 脚本的下载顺序并不是固定的，但是执行顺序是按照传入参数顺序。并且所有脚本执行完成后，importScripts才会返回。这一点和defer有着异曲同工之妙。

# 共享worker
共享worker可以被多个脚本使用，即使他们在被不同的window，iframe或者worker访问。也就是说，他们可以跨页面进行通讯，但是必须同源。
## 创建
```js
const myWorker = new SharedWorker('worker.js')
```
有个非常大的不同在于，共享的worker需要通过端口对象进行通讯。例如，在父线程中，需要调用如下方法。
```js
myWorker.port.start()
//worker 如果worker中也需要往父线程发送信息，那么也需要进行start
port.start()
```
现在你就可以发送消息了，但是所有操纵都需要在port对象上进行。
```js
myWroker.port.postMessage([1,2])
```
而在worker中也有了些不同。
```js
onconnect = function(e){
    const port = e.ports[0]
    port.onmessage = function(e){
        port.postMessage('ack')
    }
}
```
而主脚本中也需要在port对象上设定onmessage函数。