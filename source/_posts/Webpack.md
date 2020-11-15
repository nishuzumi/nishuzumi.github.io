---
title: 小小说说Webpack
date: 2020-11-13 19:17:37
tags:
catagories:
- [JS]
- [前端]
- [All In Code]
---
## 前言 🎤
众所周知，Webpack是一个静态模块打包工具，主要用于将多个JS文件进行打包成一个，这个是webpack最早的作用。随之时间的推移发展，webpack添加了许多新的功能，并且也增加了强大的定制能力，因此，Webpack也变得复杂了起来。
<!--more-->
## Webpack自己做了什么
Webpack的Loader和Plugin机制允许用户自定义一些解析方式和功能。但是在一个没有配置Loader和Plugin的Webpack内容下，它自身主要做了什么工作呢？
### All in One
webpack的打包功能，最主要的就是将多个使用到的JS文件进行打包，从而进入一个文件之中。原理是通过多个IIFE创建多个独立的作用域，同时替换原生的`require`方法，通过`__webpack_require__`来实现无需文件系统的require方式。
```js
function __webpack_require__(moduleId) {
    // 缓存检测
    if(__webpack_module_cache__[moduleId]) {
        return __webpack_module_cache__[moduleId].exports;
    }
    // 创建一个标准的module模型
    var module = __webpack_module_cache__[moduleId] = {
        exports: {}
    };

    // 执行IIFE函数
    __webpack_modules__[moduleId](module, module.exports, __webpack_require__);

    // 返回结果
    return module.exports;
}
```
不过请注意，这其中只针对`require()`，并不能运用于含有`import`的文件。如果打包时文件中含有`import`，则会出现更为复杂的`__webpack_require__`。
### tree shaking 和 chunk 减重方式
如果把所有内容打包到一个文件之中，其实没啥坏处，有时还是件好事情。但是在某些情况下，它就不再是一件美差事了。  
比如，我们老生常谈的，如果修改了一个文件，则需要更新整个bundle的问题。  
再或者就是，一些固定的vendor文件其实没有改变，但是如果更新，客户端依然需要获取整个文件。  
还有就是，比如在lodash中，我只用到了一两个方法，但是最后给我打包了整个lodash进来。
这些情况，在webpack中其实都有考虑。主要到就是用`tree shaking`和`chunk`进行处理。

#### tree shaking
`tree shaking`说白了就是把没有用到的一些代码给它剔除了，在打包的时候不使用它。而它基于的就是ES6的静态模块结构。  
通过判断是否使用过或者import过来进行标记，将从未使用过的代码部分给剔除，来达到`tree shaking`的效果。
但是，`tree shaking`只会对没有使用过对代码进行标记，但是不会主动进行处理，只有当设置为`production`或者使用了某些具有压缩功能当插件时，才会把没有使用的函数代码进行剔除。

#### chunk
chunk是webpack中一种文件分割方式，最简单的chunk是将两个完全不相关的entry进行分离，当这样他们两个的代码不会复合到一个文件里面，减少了部分的加载量，但是往往最沉重的代码是在vendor之中，比如`lodash`之类的第三方库，因为默认情况下生成chunk会导致将有关联的代码全部放入一个bundle之中。那么，这部分可以公用的第三方库就不能得到缓存机制的便利。比如我从一个页面A进入，接着专跳到页面B，那么这两个文件我都需要进行加载，而且lodash也加载了两次，非常的浪费。如果能把lodash这部分代码拿出来，单独加载，就能剩下很多资源。  

所以webpack又提供了更加详细的分割方式，提供了将import超过1次以上的代码进行单独生成。而在实际加载过程中,分为两种情况。  
如果你是提取common而分割的模块，那么chunk文件则就是一个立即函数，通过将包含实际代码的IIFE注册到self(window)到一个数组中，在需要到时候被触发。  
如果你是动态导入的模块，那么webpack是使用了jsonp来进行异步加载，通过加载完成后调用回调然后注册新加载的内容，执行完成后触发onload回调，接着执行下面的内容。这部分的异步代码使用了Promise进行管理。  

### Loader
loader 让 webpack 能够去处理那些非 JavaScript 文件，比如`import css`，这种情况。loader在本质上来说就是一个对源代码进行转换的转换器,让其变成可以执行的原生js代码。loader中可以指定使用什么loader处理，处理什么样的文件。同时你还可以使用Babel这样的编译器，让一些新语言特性能被比较老旧的环境所支持。  
但是请明确一点，Loader自身只是个处理模式，并没有具体功能，一切功能都由具体的loader进行实现。

同时`Loader`中支持两种方法，第一种是`pitch`，另外一种是直接导出函数。两者可以兼容。  
而且他们的执行顺序有点像一个洋葱🧅。并且为从左到右执行。层级分别为`Pitch A -> Pitch B -> Pitch C -> content -> C -> B -> A`。要注意的是，在`Pitch`中一般会进行一些数据传递或者其他的功能。而在直接方法中则会进行编译。同时，如果在`Pitch`阶段`return`了字符串，那么将会直接跳到同级直接方法之后。`Pitch A -> Pitch B(return 'xxx')->A`。

### Plugin
Plugin是在Webpack运行过程中的一种插入手段。它会在可以监听预先设定好的一些hook，并且进行一些操作和更新。在内部通过`Compiler`对象进行钩子注册。不过要注意的是`Compiler`并不是真正的编译器，它主要还是用于存放监听在编译周期内的hook，以方便注册和执行。  
而如果你想编写Plugin，那么重点需要关注的是`compilation`其中的内容。

**如果说Loader是基于字符串替换的话，那么Plugin就是在整个分析过程中获取非常多的处理好的数据，比如AST等等来进行第二次的处理。**

## devServer
这是Webpack提供的为数不多的外部功能。  
你可以在`webpack.config.js`对它进行配置。它的功能非常多，比如
- 可以设定一个小型服务器，用来运行编译后的文件。  
- 开启热更新功能  
- 提供HTTPS访问方式  
- 通过模拟服务端来进行Proxy从而解决跨域问题  
- 甚至可以在mock服务器api

## HMR
Hot Module Replacement（HMR），是我第一次见到Webpack时觉得最不可思议的功能。优雅，伟大而且充满神秘感。  
当然这个HMR不是说自动刷新页面。  

Webpack的HMR原理其实比较简单易懂。总结下来就是这张图：  
{% img /images/Webpack-2020-11-15-15-38-34.png %}  
在Compiler编译时，会进行增量更新判别，同时会生成完整的bundle文件和增量更新代码，而在浏览器运行的bundle中，带有和HMR服务器通讯部分会获得通知，并且会获得更新后的代码。最后，浏览器的`HMR Runtime`会判断影响范围，并且替换被影响的模块。当然，如果无法识别的时候，则会触发整页刷新。

## 总结 👨‍🏫
Webpack是一个拓展性非常非常高的打包工具，它做到了不单单是打包📦的功能，甚至可以说它做到了一个编译器的全部流程，你可以在任何地方接入这个流程。你甚至可以自己创造新的语法，然后轻松简单的使用Loader进行内容替换。在我了解Webpack之前，我一直不能理解它是什么，是编译器？是打包工具？是压缩混淆工具？其实，都不是，但是又可以认为都是。他就好像是一个简单的工厂，但是提供了你对这个工厂进行任意改造的超强大自定义能力。