---
title: 简单实现深拷贝
date: 2020-11-19 17:28:41
tags:
catagories:
- [JS]
- [前端]
- [All In Code]
---
## 前言 🎤
在JS中，默认的语言特性并没有提供深拷贝功能，只提供了浅拷贝，比如`Object.assign()`。所以本文将会简单的，通过两种方式实现深拷贝。
<!--more-->
## 递归
```js
function copy(obj,cache){
    cache = cache || new WeakMap()
    if(typeof obj != 'object' || obj === null){
        return obj
    }
    // 判断是否为对象 不是就直接返回
    
    if(cache.has(obj)) return cache.get(obj) // 查询缓存

    let cloneObj = Array.isArray(obj)?[]:{}
    cache.set(obj,cloneObj)
    for(key of Object.keys(obj)){
        cloneObj[key] = copy(obj[key],cache) // 递归掉用
    }
    return cloneObj
}
```
其实想法很简单，不是对象的直接赋值，是对象的看看有没有生成过，没有就调用递归调用，有就直接拿值。为什么要用`WeakMap`？懂得都懂。

## 迭代
迭代的想法和递归的有些变动，但是核心思想还是万变不离其宗。
```js
function copy2(obj){
    let cache = new WeakMap() // 查找表（不代表拷贝完成）
    let tasks = []
    function cloneT(o){
        return  Array.isArray(o)?[]:{}
    }
    if(typeof obj != 'object' || !obj) return obj

    let cp = cloneT(obj)
    tasks.push(obj) // 任务队列，有对象需要拷贝就会放进来
    cache.set(obj,cp) // 设置对应的clone

    while(tasks.length > 0){
        let task = tasks.shift() // 找到一个任务
        let clone = cache.get(task) // 找到对应的clone

        for(let [key,value] of Object.entries(task)){
            if(typeof value != 'object' || !value){ // 不是对象直接返回
                clone[key] = value
                continue
            }
            if (!cache.has(value)){ // 是对象 但是不在查找表内就创立一个对应的类型 用于写入
                tasks.push(value)
                cache.set(value,cloneT(value))
            }
            clone[key] = cache.get(value) // 设置引用
        }
    }
    return cp
}
```

## 总结 👨‍🏫
总的来说还是很简单的，但是这里面可能没考虑到更多的情况，还有待优化和完善。