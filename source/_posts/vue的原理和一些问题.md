---
title: vue的笔记
date: 2020-11-23 12:35:31
tags:
catagories:
- [JS]
- [前端]
---
# 前言 🎤
一个小小的笔记本📒
<!--more-->
# 变化检测
## Object的变化检测
在getter中收集依赖，在setter中更新依赖。  
当在getter中检测到某个函数调用到时候，就把它推入一个`Dep`对象中进行管理。

### definedReactive
definedReactive会给每个属性增加监听捕获的能力，会在内部维护一个`Dep`对象。他给每个属性都添加上了`getter`和`setter`。用来记录和触发对应的行为。

### Watcher
watcher则是真正需要监听数据和触发回调的部分。比如：
```js
vm.$watch('a.b.c',()=>{newVal,oldVal})
```
当`data.a.b.c`发生改变时，触发回调。

Watcher会通过触碰getter的方式注册回调到Dep中，方便未来触发`setter`时找到`watcher`。
有了Watcher还需要对对象中每一个元素进行修改，添加getter和setter。
但是在Vue2中，无法对增加data中属性进行监听，也无法对删除进行监听。因为无法监听。所以Vue提供了vm.$set和vm.$delete  

### Observer
Observer就是给对象的每个属性添加回调注册的对象。

### 小结
Dep为依赖存储，Watcher为依赖添加和回调器，Observer用于给每个属性添加依赖存储。

{% img /images/vue的原理和一些问题-2020-11-23-14-17-45.png %}

## Array的变化检测
添加拦截器进行拦截，只能拦截改变数组的方法，比如`push,pop,shift,unshift,splice,reverse,sort`。
基本原理是创建一个新的原型对象，同时设定这些方法，加入拦截器。这样如果遇到数组类型，就可以直接设定`__proto__`。如果不能设定，就直接替换数组上的方法实例（准确的说应该是添加）。  
不同的是，对数组的`Dep`不在`defineReactive`创建的闭包之中，而是在`Observer`之中。  
当一个数组触发了7个方法之一时，就会在自身上尝试查找`__ob__`,然后向其中的dep进行触发通知。  
与此同时，不单单是数组的变化需要监听，数组中的对象也是一样需要监听，同样的会进行遍历，创建`Observer`。同时还要在拦截原型中对`push,unshift,splice`进行新元素的内容拦截和注册`Observer`。
但是对于直接进行下标访问的时候，比如`this.list[0] = 2`，这种是无法进行监听的，所以无能为力。

### 小节
Array的拦截有些区别，需要在做一些特殊处理，并且只能拦截方法调用，对于下标访问这种实在是无能为力。

## 变化检测相关API实现
### vm.$watch
```js
/**
* @param {string | Function} expOrFn
* @param {Function |Object } callback
* @param {{ deep:Boolean,immediate:Boolean }} options
* @return {Function} unwatch
*/
vm.$watch = function(expOrFn,callback,options){
    //...
}
```
这个用于观察一个表达式或者一个`computed`函数的变化内容。

记录一下两个参数作用
#### deep 将会监听内部内容
```js
vm.$watch('obj',cb,{deep:true})
vm.obj.inner = 123
// cb将会被触发
```
### immediate 注册时会立刻触发回调
如果设定为true，那么在注册时会立刻触发，不需要等待到监听内容发生变化。

### watch实现原理
watch其实是对Watcher对一种封装。
```js
Vue.prototype.$watch = function(expOrFn,cb,options){
    const vm = this
    options = options || {}
    const watcher = new Watcher(vm,expOrFn,cb,options)
    if(options.immediate){
        cb.call(vm,watcher.value)
    }
    return function unwatchFn(){
        watcher.teardown()
    }
}
```
同时，为了实现取消监听，在Watcher中也要保存对应监听的`dep`内容。于是就会产生双向绑定。  
{% img /images/vue的原理和一些问题-2020-11-23-15-55-26.png %}  

