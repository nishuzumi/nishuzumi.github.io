---
title: '♻️Eventloop,⏱️Timeout,⚡️Immediate,⭕️Interval,🧾QueueMicrotask'
date: 2020-11-10 14:08:58
tags:
- [JS]
- [前端]
- [All In Code]
---
## 前言 🎤

本文的目的在于，一次性推翻80%人构建好的关于Event Loop的知识体系和一次性的完整的理解Nodejs(13以上)和浏览器中的Event Loop。

## 划分
首先进行一下基础概念的划分。
Node下特有的
- Immediate
- process.nextTick

浏览器中特有的:🈚️

双方共有的：
- queueMicroTask (nodejs > v11)
- setTimeout
- setInterval

### 宏任务
这里补充一下宏任务
|#|Node|Browser|
|-|----|-------|
|setTimeout|✅|✅|
|setInterval|✅|✅|
|I/O|✅|✅|
|setImmediate|✅|❌|
|mouseover(之类的事件)|❌|✅|
|Web API大部分异步返回方法(XHR,fetch)|❌|✅|

> 逗知识：其实setTimeout和setInterval也属于Web API

### 宏任务：注意⚠️⚠️⚠️⚠️⚠️(浏览器中)
`requestAnimationFrame`和`render`不为宏任务，也不为微任务。他们两个的触发时机在宏任务和微任务之外。
`requestAnimationFrame`的触发时机在`render`之前。而`render`的触发时机在**所有微任务处理结束**之后。
但是他们的触发顺序为`microTask->requestAnimationFrame->render`
```js
requestAnimationFrame(()=>{console.log('animation')})
queueMicrotask(()=>{console.log('micro')})

/*
micro
animation
*/
```
### 微任务
|#|Node|Browser|
|-|----|-------|
|Promise.then catch finally|✅|✅|
|MutationObserver|❌|✅|
|queueMicrotask|✅|✅|

### 微任务：注意⚠️⚠️⚠️⚠️⚠️
**`process.nextTick`不是微任务**
```js
queueMicrotask(()=>{console.log('micro')})
process.nextTick(()=>{console.log('i am not micro')})

/*
i am not micro
micro
*/
```
如果以上两个注意，都让你原有的知识大厦倾倒，那么请继续往下看，看完后你将会得到一切的答案。

## Nodejs
先看官方定义
{% img /images/Eventloop-Timeout-Immediate-Interval-QueueMicrotask-2020-11-11-14-11-56.png %}

>阶段概述
>定时器：本阶段执行已经被 setTimeout() 和 setInterval() 的调度回调函数。
>待定回调：执行延迟到下一个循环迭代的 I/O 回调。
>idle, prepare：仅系统内部使用。
>轮询：检索新的 I/O 事件;执行与 I/O 相关的回调（几乎所有情况下，除了关闭的回调函数，那些由计时器和 setImmediate() 调度的之外），其余情况 node 将在适当的时候在此阻塞。
>检测：setImmediate() 回调函数在这里执行。
>关闭的回调函数：一些关闭的回调函数，如：socket.on('close', ...)。
>在每次运行的事件循环之间，Node.js 检查它是否在等待任何异步 I/O 或计时器，如果没有的话，则完全关闭。

简单来说就是我都不知道他在说什么，😄。
所以我特意找了一张比较容易看懂的图，来简单的概括一下这些学术语言。
{% img /images/Eventloop-Timeout-Immediate-Interval-QueueMicrotask-2020-11-11-13-00-10.png %}
图片来自互联网

细心的同学已经看出来了，在图片的中间的红色区域写着`process.nextTick`和`microTasks`。
### 第一个重点
**在Node中，nextTick和microTask会在每一个阶段执行结束后被立刻执行。**。
> 他们不属于任何阶段，但是他们又属于任何阶段 ——Box

简单的来说，在Node中，每一个处理阶段结束，都会执行并清空`nextTick`和`microTask`的任务。
而Node中，分为四个阶段
- setTimeout和setInterval的到期回调执行阶段
- IO轮询阶段
- 处理setImmediate的回调阶段
- 处理某些关闭句柄的回掉阶段
反复执行。
所以说，在这个四个阶段执行间隔中，只要出现了任何`nextTick`或者`microTask`都会被执行知道清空

