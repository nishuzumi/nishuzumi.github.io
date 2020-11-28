---
title: 纯CSS实现Tip提示
date: 2020-11-28 15:17:04
tags:
---
# 前言 🎤
面试的时候面试官问到这个问题，回答的不太满意。
<!--more-->
# 实现
## 要求
内容在链接上方，圆角，有个小箭头指向链接，位置和内容左对齐。类似百度百科的效果。  
{% img /images/css-tip-2020-11-28-15-20-18.png %}
实现后的效果图：
{% img /images/css-tip-2020-11-28-15-24-34.png %}

## 代码
HTML:
```html
<div class='tip' href="#" data-target="我是一个小小的描述我是一个小小的描述我是一个小小的描述我是一个小小的描述我是一个小小的描述我是一个小小的描述我是一个小小的描述我是一个小小的描述我是一个小小的描述">
  <a href='#'>描述</a>
</div>
```
CSS:
```css
.tip:hover::after{
  content: ' ';
  width:10px;
  height:10px;
  transform:rotate(45deg);
  position: absolute;
  border:1px solid;
  bottom: 30px;
  border-left:none;
  border-top:none;
  background-color:white;
}
.tip:hover::before{
  content: attr(data-target);
  position:absolute;
  border:1px solid;
  border-radius:5px;
  bottom:35px;
  width:200px;
  overflow:hidden;
  background-color:white;
  box-sizing:border-box;
  padding:10px;
}
.tip:hover{
  position:relative;
}
```

## 解析
```css
.tip:hover{
  position:relative;
}
```
由于`position:absolute`是根据上一个非`static`定位的元素。所以需要设定`.tip`元素为`position:relative`，这个设定并没有固定的要求，可以在hover里面设定，也可以直接设定在`.tip`上。

```css
// 简化 去除外观部分 只保留核心内容
.tip:hover::before{
  content: attr(data-target);
  position:absolute;
  bottom:35px;
}
```
设置内容为属性内容，定位为绝对定位，并且设定和最下方之间的距离。

```css
.tip:hover::after{
  content: ' ';
  width:10px;
  height:10px;
  transform:rotate(45deg);
  position: absolute;
  border:1px solid;
  bottom: 30px;
  border-left:none;
  border-top:none;
  background-color:white;
}
```
这个部分的内容是用于绘制三角形的内容，通过绘制一个正方形，然后不绘制左边和上面的边，再旋转元素，并且使用一样的位移手段。

## JSFiddle
{% iframe //jsfiddle.net/BoxChen/L8r3mpnz/1/embedded/html,css,result/dark/ %}