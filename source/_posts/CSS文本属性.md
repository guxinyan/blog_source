---
title: CSS文本属性
date: 2016-05-08 10:42:20
tags:
	- CSS
	- 前端
categories:
	- 总结
---

CSS文本属性可定义文本的外观，可以改变文字的颜色、字符间距，对齐文本，对文本缩进等等。**文中所总结的属性以及属性值大都为常用属性及其属性值，并不完全(包括语法规则)**。

---
# 字体
## font-size
可设置字体的尺寸。语法规则为：`font-size:<length> | <percentage>`
<!--more -->
例如：
`font-size:20px`
`font-size:2em`  参照物为父元素
`font-size:200%` 参照物为父元素
`font-size:2rem` 参照物为根元素< html >	

## font-famliy
规定元素的字体系列。语法规则为：`font-famliy:[<famliy-name> | <generic-family>]#`
*	**famliy-name**:
	指定的系列名称：具体*字体*的名称，比如："times"、"courier"、"arial"。
*	**generic-family**	
	通常*字体系列*名称：比如："serif"、"sans-serif"、"cursive"、"fantasy"、"monospace"。
	在不同的浏览器上相同的generic-family的表现是不一样的,取决于用户安装了的字体。

## font-weight
设置文本的粗细。语法规则为:`font-weight:normal|bold|bolder|lighter|100|200|300|400|500|600|700|800|900`
## font-style
定义字体的风格。语法规则为：`font-style:normal|italic|oblique`
*	**italic**:斜体
*	**oblique**:倾斜。对于没有斜体的字体应该使用oblique属性值来实现倾斜的文字效果。

## line-height
设置行间的距离（行高）。语法规则为:`line-height:normal|<number>|<length>|<percentage>`
例如：
`line-height:40px`
`line-height:3em`
`line-height:300%`
`line-height:3`	
**父元素的行高为300%或3em时，会根据父元素的字体大小先计算出行高值然后再让子元素继承，而3是根据子元素的字体大小动态计算出行高值让子元素继承。**

## font
语法规则为：`font:[ [<font-style>||<font-weight>]? <font-size> [/<line-height>]? <font-family> ]`
缩写时font-size和font-family为必填属性值。
例如：
`font:30px/2 'Consolas',monospace`
`font:italic bold 20px/1.5 arial,serif`

## color
例如：
`color:red`
`color:#ff0000`
`color:rgb(255,0,0)`
`color:rgba(255,0,0,1)`
`color:transparnt`

---
# 对齐方式
## text-align
规定元素中的文本的水平对齐方式。语法规则为：`text-align:left|right|center|justify`
*	**justify**:两端对齐

## vertical-align
设置元素的垂直对齐方式。语法规则为:`vertical-align:baseline|sub|super|top|text-top|middle|bottom|text-bottom|<percentage>|<length>`
*	`<percentage>`的参照物为`line-height`

只有一个元素属于inline或是inline-block（table-cell也可以理解为inline-block水平）水平，其身上的vertical-align属性才会起作用。
接下来我们将以示例来进行具体说明：
html代码：
```html
<div class="demo">
    	这是测试<span>asdasdas</span>测试测试测试测试
</div>
```
*	vertical-align:baseline
<img src="/blogImg/baseline.png" alt="baseline示例图">
*	vertical-align: sub
<img src="/blogImg/sub.png" alt="sub示例图">
*	vertical-align:super
<img src="/blogImg/super.png" alt="super示例图">
*	vertical-align:top
<img src="/blogImg/top.png" alt="top示例图">
*	vertical-align:text-top
<img src="/blogImg/text_top.png" alt="text-top示例图">
*	vertical-align:middle
<img src="/blogImg/middle.png" alt="middle示例图">
*	vertical-align:bottom
<img src="/blogImg/bottom.png" alt="bottom示例图">
*	vertical-align:text-bottom
<img src="/blogImg/text_bottom.png" alt="text-bottom示例图">

---
# 格式处理
## text-indent
规t定文本块中首行文本的缩进。语法规则为：`text-indent:<length>|<percentage>`
例如：
`text-indent:2em`参照物为字体大小
`text-indent:10px`
`text-indent:20%`参照物为容器大小
*	常用`text-indent:-9999px`达到隐藏文字的目的。用于SEO

## white-space
设置如何处理元素内的空白。语法规则为：`white-space:normal|nowrap|pre|pre-wrap|pre-line`
<img src="/blogImg/white_space.png" alt="white-space行为">

## word-wrap
允许长单词或 URL 地址换行到下一行。语法规则为：`word-wrap:normal|break-word`
这里我们将以示例说明。
HTML代码
```html
<div class="demo">
    	test sagfzxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
</div>
```
*	正常情况下
<img src="/blogImg/normal.png" alt="正常情况示例图">浏览器判断第二个单词容器放不下时会自动换行。
*	设置了word-wrap:break-word
<img src="/blogImg/break_word.png" alt="break-word示例图">在正常情况的基础上切断单词。

## word-break
规定自动换行的处理方法。语法规则为：`word-break:normal|keep-all|break-all`
紧接上例。
*	设置了word-break:break-all
<img src="/blogImg/break_all.png" alt="break-all示例图">不同的是，长单词会自动跟进test后，不换行。

注意体会word-wrap:break-word与word-break:break-all区别。后者更节约空间。

## text-transform
控制文本的大小写。语法规则为：`text-transform:none|capitalize|uppercase|lowercase`
*	**capitalize**：文本中的每个单词以大写字母开头

---
# 文本修饰
## text-shadow
向文本设置阴影。语法规则为:`text-shadow:none | [ <length>{2,3} && <color>? ]#`
例如：
`text-shadow:1px 2px 3px #f00` 1px代表x轴阴影的位置，2px代表y轴阴影的位置，3px代表模糊的距离。颜色不写默认为文字颜色。

## text-decoration
语法规则为：`text-decoration:none | [underline||overline||line-through]`
例如：
`text-decoration:underline overline`

---
# 高级设置
## text-overflow
语法规则为：`text-overflow:clip|ellipsis`
**常用**：单行文字过长末尾出现……
```css
text-overflow:ellipsis;
overflow:hidden;
white-space:nowrap;
```

## cursor
规定要显示的光标的类型（形状）。语法规则为：`cursor:[<uri>,]* [auto|default|none|help|pointer|zoom-in|zoom-out|move]`
*	`<uri>`：自定义光标类型
*	help:问号
*	zoom-in:放大镜
*	zoom-out:缩小镜

例如：
`cursor:url(xx.cur),pointer`在图片无法加载的情况下采用鼠标形状。

## inherit
强制继承。
CSS中的每个属性都有一个特定值"inherit",其含义是指定继承父元素的相应属性，使用inherit一方面在代码上能地表明要继承于父元素的样式属性，另一方面也使子元素继承了那些不会被自动继承的属性。