### 第二个重点
先说重点然后看代码：**在处理nextTick和microTask时，会一直循环直到两个任务队列清空，如果你在他们的逻辑中循环创建了新的任务，那么将无法离开这个阶段**
```js
console.log('timeout');
if(true){ // flag 1
    process.nextTick(()=>{console.log('next timeout')})
    let loop = function(){
        process.nextTick(loop)
    }
    process.nextTick(loop)
}

if(true){ // flag 2
    queueMicrotask(()=>{
        console.log('micro')
    })
    
    let micro = ()=>{queueMicrotask(micro)}
    queueMicrotask(micro)
}

setImmediate(()=>{
    console.log('immediate')
})
```
可以尝试把flag1或者flag2处的true改成false，只要有任何一个循环存在，那么都无法离开这个**尾部处理阶段**——我自己编的名词。
### 第三个重点
- **每个尾部处理阶段一定会执行，就算当前阶段什么都没做，但是依然会执行**
- **尾部处理阶段顺序是nextTick->queueMicroTask 也就是 tickTask->microTask，并且这个顺序是固定的，不可以翻转，也不可以回退**
```js
console.log('timeout');

queueMicrotask(()=>{
    console.log('micro')
    process.nextTick(()=>{ // 在microTask之前执行的tickTask一旦执行完成，那么在同一个尾部阶段是不会继续被执行的。
        console.log('next')
    })
})

setImmediate(()=>{
    console.log('immediate')
})
/**
timeout
micro
next
immediate
*/
```

### 第四个重点
**事件循环有一种维持在poll阶段的倾向**
在poll阶段时，会进行一些判定处理，比如会进行计算，大概会在这个阶段停留多久——一般根据下一个要被执行的`setTimeout`或者`setInterval`来决定。
然后会在可停留时间中进行等待。直到需要执行`timer`的时候才会退出。
举个例子，假设进入poll时，又一个100ms以后的才会被执行的setTimeout，那么，就会尝试在这100ms之中，尽可能的等待IO事件触发，并处理。每当处理完后依然会判断，是否需要退出。
所以在大多数情况下，Nodejs都会在poll阶段运行。并且，会尝试清空所有返回的IO事件的回调（这个不是无限制的，有一个硬性限制）。

与此同时，这里有出现来个新问题。
> 如果回调队列中有大量的回调完成事件并且他们会堵塞100ms以上，那么setTimeout(fn,10)会怎么样
这里就是一个很有趣的地方了。
```js
const fs = require('fs')
console.log('start');
let globalStart = Date.now()

function call(callback) {
    fs.readFile(__filename, callback);
}

for(let i =0;i<20;i++){
    call(()=>{
        let start = Date.now()
        while(Date.now() - start < 100){}
        console.log(`done:${i}`)
    })
}

setTimeout(()=>{
    console.log(`${Date.now()-globalStart}ms`)
    console.log('timeout')
},10)
```
这段代码中模拟了每次IO调用超过100ms，并且触发了20次，同时也创建了一个timeout为10ms的任务。感兴趣的可以猜一猜他的运行结果？
运行结果
{% img /images/Eventloop-Timeout-Immediate-Interval-QueueMicrotask-2020-11-11-15-18-42.png %}

很明显，在有些情况下，还没等poll中回调全部执行完成，就直接开始进入了下一个循环，而有些情况，又是执行完了全部的回调才进入下一个阶段。 
不过这里不是我们讨论的重点，因为这个想说的是`pending callbacks`这个阶段。我们可以看到有些`done:x`是在已经打印了timeout之后才出现。那么也就可以说明，已经从poll阶段循环到了timer阶段，最后在`pending callbacks`触发回调。所以这个就是`pending callbacks`阶段的作用。
有些人可能保持怀疑的态度，认为并没有进入`pending callbacks`，可能之前的`read`事件并没有返回，所以重新进入了poll进行等待。我只能告诉你，我确实也无法证明他们究竟是在`pending callbacks`还是`poll`阶段被执行，如果有更好的方法，还请多多赐教。