因为巧妙的getter设计，让Vue简单而且方便的实现了对内容的监听，同时还能支持监听多个内容。

#### deep
如果要实现deep，需要给对象的子元素们逐一添加监听。简单来说就是循环遍历，查找`__ob__`，然后判断是否有过监听，如果没有就依次触发getter。

### vm.$set
```js
/**
* @param {Object | Array} target
* @param {string | number} key
* @param {any} value
* @return {Function} unwatch
*/
vm.$set(target,key,value){}
```
用于在object上设定一个属性。这个方法主要用来解决无法检测属性添加的限制。

```js
Vue.prototype.$set = function(target,key,value){}
```
#### 具体实现
处理Array首先需要判断是否为数组，并且key是否为一个合法的数组index。然后会调用数组上的`splice`方法。  
接着判断Object的情况，首先判断是否已经存在，如果已经存在那就直接set触发setter。  
如果是新增，首先会判断`target`是否合法，因为不允许在Vue上进行属性监听。然后，如果`target.__ob__`不存在的话，也不会进行监听处理。只有条件都符合的情况下，就会创建新的Reactive，同时触发父对象的通知。

### vm.$delete
```js
/**
* @param {Object | Array} target
* @param {string | number} key
*/
vm.$delete( target, key )
```
同样的限制，用于删除对象或者数组中的属性。  
#### 具体实现
他的具体实现和set基本是一个流程，具体逻辑稍微一些不同而已。

# 虚拟DOM
Vue2速度的核心点就在于这个Virtual Dom的提速，所以可以说这个是支撑了整个Vue的灵魂功能。它不但降低了DOM渲染带来的速度问题，同时还给跨平台渲染提供了基础。  
早期的Vue使用了更加细粒度的监听方法，导致开销很大，所以在Vue2使用了中等粒度的监听方法。监听级别为一个Watcher。  
在Vue中的虚拟Dom是将一个VNode渲染到视图上。但是如果直接把有元素变动的VNode直接替换DOM，那么其中有很多没变动的元素一样需要进行重计算布局渲染，导致速度下降。所以在每次渲染之前，都会进行DIFF算法，找出实际上变动的元素进行更新。  
实际上，虚拟DOM只做了两件事情
- 提供真实DOM映射的虚拟VNode节点
- 将虚拟节点与旧的节点进行比较，然后更新视图

通过核心算法Patch进行判断，来实现上面的功能。

# VNode
VNode是一个类，他可以实例化出不同的vnode实例。你可以理解vnode为一个节点描述对象，通过它可以去创建一个真正的DOM节点。  
值得注意的是，**Vue的检测策略为组件级别**，当状态发生变化时，通知到组件，然后由组件进行虚拟DOM计算。
vnode类型有以下几种
- 注释节点
- 文本节点
- 元素节点
- 组件节点
- 函数式节点
- 克隆节点

### 注释节点
```js
export const createEmptyVNode = text =>{
    const node = new VNode
    node.text = text
    node.isComment = true
    return node
}

// 内容
{
    text: "注释节点"
    isComment: true
}
```
### 文本节点
```js
{
    return new VNode(undefined,undefined,undefined,String(val))
}
// 
{
    text:"Hello World"
}
```
### 克隆节点
克隆节点是将属性复制，让新旧两个节点保持一致，作用是优化静态节点和插槽节点（Slot Node）
创建克隆节点就是传统意义上的进行克隆，没有什么特殊情况。唯一不同的是`isCloned = true`。  

### 元素节点
元素节点一般映射的是真实元素。存在以下四种有效属性：
- tag：就是标签，如div ul li
- data：节点上的数据，比如class style等
- children：子节点列表
- context：当前组件的vue实例
### 组件节点
和元素节点相似，但是有两个特有属性：
- componentOptions：组件参数，比如propsData，tag，children等
- componentInstance：组件实例，也就是Vue实例，实际上每个组件都是一个Vue实例
### 函数式节点
和组件节点相似，但是有`functionalContext`和`functionalOptions`两个独有属性。

