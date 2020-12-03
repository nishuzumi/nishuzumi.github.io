---
title: Link标签
date: 2020-11-29 10:41:49
tags:
catagories:
- [浏览器]
- [HTML]
- [前端]
---
# 前言 🎤
Link标签中，常用的方式就是用来倒入样式表，但是其实，Link的功能远不止这点。
<!--more-->
# ref=preload
当你写出
```html
<link ref='preload' href='some.js' as='script'>
```
浏览器的预加载器会在监测到该资源后，优先的进行内容的下载，并且不会被堵塞。只不过，同样的他也不会主动运行。并且，必须要填写`as`属性，否则浏览器会使用**较低**的优先级进行加载。  
当然不是所有内容都可以被预加载，一般情况下，以下内容可以被预加载。
- audio 音频文件
- document 一个需要被嵌入到`<iframe>||<frame>`中的HTML文档
- embed 一个被嵌入`<embed>`元素中的资源（很不常用，并且大部分浏览器已经弃用）
- fetch 需要通过fetch或者XHR获取的资源，比如ArrayBuffer或者JSON文件
- font 字体文件
- image 图片
- object 嵌入到`<embed>`元素中的文件
- script js文件
- style 样式表
- track WebVTT文件
- worker 一个js web worker
- video 视频文件

## 跨域
如果需要设定跨域，那么需要在link的标签上设定`crossorign`属性

# ref=prefetch
prefetch是一种页面预取技术，他允许浏览器在空闲时间下载用户将来不久会访问的文档，就好比在连续观看图片的时候，可以取到下一页的内容，让观看体验更加的流畅。虽然预取中也可以获取图片等内容，但是和`preload`等有不同的一点是，`prefetch`是在浏览器空闲时才会执行，而`preload`则是在页面加载时就开始预取。  
同样的，你可以使用`<link rel='next' href='2.html'>`来预载文档。

# media
你在link可以使用几乎和CSS中一样的媒体查询，这样浏览器就会自动判断并且按需加载。比如
```
<link rel='stylesheet' href='mobile.css' media='screen and (max-width:600px)'>
```
# crossorigin
用于指定是否使用CORS。可选值有两种
- anonymous 不带任何凭证
- use-credential 发送cookie和http基本认证信息。

# importance
相对重要性，默认为`auto`，可以设定为`high`或者`low`

# referrerpolicy
用于设定Referer标头
- no-referrer 不发送标头
- no-referrer-when-downgrade 当发生安全等级降级的情况下不发送（HTTPS->HTTP）。
- origin 表示该页面的源会成为标头内容
- origin-when-cross-origin 当同源是会发送完整URL，否则只发送源
- unsafe-url 无论是同源还是非同源，都发送完整URL