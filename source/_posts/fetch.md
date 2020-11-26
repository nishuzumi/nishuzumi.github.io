---
title: fetch
date: 2020-11-26 19:53:59
tags:
---
# 前言 🎤
Fetch是现代浏览器中提供的一个用于取代XHR的网络请求方式
<!--more-->

# 简单使用
```js
fetch('http://baidu.com').then(response=>{
    return response.json()
})
.then(json=>{
    console.log(jsoo)
})
```
# 常用的参数
在fetch函数中，可以使用第二个参数来进行一些配置。
```js
fetch(url,{
    body:JSON.stringify(data),
    cache:'no-cache',
    credentials:'same-origin', // include
    headers:{
        'usage-agent':'Mozilla/4.0',
        'content-type':'application/json'
    },
    method:'POST',
    mode:'cors',
    redirect:'follow',
    referrer:'no-referrer'
})
```

# 带凭据请求
为了让浏览器带上凭据（包括跨源），需要将`credentials`设置为`include`，如果你想只有在同一个源的时候带上凭据，那么可以设置为`same-origin`，如果你想不带凭据，可以设定为`omit`

# Response对象
当fetch处理完成请求后，会返回一个`Response`对象，其中常用的属性有：
- status 状态码
- statusText 状态字符串（比如OK）
- ok 如果status在200～299之间，则返回true，否则false

# body
参数中的body可以传入多种类型
- ArrayBuffer
- ArrayBufferView
- Blob/File
- string
- URLSearchParams
- FormData

同时，返回的Body类（Request和Response都实现了）会有以下方法
- arrayBuffer()
- blob()
- json()
- text()
- formData()

# 检测
如果你想直到浏览器是否支持`fetch`可以使用
```js
if(self.fetch){
    //yes
}else{
    //no
}
```