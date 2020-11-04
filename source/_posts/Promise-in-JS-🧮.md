---
title: "Promise in JS \U0001F9EE"
date: 2020-11-04 23:02:17
tags:
---
下面的内容为Promise的JS全实现，但是效率并不如原生Promise，因为原生Promise使用了微任务，而为了模拟同样的效果，只能使用宏任务setTimeout来替代。

为了防止在原生的Promise对象上进行修改，导致某些错误的功能误打误撞正确了，所以使用一个新的名称，取名为Bromise。

先睹为快
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