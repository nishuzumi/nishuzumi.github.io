---
title: 'new,apply,bind,call 四大天王'
date: 2020-11-08 20:21:01
tags:
catagories:
- [JS]
- [前端]
- [All In Code]
---

## 前言 🎤
先来句毒鸡汤

> “What I Cannot create, I do not understand” — Richard Feynman

作为JS家族中的四大天王，new，apply，bind，call。可谓是出现频率极高，虽然用法简单，效果清晰，但是我也一直没去仔细了解过他们的运行机制和实现方法。千载难逢的机会，那就来一步一步的仔细研究一下。

先说明，new是一个关键词，而bind，apply，call，均为Function.prototype中的方法。
<!-- more -->
## new 🆕
先看定义
>new 运算符创建一个用户定义的对象类型的实例或具有构造函数的内置对象的实例。
嗯，MDN上面的内容很是学术。简单说就是创建一个对象。

### 实现


```js
// 注意，任何与实现无关的新特性或者语法我都会用上，比如这个可变参数
// 下面是一个非常常见的实现方法
function bNew(Target,...args){
    let thisValue = Object.create(Target.prototype) // 创建一个新对象 并把新对象的__proto__指向传入参数
    let result = Target.apply(thisValue,args) // 提供上下文和参数 运行构造函数
    if(typeof result == 'object' && result != null){ // typeof null === 'object'
        // 如果构造函数中返回了一个对象，那么就以返回对象为结果（Case 1）
        return result
    }
    return thisValue;
}

// Case 0
function Computer(cpu,mem){
    this.cpu = cpu
    this.mem = mem
}

console.log(bNew(Computer,'4H',"8G"))// Computer { cpu: '4H', mem: '8G' }
console.log(new Computer('4H','8G')) // Computer { cpu: '4H', mem: '8G' }
console.log(bNew(Computer,'4H',"8G").__proto__ === new Computer('4H','8G').__proto__) // true

// Case 1
function Factory(cpu){
    return new Computer(cpu,"8G")
}

console.log(bNew(Factory,'4H'))// Computer { cpu: '4H', mem: '8G' }
console.log(new Factory("4H")) // Computer { cpu: '4H', mem: '8G' }
console.log(bNew(Factory,'4H').__proto__ === new Factory('4H').__proto__) // true
```
就这么简单，无他，唯手熟尔。（指记忆的方法）

## apply 🍎
> apply() 方法调用一个具有给定this值的函数，以及以一个数组（或类数组对象）的形式提供的参数。
刚刚我们在实现`new`的时候，以及用过了`apply`方法。所以会稍微有些了解。
其实就是把某个函数的this换成了传入的第一个参数并且执行。而且和call的区别不大，两者区别是apply接受参数以数组形式传递，而call为列表形式。

### 实现
看了一下资料，发现现在主流的做法都是，将一个函数的`this`传入，因为函数的`this`表示了它本身，如果让目标函数中添加一个新的方法，那么那个方法中的this则会被默认指向到其对象。比如：
```js
let count = {
    x:0
}
count.add = function(){
    this.x ++
}
count.add()
console.log(count.x) //1
```
同理
```js
let Count = function(){
    this.x =0
}

Count.prototype.add = function(){
    this.x++
}

let c1 = new Count()

//==== 模拟 ====//
c1['mockApply'] = function(){
    this.x += 10
}
c1.mockApply()
console.log(c1) // Count { x: 10, mockApply: [Function] }
```
很明显，我们的mockApply方法已经获得了目标对象的this，也就是说，把函数都运行在这个目标对象上，那么就可以实现apply的功能。而获取函数的方法也很简单。
```js
Function.prototype.getSelf = function(){
    return this
}

function test(x){
    return x
}

test.getSelf()(10) // 10
```
那么准备工作已经结束，我们就开始来编写自己的apply。
```js
Function.prototype.bpply = function(target,args){
    target = target || global
    let methodName = `mock${Math.random()}`
    while(target.hasOwnProperty(methodName)){
        methodName = `mock${Math.random()}` // 希望永远不会执行到这里
    }

    target[methodName] = this
    let result = target[methodName](...args)
    delete target[methodName] // 假装无事发生过
    return result
}

Math.max.bpply(null,[1,2,3]) // 3

let Count = function(){ this.x =0 }

Count.prototype.add = function(){ this.x++ }

let c1 = new Count()
let c2 = new Count()
c2.x = 10

// c2.add.apply(c1) c1.x = 1 ; c2.x = 10
c2.add.bpply(c1) // c1.x = 1;c2.x = 10
```
简单来说就是获取this，然后执行函数，再最后处理痕迹。

