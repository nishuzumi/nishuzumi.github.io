---
title: "Promise in JS \U0001F9EE"
date: 2020-11-04 23:02:17
tags:
---
## 前言
下面的内容为Promise的JS全实现，但是效率并不如原生Promise，因为原生Promise使用了微任务，而为了模拟同样的效果，只能使用宏任务setTimeout来替代。

为了防止在原生的Promise对象上进行修改，导致某些错误的功能误打误撞正确了，所以使用一个新的名称，取名为Bromise。
同时一下内容可以通过promise aplus test。并且能通过MDN上各种用例。

## 我的理解(我的猜测)
Promise在js中的本质其实就是一个具有自动处理多个回调的高阶语法糖。嗯，其实就是可以解决因为callback地狱导致的多层嵌套问题，实质上其实是把递归转化为了迭代来进行逻辑处理。

在Promise中，传入构造函数中的函数将会被**立刻执行**，而其他的内容，则会被安排微任务处理阶段进行处理。同时微任务的执行时机为宏任务结束后，所以，他会比任何在本次循环中的setTimeout或者其他类似的宏任务**都优先执行**。
所以这还涉及到一个小问题，如果当前代码循环中，不交出控制权会怎么样？
例如
```js
let bool = false
setTimeout(()=>{
    console.log(bool)
})
while(1){
    bool = !bool
}
```
其实你执行一次就会发现，你已经永远的失去了他的控制权，而且`setTimeout`内容将会永远的不能触发。这是因为js本质其实是在不断的循环解决任务。你也可以认为是这样。
```js
while(1){
    let currencyMacro = World.macros
    World.macros = []
    macroTakss(currencyMacro)
    let currencyMicro = World.micros
    World.currencyMicro = []
    microTasks(currencyMicro)
}
```
而如果我们直接创建一个文件，并且运行，那么这些代码一定是运行在`macroTasks`中。而如果调用了`setTimeout`，则将会在下次的`macroTasks`运行。
所以说，当你在`while`中触发了**无限循环**。导致js无法进入下一个循环流程。那么你将永远的停留在那不可到达的彼岸。