# patch
Virtual DOM最核心的算法就是patch，他可以将vnode渲染成真实的DOM。它也可以称之为patching，主要作用是通过对比结果找出需要更新的节点进行更新。实际上就是在DOM上修改进行更新节点的操作。总所周知，DOM的操作远不如JS计算快，所以如果能尽量小的修改DOM内容，就可以增加渲染速度。  
patch主要做三件事情：
- 创建新增节点
- 删除废弃节点
- 修改需要更新的节点

### 新增节点
当不存在oldVnode节点的时候，就会使用vnode直接进行渲染。  
同时，当oldVnode和vnode完全不是一个节点的时候，也会触发一次新增节点渲染。  

### 删除节点
就是简单的删除节点

### 更新节点
当两个节点是同一个节点，但是内容不太相同是，就会使用对比内容变化进行更新。

## 创建节点
创建节点的过程也非常的简单，当一个vnode节点含有tag属性时，就可以知道他是一个元素节点，通过遍历vnode的children节点，依次进行`document.createElement`就可以创建出实际的DOM节点，然后通过`parentNode.append`插入页面。同样的，如果是文本节点，那么可以通过`document.createTextNode`创建。如果是注释节点，就通过`document.createComment`创建。

## 删除节点
删除节点同样的简单，记录下要删除的开始和结束index，然后循环中将其删除即可。

## 更新节点
更新节点分为多种情况
### 静态节点
某些永远不会发生改变的节点我们称之为静态节点，比如`<p>我是静态节点</p>`，这种节点一旦渲染完成就不会收到影响，因为他们不包含任何的动态数据处理。  
### 虚拟节点
当两个vnode节点不是静态节点的时候，就需要进行对比判断，首先判断的情况是有`text`属性。  
#### 有Text
如果有text，那么就会直接调用`setTextContent`方法--`node.textContent`，将DOM节点内容改为text内容。
#### 无Text
如果没有Text，那么有可能是一个元素节点，这个时候又有两种情况。  
1. 有children的情况
如果有children那么就会对children进行比较，如果oldVnode没有children，那么旧节点可能是文本节点或者是一个空标签，如果是文本节点，就把它变成一个空标签，然后children中对节点挨个加入。
2. 没有对情况
如果新节点没有children，那么就说明是个空节点，此时直接删除旧节点中的各种内容，让他变成一个空标签。

## 更新子节点
更新子节点分为四种操作，更新，新增，删除，移动。
### 更新策略
1. 创建子节点
循环比较的时候，如果发现old之中没有新的节点，那么就会产生一个插入动作。
{% img /images/vue的原理和一些问题-2020-11-24-13-37-19.png %}
2. 更新子节点
当在两个vnode之中都存在一个相同的节点，那么就需要进行更新操作
3. 移动子节点
当某个节点相同，但是位置不同的时候，就需要进行移动子节点。通过`Node.insertBefore()`即可移动。
4. 删除子节点
当节点在newChildren中存在，在old中不存在的时候，就会产生删除。

### 优化策略
有一个优化检测方式，会尝试新旧children的开头结尾元素是否相同，同时还会去对比是否交叉相同。  
{% img /images/vue的原理和一些问题-2020-11-24-13-52-40.png %}  
通过这个方法不断的从两侧向中间收敛判断，如果是在是不符合这些条件，就会使用暴力查找。  

# 模版编译
## 模版编译为渲染函数
模版编译大体分为三个部分：
- 解析为AST
- 遍历AST标记静态节点
- 使用AST生成渲染函数
由三个模块实现，分别是：
- 解析器
- 优化器
- 代码生成器
{% img /images/vue的原理和一些问题-2020-11-24-14-31-05.png %}  
### 解析器
解析器分为好几个包括HTML，文本，过滤器解析器。文本解析器主要用于解析带变量的问题，比如`Hello {{ name }}`。  
而HTML解析器比较重要，当HTML解析起解析到标签开始结束，或者文本，注释时，就会触发钩子函数。主线上要做的事情就是监听HTML解析器，当钩子函数触发时，就生成AST节点。

