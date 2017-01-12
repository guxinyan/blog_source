---
title: CSS盒模型
date: 2016-05-09 15:49:51
tags:
	- CSS
	- 前端
categories:
	- 总结
---

盒模型是CSS中一个重要概念，理解了盒模型我们才能更好地对页面进行排版。什么是盒模型呢？接下来，我们将对盒模型进行具体讲解。

---
# 标准盒模型和IE盒模型
其实盒模型有两种，一种是W3C标准盒模型，一种是IE盒模型，它们对盒子模型的解释各不相同。接下来，我们先看标准盒模型。
<!--more-->
<img src="/blogImg/标准盒模型.png" alt="标准盒模型">
border、margin、padding都有top、right、bottom、left四部分
`content=width*height`

再来看IE盒模型：
<img src="/blogImg/IE盒模型.png" alt="标IE盒模型">
很明显，IE盒模型和标准盒模型的不同就在于，IE盒模型的content部分包括padding和border。

例如：
一个盒子的margin为20px,paddding为10px,border为5px,content的宽为200px,高为50px。
标准盒模型占据的位置：宽=20px*2+5px*2+10px*2+200px=270px,高=20px*2+5px*2+10px*2+50px=120px
标准盒模型实际大小为：宽=5px*2+10px*2+200px=230px,高=5px*2+10px*2+50px=80px
IE盒模型占据的位置：宽=20px*2+200px=240px,高=20px*2+50px=90px
IE盒模型实际大小为：宽=200px,高=50px

**在网页首部加上DOCTYPE声明告诉浏览器用标准盒模型去解释盒子**

---
# width
语法规则为:`width:<length>|<percentage>|auto|inherit`
*	`<percentage>`的参照物为父元素
*	行内元素无法设置宽度
*	max-width
*	min-width

---
# height
语法规则为：`height:<length>|<percentage>|auto|inherit`
*	`<percentage>`的参照物为父元素
*	行内元素无法设置高度
*	max-height
*	min-height

---
# padding
语法规则为：`padding:[<length>|<percentage>]{1,4}|inherit`
例如：
`padding:20px`=> `padding:20px 20px 20px 20px`
`padding:40px 30px 20px 10px`=>`padding-top:40px`、`padding-right:30px`、`padding-bottom:20px`、`padding-left:10px`	**方向顺时针旋转。遵守TRBL原则**
`padding:20px 10px`=>`padding:20px 10px 20px 10px`
`padding:20px 10px 30px`=>`padding:20px 10px 30px 10px`
缩写原则：**对面相等，后者省略;四面相等，只设一个** 

---
# margin
语法规则为：`margin:[<length>|<percentage>|auto]{1,4}|inherit`
很明显，margin比padding多了一个属性值auto,浏览器会自动分配空间。
margin具体缩写原则和padding一样。

## 垂直外边距合并
1.**毗邻元素外边距合并（取大）**
将以例子说明。
html代码：
```html
<div class="demo1"></div>
<div class="demo2"></div>
```
css代码
```css
 .demo1{
	 width:200px;
	 height: 100px;
	 background: #000;
	 margin-bottom:50px;
}

.demo2{
	width:200px;
	height: 100px;
	background: #f00;
	margin-top: 20px;
}
```
最终效果图：
<img src="/blogImg/margin合并1.png" alt="margin合并示例图">

2.父元素与第一个/最后一个子元素外边距合并
以例子说明：
html代码：
```html
<div class="demo"></div>
<div class="demo1">
	<div class="demo2"></div>
</div>
```
css代码：
```css
.demo{
	width:400px;
	height: 200px;
	background: #000;
}
.demo1{
	width:400px;
	height: 200px;
	background: #000;
	margin-top:50px;
}
.demo2{
	width:200px;
	height: 100px;
	background: #f00;
	margin-top: 20px;
}
```
最终效果图：
<img src="/blogImg/margin合并2.png" alt="margin合并示例图">
*如果本意子元素添加margin-top是为了和父元素隔开距离，可以给父元素添加border-top或者padding-top。*

