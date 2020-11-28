---
title: fetch
date: 2020-11-26 19:53:59
tags:
---
# 前言 🎤
Fetch是现代浏览器中提供的一个用于取代XHR的网络请求方式
<!--more-->

# 跨域规则
fetch默认情况下为同源规则，只允许同源跨域，并且默认是附带同源cookie（在早期版本时有些浏览器默认为不附带任何凭证）

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

# 参数解析
## method
请求的方法，比如`GET`,`POST`,`PUT`等等
## headers
想要添加的头数据，以对象形式传入，可以小写头部字段
## body
任何你想传入的body，可以是`Blob`,`BufferSource`,`FormData`,`URLSearchParams`,`USVString`，请注意，`GET`和`HEAD`没有Body。
## mode
请求模式，默认为`cors`，可选内容有：`cors`,`no-cors`,`same-origin`,`navigate`。
## credentials
可选内容有：`omit`,`same-origin`,`include`  
分别为忽略，同源携带，强制携带
## cache
可选内容为：`default`,`no-store`,`reload`,`no-cache`,`force-cache`,`only-if-cached`
- default 有缓存且有效则返回缓存，有缓存但是过期则尝试更新，无缓存则直接请求
- no-store 直接请求 无视缓存 而且不保存缓存
- reload 直接请求 无视缓存 但是保存缓存
- no-cache 有缓存但是直接请求，如果缓存有效，则用缓存，否则更新
- force-cache 强制使用缓存，不管过期
- only-if-cached 只找缓存，如果没有就报错
## redirect
重定向处理模式：`follow`,`error`,`manual`。默认情况下为自动处理。
## referrer
可选内容为：`no-referrer`,`client`。默认为`client`

# 附加内容
## FormData
该接口提供了一种使用键值对的方式进行构造，同时也可以轻松的通过`XMLHttpRequest.send()`进行发送。FormData无法直接使用对象进行内容填充，只能通过不断的set来添加内容。同时set函数中，如果第二个参数是一个blob或者file内容，那么第三个参数可以传入一个文件名。
## URLSearchParams
## USVString