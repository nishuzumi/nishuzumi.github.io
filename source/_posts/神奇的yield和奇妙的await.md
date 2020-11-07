---
title: 神奇的yield和奇妙的await
date: 2020-11-06 18:25:59
tags:
catagories:
- [JS]
- [前端]
- [All In Code]
---
## 前言 🎤
说实话，yield这东西，在平时使用的是真的真的非常非常的少，他的出现频率对于大多人来说不亚于在Virtual Studio中随便打一个字符然后出现一个超级长的自动补全内容，而且这个超长自动补全恰好是你需要的内容一样的低概率。

但是，如果对于一些特殊情况，而且你恰好有适用场景，那么这个yield可以说是一把利剑。

而Babel中，把async转化成原生代码的插件`transform-async-to-generator`也是利用了yield进行。所以，在理解await之前，必须要先理解yield。
<!-- more -->
## YIELD

首先我们要理解，yield是什么，有什么用，能干什么，怎么才能去获取Promise的控制权。想要了解这些问题，最好的办法就是看个例子。
```js
let ge = (function* (){
    let c = yield 5 // ge.next()=>
    console.log(c)// =>15
})()

console.log(ge.next()) //=>{ value: 5, done: false }
console.log(ge.next(15)) //=>{ value: undefined, done: true }

```
如果是聪明的小伙伴，到这可能就已经明白了await/async他们的转换原理。如果没看懂也没关系，我们这里慢慢讲解。
首先上面发生了几件事情：
第一件是执行了一个IIFE，然后`function* ()`就生成了一个`iterable`。注意，`function* ()`中的这个星号(*)，是写在function关键词之后的，不是写在括号前面的。千万不要认为这个星号是一个函数名称。有些人喜欢写成`function *()`，这些人要么就是不知道这是啥东西，要么就是要故意让你感到迷惑。都是些坏家伙😠。

在获得一个迭代器后，当程序第一次调用`iterable.next()`，程序会立刻执行，知道遇到`yield`，同时`next()`的返回结构为`{value:any,done:bool}`。如果done为true，那么value将会是undefined，这就证明迭代器中的内容已经完全的执行完成了。同时，如果迭代器中显式return，同样的也算执行完成。你可以无限次的执行next()，但是，只要迭代器已经执行完成，那么一定会返回`{value:undefined,done:true}`。

当然，上述代码中，第一次next并没有执行完成，所以他会返回`{ value: 5, done: false }`。同时，当你继续调用`next()`,程序将会继续执行，不过，上述代码调用的时候传入了一个参数，值为`15`，那么，这个值会被当成上个yield的结果返回到迭代器中继续执行，所以此时`c == 15`。

## BSYNC
了解了yield后，我们就来尝试实现一个async/await。当然，我们在代码中并没有直接修改js关键词和解析器的能力。所以我们这里会使用一些类似的方法来模拟。我们会创建一个新的函数`bsync`来模拟`async`，同时我们用`yield`来替代`await`