## 水平居中
margin:0 auto;

---
# border
语法规则为：`border:[<border-width>||<border-style>||<border-color>]|inherit`
*	`border-width:[<length>|thin|medium|thick]{1,4}|inherit`
*	`border-style:[solid|dashed|dotted|...]{1,4}|inherit`
*	`border-color:[<color>|transparent]{1,4}|inherit`

例如：
`border:1px dashed blue`
`border:1px solid` 颜色不填写默认为字体颜色

---
# border-radius
语法规则为：`border-radius:[<length>|<percentage>]{1,4}[/[<length|<percentage>]{1,4}]?`
例如：
`border-radius:10px`
`border-radius: 10px 20px 30px 40px/40px 30px 20px 10px`效果如下图：
<img src="/blogImg/border_radious.png" alt="border-radius示例图">
我们可以看出border-radius四个参数分别代表左上、右上、右下、左下,`/`前面表示水平半径，`/`后面表示垂直半径
`border-radius：50%`圆形
`border-top-left-radius:10px`

**IE8及以下不支持**

---
# overflow
语法规则为：`overflow:visible|hidden|scroll|auto`
*	overflow-x
*	overflow-y

---
# box-sizing
设置width、height制定区域。语法规则为：`box-sizing:content-box|border-box|inherit`
*	border-box：宽度和高度包括padding和border
*	IE7及以下不支持

---
# box-shadow
语法规则为:`box-shadow:none|<shadow>[,<shadow>]*`
*	`shadow:inset? && <length>{2,4} && <color>?`
*	IE8及以下不支持

例如：
`box-shadow:4px 6px 3px 0px red;` 4px指水平偏移，6px指垂直偏移,3px指模糊半径,0px指阴影大小，颜色不填默认为字体颜色
`box-shadow:inset 0px 0px 5px red`内阴影
`box-shadow:3px 3px 5px 2px,inset 0px 0px 5px red`多阴影

<font color="red">\*黑魔法：利用box-shadow做多重边框</font>
```css
width:50px;
height: 50px;
background: yellowgreen;
box-shadow: 0 0 0 10px #655,
            0 0 0 20px deeppink,
            0 0 0 30px yellow;
```
<img src="/blogImg/border_by_box_shadow.png" alt="多重边框">
同时，也可以在这层边框外面加一层常规的投影
```css
box-shadow: 0 0 0 10px #655,
            0 0 0 20px deeppink,
            0 2px 5px 30px rgba(0,0,0,.5);
```
但是，投影不影响css布局，如果我们需要多层边框也占“位置”的话，可以给box-shadow添加inset属性。
<font color="red">*注：*box-shadow无法做虚线边框，</font>这时我们可以采用outline，具体在下方阐述。

---
# outline
outline （轮廓）是绘制于元素周围的一条线，位于边框边缘的外围，可起到突出元素的作用。
语法规则为：`outline:[<outline-width>||<outline-style>||<outline-color>]|inherit`
*	`outline-width:<length>|thin|medium|thick|inherit`
*	`outline-style:solid|dashed|dotted|...|inherit`
*	`outline-color:<color>|invert|inherit`invert指和背景色相反
*	不占空间
*	在border外

例如：
`outline:5px dashed blue`

<font color="red">\*黑魔法：利用outline做多重边框</font>
```css
width:50px;
height: 50px;
background: yellowgreen;
outline: 10px solid deeppink;
```
通过outline-offset可以实现缝边效果
```css
width:50px;
height: 50px;
background: yellowgreen;
outline: 1px dashed white;
outline-offset: -5px;
```
<img src="/blogImg/dashedBorder_by_outline.png" alt="缝边效果示例图">
<font color="red">注：outline只能实现双层边框, 边框不能贴合border-radius属性产生的圆角，</font>而box-shadow能贴合圆角