### 优化器
优化器用来遍历AST找出静态子树，并且打上标记。每次重渲染，不会为静态节点创建新的虚拟节点，而是直接克隆。

### 代码生成器
代码生成器则是编译的最后步骤，他将AST转换成渲染函数，也可以称之为代码字符串。  
比如`<p title="Berwin" @click="c">1</p>`生成的是：`with(this){return _c('p',{attrs:{"title":"Berwin"},on:{"click":c}},[_v("1")])}`。将其格式化，得到的结果是：
```js
with (this) {
    return _c(
        'p',
        {
            attrs: { "title": "Berwin" },
            on: { "click": c }
        },
        [_v("1")]
    )
}
```
随后通过`new Function(code)`即可创建一个函数。

# 实例方法和全局API
在Vue中，有一些Mixin函数，他们的作用其实就是把一些操作挂载到目标对象的prototype之上。同时Vue还提供了`$on,$once,$off,$emit`这几个用于事件的方法。这几个方法和js的事件系统基本保持一致。

## 生命周期相关方法
关于生命周期的方法有
- $mount
- $forceUpdate
- $nextTick
- $destory

### forceUpdate
`vm.$forceUpdate`会让Vue强制重新渲染，，不过它只会影响实例本身和插入插槽的子组件，而不是所有的子组件。
```js
Vue.prototype.$forceUpdate = function(){
    const vm = this
    if(vm._watcher){
        vm._watcher.update()
    }
}
```

### destory
在destory中，主要需要做的就是清理和其他实例的连接，解绑监听和指令。
```js
Vue.prototype.$destory = function(){
    const vm = this
    if(vm._isBeingDestoryed) return
    callHook(vm.'beforeDestory')
    vm._isBeingDestoryed = true
}
```
这其中执行了一次生命周期事件调用。  
具体的删除逻辑这就不再显示。  

### nextTick
其实就是微任务，只不过在多个版本里面出现了多种解决方案，从微任务到宏任务再到双选择，最后回到了微任务。

### mount
这个方法常常由Vue内部进行调用。

# 生命周期
## 初始化阶段
{% img /images/vue的原理和一些问题-2020-11-24-16-24-37.png %}
这个阶段主要是Vue进行一些属性初始化等内容
## 模版编译阶段
{% img /images/vue的原理和一些问题-2020-11-24-16-25-48.png %}
这个阶段用于编译模版内容到代码文本中。
## 挂载阶段
{% img /images/vue的原理和一些问题-2020-11-24-16-27-46.png %}
这个阶段用于将实例挂载到DOM上，并且会开始watcher进入监听状态。大部分情况下，Vue到组件都在这个阶段。当出现更新时，便会触发`beforeUpdate`，然后是`updated`
## 卸载阶段
{% img /images/vue的原理和一些问题-2020-11-24-16-29-11.png %}
当进入卸载阶段，整个Vue组件就会从父元素中删除，并且取消依赖监听。

# 一些问题
## v-model双向绑定
v-model是一个类似语法糖当存在，他会在不同当元素上注入不同的property和监听不同的事件，来完成双向绑定的操作。
## v-for key问题
当在使用v-for的时候，index和元素并没有产生关联。所以当元素发生变动时，vue只能感知到元素发生了变动，但是不会知道具体怎么改变。因此在patch进行比较的时候，当删除一个元素的时候，vue只能感知到三个元素变成了两个。于是就把最后一个元素删除了。所以，当添加key的时候，vue的判断元素是否相同则会使用key作为标准。因为vue是*就地更新算法*，如果不使用key，就会导致性能浪费。

## computed和watch
computed的内容会在变化后运算，并且生成缓存，如果有地方使用这个内容，并不会重新执行computed。  
watch用于监听对象内容的变化，当内容发生变化时，就会执行watch内容。