我们先尝试实现一个简单的。
```js
function bsync(fn){
    let ge = fn()
    return ge.next().value
}

console.log(bsync(function* (){
    yield 5
}))
```
好了，我们成功的实现了一个毫无作用的`bsync`函数。
很明显，我们需要yield的是一个Promise，而不是一个立即数，所以说我们需要`yield <Thenable>`。
```js
function bsync(fn){
    let ge = fn()
    return ge.next().value
}

console.log(bsync(function* (){
    yield new Promise((resolve)=>{
        console.log(10)
        resolve(10)
    })
}))

//10
//Promise {10}
```
很明显，我们想使用async的目的不是为了这个。如果为了达到多个Promise顺序执行。我们需要加一点细节。
```js
function bsync(fn){
    let ge = fn()
    let next = function(result){
        if(result.done == true) return
        next(ge.next())
    }
    return next(ge.next())
}

bsync(function* (){
    yield new Promise(r =>{
        console.log(10)
        r(10)
    })
    yield new Promise(r =>{
        console.log(20)
        r(20)
    })
})

//output
//10
//20
```
但是这并没涉及到Promise的异步内容，因为构造Promise的函数是会被立刻执行的。所以说我们的`bsync`其实没有发挥任何作用。那么我们来试试多加一个`then`
```js
function bsync(fn){
    let ge = fn()
    let next = function(result){
        if(result.done == true) return
        next(ge.next())
    }
    return next(ge.next())
}

bsync(function* (){
    yield new Promise(r =>{
        console.log(10)
        r(10)
    }).then(v=>{
        console.log(v + 5)// expect 20
    })
    yield new Promise(r =>{
        console.log(20)
        r(20)
    })
})
//output
//10
//20
//15
```
bad news，我们的bsync并不能达到预期效果，所以我们还需要修改一下，我们需要让`next`的调用时机在上一个`result`的Promise结束后才调用。所以我们套一个Promise上去。为什么只需要套一个Promise？还记得Promise的return结果么，无论then有多长多少，你永远会获得最后一个then产生的Promise。
```js
function bsync(fn){
    let ge = fn()
    let next = function(result){
        if(result.done == true) return
        Promise.resolve(result).then(()=>{
            next(ge.next())
        })
    }
    return next(ge.next())
}

bsync(function* (){
    yield new Promise(r =>{
        console.log(10)
        r(10)
    }).then(v=>{
        console.log(v + 5)// expect 20
    })
    yield new Promise(r =>{
        console.log(20)
        r(20)
    })
})
//output
//10
//15
//20
```
great!我们成功的实现了async/await！但是我们需要多添加一些测试用例，来验证我们的代码。
```js
function bsync(fn){
    let ge = fn()
    let next = function(result){
        if(result.done == true) return
        Promise.resolve(result).then(()=>{
            next(ge.next())
        })
    }
    return next(ge.next())
}

bsync(function* (){
    yield new Promise(r =>{
        console.log(10)
        r(10)
    }).then(v=>{
        console.log(v + 5)// expect 20
    })
    console.log('tmp')
    yield new Promise(r =>{
        console.log(20)
        r(20)
    }).then(()=>{
        console.log(40)
    })
    let ten = yield Promise.resolve(10)
    console.log(ten)
})
/*output
10
15
tmp
20
40
undefined
*/
```
大部分情况还是符合预期的，但是最后一个就出了点小问题，最后一个应该为10，而不是undefined。所以我们再次修改bsync
```js
function bsync(fn){
    let ge = fn()
    let next = function(result){
        if(result.done == true) return
        Promise.resolve(result.value).then((value)=>{
            next(ge.next(value))
        })
    }
    return next(ge.next())
}

bsync(function* (){
    yield new Promise(r =>{
        console.log(10)
        r(10)
    }).then(v=>{
        console.log(v + 5)// expect 20
    })
    console.log('tmp')
    yield new Promise(r =>{
        console.log(20)
        r(20)
    }).then(()=>{
        console.log(40)
    })
    let ten = yield Promise.resolve(10)
    console.log(ten)
})
/*output
10
15
tmp
20
40
10
*/
```
啊哈！完美！
也不对，那么如果我`throw new Error`或者`Promise.reject`会发生什么？
```js
function bsync(fn){
    let ge = fn()
    let next = function(result){
        if(result.done == true) return
        Promise.resolve(result.value).then((value)=>{
            next(ge.next(value))
        })
    }
    return next(ge.next())
}

bsync(function* (){
    try{
        yield new Promise(r =>{
            console.log(10)
            r(10)
        }).then(v=>{
            console.log(v + 5)// expect 20
        })
        console.log('tmp')
        yield new Promise(r =>{
            console.log(20)
            r(20)
        }).then(()=>{
            console.log(40)
            throw new Error('boom')
        })
        let ten = yield Promise.resolve(10)
        console.log(ten)
    }catch(e){
        console.log(e.message)
    }
})
/* output
10
15
tmp
20
40
(node:34117) UnhandledPromiseRejectionWarning: Error: boom ....
*/
```
bad~，很明显，核弹捕获失败了。所以我们需要在bsync中加入关于错误处理的部分代码。
```js
function bsync(fn){
    let ge = fn()
    let next = function(result){
        if(result.done == true) return
        Promise.resolve(result.value).then((value)=>{
            next(ge.next(value))
        },reason=>{
            next(ge.throw(reason))
        })
    }
    return next(ge.next())
}

bsync(function* (){
    try{
        yield new Promise(r =>{
            console.log(10)
            r(10)
        }).then(v=>{
            console.log(v + 5)// expect 20
        })
        console.log('tmp')
        yield new Promise(r =>{
            console.log(20)
            r(20)
        }).then(()=>{
            console.log(40)
            throw new Error('boom')
        })
        let ten = yield Promise.resolve(10)
        console.log(ten)
    }catch(e){
        console.log(`onError:${e.message}`)
    }
})
/* output
10
15
tmp
20
40
onError:boom
*/
```
啊哈！我们成功啦。我们成功的完成了async/await的所有功能！cheers🍻！！
最后简单的修复一下最后返回结果的问题。
```js
function bsync(fn){
    let ge = fn()
    let next = function(result){
        if(result.done == true) return Promise.resolve(result.value)
        Promise.resolve(result.value).then((value)=>{
            next(ge.next(value))
        },reason=>{
            next(ge.throw(reason))
        })
    }
    return next(ge.next())
}
```
这样以来，我们就成功的创建了一个能应付大多数情况的伪async了。

## 结语 👨‍🏫
其实async的原理和yield的原理是差不多的。我觉得他只是一个对yield进行封装的高阶语法糖。但是，async的性能是要远远高出连续then的，最重要的原因是，async只开辟了两个栈空间，而直接使用Promise.then，v8就需要保存之前的栈环境。当然了，更加具体的优化，就是由v8来进行处理了。