---
title: CSS小结 优先级 变量
date: 2020-11-25 15:31:53
tags:
---
# 前言 🎤
本文说说CSS中选择器优先级和一些不常见的使用方法
<!--more-->

# 优先级
优先级其实就是给CSS一个权重，由多种因素决定
## 选择器类型
一般来说，越能具体表现某个元素的选择器，优先级越高。以下元素的权重由低到高。
- 类型选择器，伪元素 （div,::before）
- 类选择器，属性选择器，伪类 （.example,[type='radio'],:hover）
- ID选择器 （#example）

通配符（*），关系选择（+，>，～，‘ ’，||）,否定伪类（:not()）这些对优先级没有影响。

### 关系选择
- '+' 相邻兄弟们选择器 用于选择紧紧挨着的兄弟元素
- '>' 子元素选择器，用于选择第一个子后代
- '~' 通用兄弟家选择器，选择同层级的兄弟元素
- '||' （实验性功能）表示没看懂什么意思

## !important 例外规则
当你在一个样式里面使用`!important`的时候，你可以认为此时你的样式声明已经到了另外一个层次，这个层次会比所有的常规声明都要优先。但是，其他的，使用了`!important`的样式声明，一样会加入这个层次，然后继续按照一般的优先级继续进行排序。

## :is()和:not()
实际上 这两个选择器不会被当成伪类，而是会当成普通的选择器进行计数。
## 形式优先级
```css
*#foo {
  color: green;
}

*[id="foo"] {
  color: purple;
}
```
在这种情况下，即使属性选择器中使用的是id，但是它一依然是属性选择器。因此id选择优先级更高。

## 无视DOM树距离
在同一个层级的选择器中，越靠下的声明优先级越高，并且无视DOM树距离。
```css
body h1 {
  color: green;
}

html h1 {
  color: purple;
}
```
HTML:
```html
<html>
  <body>
    <h1>Here is a title!</h1>
  </body>
</html>
```

## 直接添加 继承样式
为目标直接添加，和继承的优先级相比，直接添加的优先级更高。
# 伪选择器
## :root
这个可以选择文档的根元素，同时优先级更高，其他的和`html`选择器相同。
一般情况下，可以用于声明全局变量：
```css
:root{
    --global-height:100vh;
}
```
## :target
用于设定URI中#any转跳时显示的css

# 自定义属性
你可以在CSS文档中使用自定义属性，一般以`--`开头。该属性具有继承性，并且大小写敏感。如果需要使用，则需要用`var()`进行包裹即可使用。
备选值：`var(--my-var,var(--my-backgroun,pink))`
无效值：当使用无效值，则会自动忽略。

# 奇妙的position
## fixed
众所周知，而且几乎所有人都认为，fixed是基于视窗定位的。也确实，大部分情况下确实如此，但是有一种情况，恰恰相反。  
当某个元素当祖先元素之中，其属性中含有`transform`,`perspective`,`filter`的时候，元素会相对于该祖先元素进行定位，而不是`viewport`。

# attr
一个理论上能支持任何的CSS属性但是目前只能支持Content属性的超级高级css功能。

```css
div{
  content:attr(data-content);
}
```