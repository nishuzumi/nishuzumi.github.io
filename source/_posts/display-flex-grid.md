---
title: 'display:flex grid && float'
date: 2020-11-24 18:45:36
tags:
catagories:
- [CSS]
- [前端]
---
## 前言 🎤
了解一下flex和grid两种布局手段
⚠️ 此文章内容还未完善
<!--more-->
## Flex
flex是一种一维的布局手段，他一次只能处理一行或者一列上的元素。甚至不能换行。

### 轴线
Flex有两根轴线，主轴和交叉轴。主轴由`flex-direaction`决定，而交叉轴则是垂直与他。flex中任何属性基本都和这两个轴有关，所以必须拿下。

#### 主轴
主轴由`flex-direaction`定义，可取`row,row-reverse,column,column-reverse`
其中`row*`是以`inline`方向为主轴，同时这也是默认的布局方式。
{% img /images/display-flex-grid-2020-11-24-19-16-02.png %}
如果选择了`column*`那么就是`block`方向为主轴，也就是垂直排列的方式。
{% img /images/display-flex-grid-2020-11-24-19-18-11.png %}

### Flex容器
flex容器中的元素有这些特点
- 元素拍成一排，按照`row-direaction`排列
- 从主轴起始线开始排列
- 不会进行拉伸，但是可以缩小（很关键）
- 在交叉轴方向上会被拉伸
- [`flex-basis`](#flex-basis)属性为*auto*
- [`flex-wrap`](#flex-wrap)属性为*nowrap*

{% iframe https://codesandbox.io/embed/gallant-einstein-5vrub?fontsize=14&hidenavigation=1&theme=dark %}

#### 改变方向
如果使用`row-reverse`或者`column-reverse`。那么起始方向将会是相反的方向。
{% img /images/display-flex-grid-2020-11-24-19-41-08.png %}

#### flex-wrap
当容器设定了固定宽度时，可能因为元素太大导致无法显示完全。这个时候会默认产生缩小flex元素。如果希望不让元素缩小，那么就需要进行设定`flex-wrap:wrap`，这样就会让元素自动换行。如果不愿意换行，元素又实在很大，那么就会发生溢出情况。  
{% img /images/display-flex-grid-2020-11-24-19-49-10.png %}

### 简写属性 flex-flow
你可以将`flex-direction`和`flex-wrap`简写为`flex-flow`，第一个顺位参数为`direaction`，第二个为`wrap`。

### flex元素上当其他控制属性
为了更好的控制flex，flex还有三个额外属性：
- flex-grow
- flex-shrink
- flex-basis

**可用空间**  
可用空间是指一行中，还剩下多少空间可以使用。
{% img /images/display-flex-grid-2020-11-24-19-53-37.png %}

#### flex-basis 元素空间大小
这个属性是用来定义元素空间大小的，默认情况下，这个值是`auto`，而且是最小值。如果设定为`100px`，那么每个元素都会被设定宽度。注意，**这个属性是放置于flex元素上，而不是flex容器上**。

#### flex-grow 元素可用空间增长系数
flex-grow如果被设定为一个正数，那么就会以`flex-basis`为基础，沿着主轴增长尺寸，这会让元素伸展，并且占据可用空间。如果给每个元素设定`flex-grow:1`，那么每个元素就会平分剩下的元素空间。比如，如果剩余空间为400px,而三个元素中设定的`flex-grow`为`1,2,1`那么就会按照`100,200,100`进行分配。

#### flex-shrink 元素收缩处理
收缩处理和grow的规则并不相同，反而是有自己的想法。如果说将`flex-shrink:0`，那么就不会允许收缩，因而让他们溢出。
{% img /images/display-flex-grid-2020-11-24-20-26-58.png %}
如果为1，那么他们会同比例的缩减大小，但是无论如何都不会缩减成为0。

### flex简写
`flex: grow shrink basis`

### 元素对齐
#### align-items
这个属性可以让元素在交叉轴方向对齐。默认值为`align-items:streth`，即拉伸。  
他有四个属性值
- stretch（拉伸）
- flex-start （起始线对齐）
- flex-end （结束线对齐）
- center （中心线对齐）
{% img /images/display-flex-grid-2020-11-24-21-17-09.png %}
#### justify-content
这个属性是用来定义主轴上的对齐，默认值为`stretch`。可用值为
- stretch （默认，和起始线一样）
- flex-start （起始线对齐）
- flex-end （结束线对齐）
- center （居中）
- space-around （周围空隙）
- space-between （之间空隙）

{% img /images/display-flex-grid-2020-11-24-21-24-20.png %}

#### align-content
用于决定交叉轴的对齐方式。相比justify多了几种属性。
- 基线对齐方式
- 分布式对齐方式
- 溢出对齐方式

//todo: 此部分有待完善
https://developer.mozilla.org/zh-CN/docs/Web/CSS/align-content
### order 元素顺序
这个元素可以设定元素的排列顺序，能直接修改元素的出现位置

### visibility: collapse
这个属性用于将flex元素从布局上进行渲染移除。但是会保持一个尺寸控制。比如某个元素的高度被撑到400px，即使其他元素的`max-content`只有100px，在折叠的情况下，依然会让他们保持400px的高度。  
但是这个属性在chrome中表现不是很好，在chrome中，`collapse`只会出现元素隐藏的效果——即无法看到元素，但是可以看到隐藏元素留下的空白，而在firefox中，元素会从布局中移除，并且保留高度支撑。  
chrome中的效果：
{% img /images/display-flex-grid-2020-11-25-09-57-46.png %}  
firefox效果：  
{% img /images/display-flex-grid-2020-11-25-09-58-05.png %}

### visibility:hidden和display:none区别
`display:none`会从文档流中删除。  
`visibility:hidden`会保持文档布局，但是看不到元素。  

## grid
grid是css的网格布局，最大的特点它是一个二维的布局方式。最早的网格系统可以参考css框架`bootstrap`，在浏览器没有提供原生支持之前，网格系统都是通过一些其他方式进行定义。我敢保证，grid是目前最强大的布局方式。  
一个网格系统首先就是**行（row）**和**列（column）**，还有他们之间的空隙，一般称之为**沟槽（gutter）**
{% img /images/display-flex-grid-2020-11-25-10-45-55.png %}

### grid-template-columns
正如其名，是用来控制*列*的宽度设定，同时也设定了其中列的数量，有多少参数就有多少个列。例如：
```css
.container{
    display:grid;
    grid-template-columns:200px 200px 200px;
    //or
    grid-template-columns:repeat(3,200px);
}
```
{% img /images/display-flex-grid-2020-11-25-11-18-24.png %}  
#### fr
在使用百分比进行布局的时候，会出现一些以外情况。比如，当你设定了gap距离的时候，并且overflow设定为scroll，那么就会出现滚动条。或者出现溢出的情况。当然，你可以在布局中强制减去这些gap距离，显然这种方法就非常的不优雅。
而fr可以称之为free space。你可以认为它是用于表示当元素计算完成`max-content`和`gap`，还有其他各种内容后，实际上剩余的空间。
因此，可以通过设定`grid-template-column:repeat(4,1fr)`。这样他们就会均分所有的剩余空间。你可以认为这是一个权重，所有fr总和为总权重，然后使用当前fr去处以总权重，就表示可分到的百分比。例如：  
1fr 1fr 1fr fr = 25% 25% 25% 25%  
2fr 1fr 1fr = 50% 25% 25%

### 间隙
网格间隙分为两种`grid-column-gap`列间隙和`grid-row-gap`行间隙。当然你可以使用`grid-gap`来同时设定两个间隙，只不过，不允许使用fr单位表示，其他的都可以。  
在要说一点的是，现在以`grid`开头的间距设定都处于不推荐状态，因为目前前端有三种布局方式都含有gap属性`multi-column`，`flex`，`grid`，因为功能相似，所以为了往规范靠近，现在可以使用不带任何头的`gap`来进行定义。同样的这三种布局都可以完美兼容，不过flex的兼容性可能不太好。

### repeat
repeat和传统的函数不一样，你可以简单的认为他是宏脚本。因为你可以这样使用repeat,`repeat(2,1fr 2fr) == 1fr 2fr 1fr 2fr`。

### 显式网格和隐式网格
显式网格是我们主动定义`grid-template-rows`或者`grid-temeplate-columns`而创建的。而隐式网格则是当有内容放入时才会生成。
在隐式网格中，行列的大小默认`auto`，会根据放入的内容自行调整。当然你可以使用`grid-auto-rows`或者`grid-auto-column`进行手动设定隐式网格大小。  
**总结来说，当显式网格没有定义的元素，都会使用隐式网格的定义来处理**

### minmax()函数
`minmax(min,max)`当某些情况，可能元素内容大小超出了一定高度，就会不够用，那么我们可以设定最小和最大。比如`grid-auto-rows:minmax(100px,auto)`，就可以让最大自适应，最小100px。

### 自动多列填充
你可以使用一些新功能来完成自适应的效果，比如`grid-template-columns:repeat(auto-fill,minmax(200px,1fr))`。这样设定的话，就会出现非常神奇的效果，大致描述为*元素单个大小最小为200px，当页面缩小到无法容下时，就会自动的扩充元素大小，并且减少列数。
{% img /images/display-flex-grid-2020-11-25-14.08.26.gif %}
很明显的是，每一个元素的大小都大于200px，符合预期。

### 基于线的元素放置
定义完成网格后，我们可以设定相关的线来对齐进行更为详细的布局。不过这个线的起始结束和`flex`一样，和文档书写有关，一般情况左到右，但是比如阿拉伯语时，右到左。  
你可以使用以下属性控制元素的位置：
- grid-column-start
- grid-column-end
- grid-row-start
- grid-row-end
当然一般情况下都会使用他们的简写形势
- grid-column
- grid-row
其中使用`/`进行分割。
{% img /images/display-flex-grid-2020-11-25-14-37-47.png %}  
其实就是画格子，非常的简洁。

### 强大的命名 grid-template-areas
如果使用线的元素放置，又时候你可能会完全不知道到底做成了什么样子，在复杂的布局中更能体现，所以，基于别名的区域放置功能就解决了这个问题。  
```css
//首先是定义区域名称
.header{
    grid-area: header;
}
article {
  grid-area: content;
}
.container{
    grid-template-areas:
            "header header"
            "article article";
    grid-template-columns: repeat(2,1fr)
}
```
基本来说，有多少行，有多少列，就需要有多少个字符串出现在`grid-template-areas`。这本质上还是画格子，但是能更好的看清楚具体位置和效果。  
不过，还是有几个注意的点。
- 你需要填满每个格子
- 重复写上区域能让区域位置变多
- 不能画非矩形区域
- 可以使用`.`让格子留空

## float
float可以让一个元素往某个方向冲锋，直到碰到外层壁或者上一个float元素为止。同时这个元素会从文档流中移除，但是保留部分流动性。
一般情况下我们只会用到三种：
- left
- right
- none
当然还有随着文档语言修改方向的
- inline-start
- inline-end

### 清除浮动 clear
如果你不希望让某些元素处于浮动的高度之中，你可以使用`clear`来决定元素的位置将会处于什么状况。如果你使用`clear:left`，那么你的元素就会在具有`float:left`元素的下方。
clear的可选参数和float基本一致，但是多出`both`值，`both`表示左右均不允许有浮动元素存在。