### 第五个重点
**setImmediate也没有那么的immediate**
所有人都知道`setImmediate`的执行一般会比`setTimeout`优先，但是其实他们也存在相反的情况。有时候是`immediate`快，有时候是`timeout`快，这种情况在直接写出
```js
setTimeout(()=>{console.log('timeout')},0)
setImmediate(()=>{console.log('immediate')})
/*
有时候
timeout
immediate

有时候
immediate
timeout
*/
```
这种毫无意义的代码下会更容易出现。所以为什么会有这种情况？
{% img /images/Eventloop-Timeout-Immediate-Interval-QueueMicrotask-2020-11-11-15-59-29.png %}
[图片来源](https://medium.com/softup-technologies/node-js-internals-event-loop-in-action-59cde8ff7e8d)
简单的来说，在进入事件循环之前，你只需要进行js代码执行。那么这里也会产生耗时，所以，这里的运算时间就充满了不确定性。也许在运行`setTimeout(fn,1)`的那一瞬间，当前的ms可能为10，但是当node执行完所有代码之后，导入了一大堆杂七杂八的库，此时的ms可能已经成为了14，很明显，`setTimeout`被优先执行。那么如果说，当前的ms可能还是10，那么就会跳过`timer`阶段，所以看起来就像是`setImmediate`优先执行。那么有人可能会问，为啥要`setTimeout(fn,1)`，而不是`setTimeout(fn,0)`？这就是下一个重点。

### 第六个重点
**在Nodejs下，`setTimeout`的最小延迟数是1，最大数为2147483647，如果小于或者大于最大，那么都会被修改为1**
这里其实没啥好说的，就是这么设计的。不过有趣的是，最小为1其实是向浏览器看齐而这么做的。（源码的注释中是这么写的😄）

### 第七个重点
**Promise.then中使用的微任务，和microTask为同一等级**（存疑）
为什么有个存疑🤨，因为从表现上来看，没有错误，但是我并没有去阅读详细的代码来证明它的确定性。
运行代码如下
```js
queueMicrotask(()=>{console.log('micro')})
Promise.resolve(1).then((v)=>{
    console.log(v)
})
queueMicrotask(()=>{console.log('micro2')})
/**
micro
1
micro2
*/
```
但是，根据这个结论来看，`Promise.then`就是一个微任务。并没有高人一等。
## Browser
{% img /images/Eventloop-Timeout-Immediate-Interval-QueueMicrotask-2020-11-11-16-30-53.png %}
是不是看起来和Node的Event loop有很大的不同？如果我告诉你可以这么理解，会不会更好？
> Browser中，去除了`pending callbacks`,`poll`,`check`,`close callbacks`,并全部塞入`timer`，微任务处理时机不变
但是有一点小问题，就是事件循环的结束时机。但是等会会讲。

而浏览器中的执行顺序是这样。`one macroTask->all microTask->requestAnimationFrame->render`。
而且，和Node相似的是，他们的`setTimeout`或者`setInterval`都是尽可能的执行（时间无法保证完全符合预期）。
还有一点，那就是`Web API`，在浏览器中，所有涉及到`Web API`的内容都交给C++引擎进行处理。也就是说，当你的`setTimeout`到底预计时间了，就算你的js引擎被某个`while(true)`给堵住了，这个`setTimeout`的内容依然会被推入到一个`TaskQueue`之中。等待下一次`Event Loop`的取出。
### 重点一
**在浏览器中，微任务的执行时机是在调用栈清空时**
**浏览器中一次性只执行一个宏任务**

浏览器中的宏任务处理方式和Node稍微有一点点不同，Node会在一个阶段把所有到期任务都执行完成。而浏览器中，是会在执行完成一个宏观任务时清空微任务队列。而且微任务队列不会收到网络相应或者其他类型的回掉任务干扰。也就是说，和Node一样，如果你在微任务中无限创建新的微任务，那么你基本可以和这个页面说再见了👋。
如果不信，可以试试这个。
```js
let call = ()=>{queueMicrotask(call)}
queueMicrotask(call)
```

### 重点二
**`setTimeout`的最小时间为1ms，而当嵌套等级达到一定时（跟随浏览器，3-5层），为4ms**
```js
let startTime = Date.now()
let call = ()=>{setTimeout(call);console.log(`${Date.now() - startTime}`)}
call()
```
{% img /images/Eventloop-Timeout-Immediate-Interval-QueueMicrotask-2020-11-11-19-11-53.png %}
不要问1到哪去了，如果实在不懂可以重新看文章，还是不懂可以悄悄问我，咱们别丢人。

### 重点三

## 总结
- process.nextTick不是微任务
- requestAnimationFrame也不是微任务
- Promise.then中的创建方式就是创建微任务，并且和queueMicrotask同级
- setTimeout(fn,0)并不是0，最小为1
- 如果你还不懂，可以直接问我，如果你有更好的想法，那么我们一起构建更好的文章，如果你不懂而且也不问我，那么我直接切腹自尽😭。
- 有些内容我可能没有写出来，或者你有任何疑问，我都会补充在文章之中。
