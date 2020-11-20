---
title: 请求 in Browser
date: 2020-11-19 15:18:15
tags:
catagories:
- [JS]
- [前端]
- [浏览器]
- [All In Code]
---
## 前言 🎤
本文会总结在浏览器中常见的网络请求方式。

<!--more-->
## Ajax
老生常谈Ajax，不支持跨域。
```js
function ajax(method,url,cb,data = {}){
    let u = new URL(url)
    let dataRaw = ''
    if(method.toLowerCase() == 'get'){
        Object.keys(data).forEach((key)=>u.searchParams.set(key,data[key]))
    }else{
        dataRaw = Object.keys(data).map((key)=>`${key}=${data[key]}`).join('&')
    }

    let xhr = new XMLHttpRequest
    xhr.onreadystatechange = function(){
        if(xhr.readyState === 4){
            if(xhr.status === 200 || xhr.status === 304)
                return cb(xhr.responseText)
            throw new Error(xhr.status)
        }
    }

    xhr.open(method,u.toString(),false)
    if(method.toLowerCase() == 'post'){
        xhr.setRequestHeader('Content-Type','application/x-www-form-urlencoded')
    }
    xhr.send(dataRaw)
}
```
要注意的几点是：
- 发送post时候需要设置头部
- 只有当readystate为4时请求才算完成
- 错误的情况一样要等到readystate为4才能得知

## Fetch
Fetch是现代化浏览器提供的一个高层封装的网络请求方法。他可以让你用非常简明优雅的方式创建网络请求。支持跨域。
```js
fetch('http://example.com/movies.json')
  .then(function(response) {
    return response.json();
  })
  .then(function(myJson) {
    console.log(myJson);
  });
```
### Fetch的强制中断
Fetch有一个singal功能，允许开发者强制中断请求，只不过这个特性不是支持情况参差不齐。
```js
let controller = new AbortController()
fetch('http://example.com/movies.json',{signal:controller.signal})
```

## 长轮询(Long polling)
长轮询是一种机智而且巧妙的轮询方式，他能减少轮询带来的网络占用，同时也能保证消息的及时性。  
长轮询和普通轮询操作方式基本一样，差别在于服务器的返回控制。这需要服务端程序能支持挂起链接，在一般情况下，各种后端程序都可以支持。  
当客户端发起请求时，后端会接受请求，并且处于长时间的等待状态，直到某些条件满足时，才会进行数据返回。比如，客户端需要监听用户是否点击邮箱内的激活链接，当发起请求时，后端将会挂起前端请求，直到用户点击激活链接并激活，后端才返回链接。接着前端根据需求决定是否继续发起请求。

这种请求方式一般放在消息量较少的情况，如果消息量增大，可以选用`WebSocket`或者`Server Send Events`。

## Server Send Events
`Server Send Events`是一种较早出现的单向数据传输方案（但是IE依然不支持）。它允许一种订阅的方式让服务器不断的传输数据到客户端。这种方法需要前后端进行配合改造，并不是能直接无痛使用的方案。同时，他也是一个HTTP协议的请求。支持跨域。

### 发起事件流
```js
let eventSource = new EventSource("https://www.example.com/index.php")
```
### 接受数据
服务端需要使用一种特殊的格式来写入
```
//etc
data: hello? 

data: i am
segment

data: ok
``` 
如上所示，数据需要用`\n\n`进行分离，才算一个信息段。同时要以`data:`开头。

客户端则需要
```js
eventSource.onmessage = function(ev){
    console.log(ev.data)
}
// or addEventListener
```
好处不止这些，我们不需要处理断线的情况，客户端会帮助我们自动重连，这可是WebSocket没有的好处。不过，在遇到一些错误的状态码（除:200.301,307），或者返回的头部中不为`Content-Type: text/event-stream`。则不会进行重新连接。

如果你想主动断开连接，则需要使用`evnetSource.close()`。
### 消息ID
通常情况下，在每个消息应该附带一个`id:`比如：
```
data: hello
id: 1

data: world
id:2

```
当浏览器检测到具有ID内容时，则会自动的更新`eventSource.lastEventId`。并且在断行重连时，会设置一个`Last-Event-ID`的头部。

`Server Sent Events`相比`WebSocket`来说，功能更为轻量，但是在某些特殊场景下，也能发挥不错的效果。

## WebSocket
伟大的WebSocket是前端中最为核心的一个网络请求功能。对于实时性高的网络应用，它简直一项是开天辟地的创举，这标志着网页多人游戏的无依赖网络层的诞生，标志着前端交易系统可能性的胜利，标志着前端游戏无框架主义开发者从此站起来了！  
`WebSocket`使用者一种特殊的协议，常常表现为`ws://`或者`wss://`开头。一个`WebSocket`对象上有着四种基础事件,`open`,`message`,`error`,`close`。如果你想发送数据，则可以使用`.send()`进行发送。支持跨域。

因为使用`WebSocket`地难度已经达到了一个令人发指的简单程度，所以就不再进行简单内容的叙述。  
而且`WebSocket`知名和流行程度已经达到了一个新的境界，如果你说你是前端不知道`WebSocket`，那么真有可能会贻笑大方。

## 总结 👨‍🏫
`Ajax`是浏览器最早出现的网络请求方式，在此之前JS只是一个网页小玩具。随之事件的进步和发展，又出现了`Server Send Events`这种高级一点的信息传递方式。再后来现代浏览器又提供了`fetch`和`WebSocket`这两种对标`Ajax和Server Send Events`的新功能。互联网的发展之速度仿佛转上了加速器一般大步向前。