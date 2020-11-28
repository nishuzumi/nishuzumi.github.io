---
title: 跨源资源共享
date: 2020-11-26 17:50:23
tags:
---
# 前言 🎤
浏览器本身不允许跨源请求，目的是为了保证安全。
<!--more-->

# 安全性
XHR和Fetch都遵循同源策略，只允许他们从同一个域进行内容加载，除非包含了正确都CORS头。  

# 跨
早期的AJAX确实不支持跨源访问，这个是浏览器和服务器一同决定的，同时IE10以下都不支持CORS，因此，很多时候大家都会说AJAX不支持跨源请求。  
同时，在遇到这几种条件才会触发跨域请求：
- 由XHR和Fetch发起的跨域请求
- Web字体
- WebGL贴图
- `drawImage`绘制图片到canvas时
# 两种请求
CORS分为两种请求，一种是*简单请求*，一种是*非简单请求*。
简单请求需要满足两个条件
```js
['get','head','post'].indexOf(method) != -1
&&
```
头信息不超过这几种：
- Accept
- Accept-Language
- Content-Langugae
- Last-Event-ID
- Content-Type:'application/x-www-form-urlencoded','multipart/form-data','text/plain'

这事需要兼容表单，因为表单一直可以跨域请求。所以早期的AJAX也属于这种情况。

## 简单请求
当有简单请求需要发送时，浏览器并不会进行`preflight`处理，因为这种跨域请求和普通表单毫无两样。不过，这种请求在发送的时候，会强制带上`Origin`的头部，用来给服务器进行同源判断。而服务端如果没有返回`Access-Control-Allow-Origin`，或者不为`*`，或者返回了一个域名但是和`Origin`不是一个域，那么都会请求失败。

## 预检请求 Preflight
当遇到一个非简单请求当时候，浏览器会尝试发出预检请求——*OPTIONS*，这个请求会确认服务器是否允许该请求，以避免对用户数据产生不良影响。  
在预检请求的时候，会带有一个`Access-Control-Request-Headers`的头字段，这个字段内容会包含了那些不包含在简单请求中头字段的键名。如果预检完成，才会继续发送真正的请求。比如浏览器返回预检内容为：
```html
Access-Control-Allow-Origin: http://example.com
Access-Control-Allow-Methods: POST,GET,OPTIONS
Access-Control-Allow-Headers: Content-Type,Custom-Type
Access-Control-Max-Age: 86400
```
同时`Access-Control-Max-Age`可以表示在一定时间内不需要为同一个请求发送预检请求。

## 附带身份凭证
**一般情况下，跨源的`XMLHttpRequest`和`Fetch`都不会发送凭证**，如果需要在XHR中发送凭证，则需要设定一个特殊的标识位。
```js
function call(){
    let xhr = new XMLHttpRequest;
    xhr.open('GET',url,true)
    xhr.withCredentials = true;
    xhr.send()
}
```
当然，这需要服务器返回的头内容中需要含有`Access-Control-Allow-Credentials:true`，否则不会返回内容。并且需要注意的是，如果需要允许携带Credentials，那么服务器的`Access-Control-Allow-Origin`不能为`*`，否则会直接失败。