因此，可以很明显的了解到`promise`的部分执行流程。同时你也可以写出20%有关输出先后的问题。那么还剩下80%。
举个例子(出自[掘金](https://juejin.im/post/6844903987183894535#heading-2))
```js
new Promise((resolve, reject) => {
    console.log("外部promise");
    resolve();
})
    .then(() => {
        console.log("外部第一个then");
        new Promise((resolve, reject) => {
            console.log("内部promise");
            resolve();
        })
            .then(() => {
                console.log("内部第一个then");
            })
            .then(() => {
                console.log("内部第二个then");
            });
    })
    .then(() => {
        console.log("外部第二个then");
    });
    /** output:
        外部promise
        外部第一个then
        内部promise
        内部第一个then
        内部第二个then
        外部第二个then
    */
```

```js
new Promise((resolve, reject) => {
    console.log("外部promise");
    resolve();
})
    .then(() => {
        console.log("外部第一个then");
        new Promise((resolve, reject) => {
            console.log("内部promise");
            resolve();
        })
            .then(() => {
                console.log("内部第一个then");
            })
            .then(() => {
                console.log("内部第二个then");
            });
    })
    .then(() => {
        console.log("外部第二个then");
    });
    /** output:
        外部promise
        外部第一个then
        内部promise
        内部第一个then
        外部第二个then
        内部第二个then
    */
```
如果你不知道为什么少了一个`return`就导致后面两行的输出顺序发生了改变，那么请君与我一同研究一下`promise`的写法。

## 实现Bromise
```js
const PENDING = "pending"
const Fulfilled = "fulfilled"
const Reject = "reject"

function Bromise(fn) {
    let self = this
    this.status = PENDING
    this.onResolve = []
    this.onReject = []

    function onResolve(value) {
        if (self.status != PENDING) {
            return
        }

        self.status = Fulfilled
        self.value = value
        self.onResolve.forEach(cb => cb());
    }

    function onReject(reason) {
        if (self.status != PENDING) {
            return
        }
        self.reason = reason
        self.status = Reject
        self.onReject.forEach(cb => cb());
    }

    try {
        fn(onResolve, onReject)
    } catch (e) {
        onReject(e)
    }
}
```
上面的代码非常简单，可以说是promise最简单的部分。其中的fn其实是我们在创建Promise时候传入的执行函数。比如：
```js
new Promise((resolve,reject)=>{
    // doSomething
    resolve()
})
```
显而易见的，我们每当完成任务的时候，都会调用传入的参数`resolve`，而这个`resolve`则是构造函数中的`onResolve`方法。这个方法会依次执行挂在`this.onResolve,this.onReject`上面的函数。而这个两个数组，模拟的是微任务中的promise部分。
接着是第二个重要的部分`resolvePromise`，这个`resolvePromise`和`Promise.resolve`并不是一个函数，请注意。
```js
function resolveBromise(promise, x, resolve, reject) {
    if (promise === x) {
        reject(new TypeError("Chaining cycle"))
        return
    }

    if (!(x && typeof x === 'object' || typeof x === 'function')) {
        resolve(x)
    }

    let used;
    try {
        let then = x.then
        if (typeof then === 'function') {
            then.call(x, (y) => {
                if (used) return
                used = true
                resolveBromise(promise, y, resolve, reject)
            }, (r) => {
                if (used) return
                used = true
                reject(r)
            })
        }else{
            if (used) return
            used = true
            resolve(x)
        }

        
    } catch (e) {
        if (used) return
        used = true
        reject(e)
    }
}
```
其实和`Promise.resolve`的实际效果差不多，但是他保证了第一个参数promise和第二个参数x不是同一个对象，因此不会出现无限`resolvePromise`。
上面这段代码的主要作用是把不是thenable的值resolve返回，把是thenable的运行后继续这个判断，直到返回值。

接下来实现最重要的`then`,整个Promise其实就这三个部分，至于其他的catch，finally，都是一些小封装。
```js
Bromise.prototype.then = function (onResolve, onReject) {
    onResolve = typeof onResolve === 'function' ? onResolve : value => value
    onReject = typeof onReject === 'function' ? onReject : reason => { throw reason }

    let self = this
    let result = new Bromise((resolve, reject) => {
        if (self.status === PENDING) {
            self.onResolve.push(() => {
                setTimeout(() => {
                    try {
                        resolveBromise(result, onResolve(self.value), resolve, reject)
                    } catch (e) {
                        reject(e)
                    }
                })
            })

            self.onReject.push(() => {
                setTimeout(() => {
                    try {
                        resolveBromise(result, onReject(self.reason), resolve, reject)
                    } catch (e) {
                        reject(e)
                    }
                })
            })
        }else{
            setTimeout(() => {
                try {
                    let callTarget = self.status === Fulfilled?onResolve:onReject
                    resolveBromise(result, callTarget(self.status === Fulfilled?self.value:self.reason), resolve, reject)
                } catch (e) {
                    reject(e)
                }
            });
        }
    })
    return result
}
```
注意这个方法，如果想要理解，那么要注意几点。
 1.Promise会立刻执行构造时传入的函数，因此，then中构造的Promise会立刻执行，但是内容均为创建微任务，也就是说，**then的内容，被立刻的分到了下一个微任务循环中**
 2.每当你调用`then`，你都会获得一个新的Promise，因此，你接下来如果继续调用then，**只会和上个Promise关联**
 3.虽然setTimeout有先来后到的顺序机制，但是注意，在一个Promise结束的时候，他会立刻执行他两个任务序列，也就是`this.onResolve,this.onReject`，但是，还请注意，**这两个序列的具体内容，将会在下一个微任务中运行**
 4.当你在一个then中返回了一个Promise，这么那个then自身的Promise状态，将会由返回值来决定。

请注意，这个4将会是很多很多问题的关键。那么为什么return的Promise会接管原本的Promise的状态？
答案就在`then`的代码中。
```js
try {
    let callTarget = self.status === Fulfilled?onResolve:onReject
    resolveBromise(result, callTarget(self.status === Fulfilled?self.value:self.reason), resolve, reject)
} catch (e) {
    reject(e)
}
// 如果你把它简化一下。假设默认的只会出现Fulfilled状态。那么这段代码应该为
try {
    resolveBromise(result, onResolve(self.value), resolve, reject)
} catch (e) {
    reject(e)
}
//而上文提到过，resolvePromise的作用是 “把不是thenable的值resolve返回，把是thenable的运行后继续这个判断，直到返回值。”
//因此，作为return返回的Promise，此时将会被注入到self.value中，并且传递到resolvePromise当中
```
所以，我们可以变相的认为，因为`return <Thenable>`的逻辑，导致`Thenalbe`的逻辑被强制的提升。也就是说，任务执行提前了。
通俗易懂的说，外层`then`的状态，将会由**return的Promise的状态**来决定。~~称之为紧急提升~~

{% 初始状态的Promise /images/Promise-in-JS-🧮-Promise.png %}

所以说，你可以简单的认为有这样几个队列。
|Next|OnResolve|OnReject|
|----|---------|--------|
|将会在下次微任务循环执行时执行|将会在自身Promise执行resolve完成后下一个微任务循环执行|将会在自身Promise执行reject完成后下一个微任务循环执行|



## 完整代码
```js
const PENDING = "pending"
const Fulfilled = "fulfilled"
const Reject = "reject"

function Bromise(fn) {
    let self = this
    this.status = PENDING
    this.onResolve = []
    this.onReject = []

    function onResolve(value) {
        if (self.status != PENDING) {
            return
        }

        self.status = Fulfilled
        self.value = value
        self.onResolve.forEach(cb => cb());
    }

    function onReject(reason) {
        if (self.status != PENDING) {
            return
        }
        self.reason = reason
        self.status = Reject
        self.onReject.forEach(cb => cb());
    }

    try {
        fn(onResolve, onReject)
    } catch (e) {
        onReject(e)
    }
}

Bromise.prototype.then = function (onResolve, onReject) {
    onResolve = typeof onResolve === 'function' ? onResolve : value => value
    onReject = typeof onReject === 'function' ? onReject : reason => { throw reason }

    let self = this
    let result = new Bromise((resolve, reject) => {
        if (self.status === PENDING) {
            self.onResolve.push(() => {
                setTimeout(() => {
                    try {
                        resolveBromise(result, onResolve(self.value), resolve, reject)
                    } catch (e) {
                        reject(e)
                    }
                })
            })

            self.onReject.push(() => {
                setTimeout(() => {
                    try {
                        resolveBromise(result, onReject(self.reason), resolve, reject)
                    } catch (e) {
                        reject(e)
                    }
                })
            })
        }else{
            setTimeout(() => {
                try {
                    let callTarget = self.status === Fulfilled?onResolve:onReject
                    resolveBromise(result, callTarget(self.status === Fulfilled?self.value:self.reason), resolve, reject)
                } catch (e) {
                    reject(e)
                }
            });
        }
    })
    return result
}

function resolveBromise(promise, x, resolve, reject) {
    if (promise === x) {
        reject(new TypeError("Chaining cycle"))
        return
    }

    if (!(x && typeof x === 'object' || typeof x === 'function')) {
        resolve(x)
    }

    let used;
    try {
        let then = x.then
        if (typeof then === 'function') {
            then.call(x, (y) => {
                if (used) return
                used = true
                resolveBromise(promise, y, resolve, reject)
            }, (r) => {
                if (used) return
                used = true
                reject(r)
            })
        }else{
            if (used) return
            used = true
            resolve(x)
        }

        
    } catch (e) {
        if (used) return
        used = true
        reject(e)
    }
}

Bromise.resolve = function(value){
    if(value instanceof Bromise){
        return value
    }

    return new Bromise((s,j)=>{
        if(value&&value.then&&typeof value.then === 'function'){
            setTimeout(()=>{
                value.then(s,j)
            })
        }else{
            s(value)
        }
    })
}

Bromise.reject = function(value){
    return new Bromise((x,y)=>{
        y(value)
    })
}

Bromise.prototype.catch = function(onReject){
    return this.then(null,onReject)
}

Bromise.prototype.finally = function(cb){
    return this.then(value=>{
        return Bromise.resolve(cb()).then(()=>{
            return value
        })
    },reason=>{
        return Bromise.resolve(cb()).then(()=>{
            throw reason
        })
    })
}

Bromise.all = function(promises){
    return new Bromise((s,j)=>{
        let index = 0
        let result = [];
        if(promises.length == 0){
            return s(result)
        }

        let deal = function (i,data){
            result[i] = data
            if(++index == promises.length){
                return s(result)
            }
        }

        for(let i = 0;i<promises.length;i++){
            Bromise.resolve(promises[i]).then((value)=>{
                deal(i,value)
            },err=>{
                j(err)
            })
        }
    })
}

Bromise.race = function(promises){
    return new Bromise((resolve,reject)=>{
        for(let one of promises){
            Bromise.resolve(one).then(value=>{
                resolve(value)
            },err=>{
                reject(err)
            })
        }
    })
}

Bromise.any = function(promises){
    return new Bromise((resolve,reject)=>{
        if(promises.length === 0) return reject()
        let pending = 0
        for(let one of promises){
            if(! one instanceof Bromise) continue
            ++pending
            one.then(value=>{
                resolve(value)
            },err=>{
                if(--pending == 0) reject(new Error("No Promise in Promise.any was resolved"))
            })
        }

        if(pending == 0){
            resolve()
        }
    })
}

Bromise.allSettled = function(promises){
    return new Bromise((resolve,reject)=>{
        let result = []
        if(promises.length === 0){
            return resolve(result)
        }

        for(let one of promises){
            Bromise.resolve(one).then(value=>{
                result.push({ status: "fulfilled", value: value })
            },reason=>{
                result.push({ status: "rejected", reason: reason })
            }).finally(()=>{
                if(result.length >= promises.length){
                    resolve(result)
                }
            })
        }
    })
}

Bromise.defer = Bromise.deferred = function () {
    let dfd = {};
    dfd.promise = new Bromise((resolve, reject) => {
        dfd.resolve = resolve;
        dfd.reject = reject;
    });
    return dfd;
}

module.exports = Bromise
```