## bind 🩹
什么是bind？bind是一个把函数的this进行指定，并且创建一个新的函数。当新函数被调用时，将会在this参数上执行bind的函数。
```js
let x = [1,2,3]
let reduce = Array.prototype.shift.bind(x) //bind
reduce() // call bind

console.log(x) //[2,3]
```

### 实现

bind的介绍中有一句话
>绑定函数也可以使用 new 运算符构造，它会表现为目标函数已经被构建完毕了似的。提供的 this 值会被忽略，但前置参数仍会提供给模拟函数。
所以需要注意一下

```js
//以下代码将使用新特性方便代码理解
Function.prototype.bBind = function(that,...args){
    let target = this
    let F = function(){}
    if (this.prototype) // 不要忘记了 当前this指向的是Function.prototype 而Function.prototype.prototype是不存在的
        F.prototype = this.prototype
    let bound = function(...newArgs){
        let finalArgs = args.concat(newArgs) // 合并两次参数 你可以认为是 xx.bBind(that,args...)(newArgs...)
        return target.apply(this instanceof F ? this : that,finalArgs) 
        //注意这个this指向的是bound。当返回的bound函数被用作为构造函数的时候(new bound)，忽略之前绑定的this，也就是忽略that
    }
    bound.prototype = new F()
    return bound
}
```
当然这个实现其实有问题。比如：
>这部分实现依赖于 Array.prototype.slice()， Array.prototype.concat()， Function.prototype.apply() 这些原生方法。
>这部分实现创建的函数并没有 caller 以及会在 get，set 或者 deletion 上抛出 TypeError错误的 arguments 属性这两个不可改变的“毒药” 。（假如环境支持Object.defineProperty()， 或者实现支持__defineGetter__ 和 __defineSetter__ 扩展）
>这部分实现创建的函数有 prototype 属性。（正确的绑定函数没有）
> 这部分实现创建的绑定函数所有的 length 属性并不是同ECMA-262标准一致的：它创建的函数的 length 是0，而在实际的实现中根据目标函数的 length 和预先指定的参数个数可能会返回非零的 length。

也就是说，当你对某个function去获取length的时候，返回的内容是你必须提供参数的数量。而使用这个方法bind的结果不具有这个特性。
如果你想知道更加完整的符合规范的实现。可以看[这个](https://github.com/Raynos/function-bind/blob/master/implementation.js)
## call ☎️
有人可能有疑问，为什么没有call！！😠

还记得call和apply的区别吗？
>注意：该方法的语法和作用与 apply() 方法类似，只有一个区别，就是 call() 方法接受的是一个参数列表，而 apply() 方法接受的是一个包含多个参数的数组。 - MDN
```js
// lazy man create a lazy call
Function.prototype.ball = function(that,...args){
    return Function.prototype.apply(that,args)
}
```

## 结语 👨‍🏫
其实多实现一点这种js的底层函数，有利于理解或者加深自己对js的认知。但是往往很多情况下，会越理解越混乱。。所以，当你遇到一些你不懂的操作，一些你不知道什么含义的逻辑，我推荐你还是把那一部分代码注释了，然后看看注释前和注释后的结果，往往这样你可以获得更深刻的理解。