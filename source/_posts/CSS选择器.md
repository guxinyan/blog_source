---
title: CSS选择器
date: 2016-04-27 01:36:43
tags:
	- CSS
	- 前端
categories:
	- 总结	
---

CSS选择器向来是每个前端菜鸟入职面试不得不直视的问题，是我们需要掌握的基础。在此对CSS选择器进行一下整理。

---
## 标签选择器
举例说明（下文将不赘述）：`p{color:blue}`
<!--more-->

---
## 类选择器
`.className{color:blue}`
选择器以**"."**开头，可在文档中出现**多次**，元素添加名为className的class属性。
className*命名规范*：由字母、数字、_组成，必须意义字母开头，区分大小写。

---
## ID选择器
`#idName{color:blue}`
选择器以**"#"**开头，在文档中**唯一**，元素添加名为idName的id属性。
idName*命名规范*：与类选择器类似。

---
## 通配符选择器
`*{paddding:0;margin:0}`
例子是CSS重置中常见的通配符选择器匹配样式，"*"匹配所有标签

---
## 属性选择器
*	**[att]**
`[disabled]{color:blue;}`
匹配带有disabled属性的元素
*	**[att=val]**
`[type="text"]{color:blue}`
匹配type属性为text的元素
*	**[att~=val]**
`[class~="sport"]{color:blue}`
等同于`.sport{color:blue}`
匹配含有sport类名的元素，例如`<div class="temp sport"></div>`
*	**[att|=val]**
`[lang|="en"]{color:blue}` 
匹配lang属性值为"en"或者以"en-"开头的元素
*	**[att^=val]**
`[href^="#"]{color:blue}`
匹配href属性以"#"开头的元素
*	**[att$=val]**
`[href$="pdf"]{color:blue}`
匹配href属性以"pdf"结尾的元素
*	**[att*=val]**
`[href*="163.com"]{color:blue}`
匹配href属性包含“163.com”字符串的元素
**注意与`[att~=val]`的区分**	

---
## 伪类选择器
### 动态伪类
动态伪类,不存在于HTML中，只有当用户和网站交互的时候才能一线出来。动态伪类包含两种，一种我们在链接中常见的锚点伪类，如“:link”、":visited";另一种被成为用户行为伪类,如“：hover”。
*	**锚点伪类**
```css
a:link{color:grey}	/*链接没有被访问时前景色为灰色*/
a:visited{color:yellow}	/*链接被访问过后前景色为黄色*/
a:hover{color:green}	/*鼠标悬浮在链接上时前景色为绿色*/
a:active{color:blue}	/*鼠标点中激活链接那一下前景色为蓝色*/
```
**锚点伪类的设置顺序必须如此！！！**
*	**用户行为伪类**
1.	:hover 用于当用户把鼠标移到元素上面时的效果
2.	:active 用于用户点击元素那一下的效果
3.	:focus	用于元素成为焦点，常用语表单元素

`对于:hover在IE6下只有a元素支持，:active只有IE7-6不支持，:focus在IE6-7下不被支持。`

### UI元素状态伪类
我们把":enabled",":disabled",":checked"伪类称为UI元素状态伪类，这些主要是针对于HTML中的Form元素操作，最常见的比如我们"type="text"有enable和disabled两种状态，前者为可写状态后者为不可状态；另外"type="radio"和"type="checkbox""有"checked"和"unchecked"两种状态。
`input[type="text"]:disabled{color:blue}` 
`IE6-8不支持":checked",":enabled",":disabled"这三种选择器。`

### CSS3的:nth选择器
*	:first-child
`li:first-child{color:blue}`匹配li的第一个子元素
*	:last-child
`li:last-child{color:blue}`匹配li的最后一个子元素
*	:nth-child()
:nth-child()可以选择某个的一个或多个特定的子元素，你可以按这种方式进行选择：
```css
	:nth-child(length);/*参数是具体数字*/
	:nth-child(n);/*参数是n,n从0开始计算*/
	:nth-child(n*length)/*n的倍数选择，n从0开始算*/
	:nth-child(n+length);/*选择大于length后面的元素*/
	:nth-child(-n+length)/*选择小于length前面的元素*/
	:nth-child(n*length+1);/*表示隔几选一*/
	//上面length为整数
```
`不能引用负值，也就是说li:nth-child(-3)是不正确的使用方法。`
`这里的“n”只能是"n"，不能使用其他字母代替`
nth-child(even)表示选中偶数行，nth-child(odd)表示选中奇数行
*	:nth-last-child()
与:nth-child()类似，不同的是，从最后一个元素开始算，来选择特定元素。
*	:nth-of-type()
类似于:nth-child，不同的是他只计算选择器中指定的那个元素
`.demo p:nth-of-type(even) {background-color: lime;}`
*	:nth-last-of-type()
*	:first-of-type和:last-of-type
*	:only-child和:only-of-type
":only-child"表示的是一个元素是它的父元素的唯一一个子元素。
`.demo .post p:only-child {background: red;}`p元素是.post下唯一一个子元素
:only-of-type是表示一个元素他有很多个子元素，而其中只有一个子元素是唯一的。
*	:empty
:empty是用来选择没有任何内容的元素，包括没有空格。
*	否定选择器（:not）
当然CSS3还新增了一些类似:target(),:lang()等选择器，不进行细说。

---
## 伪元素选择器
*	::first-line选择元素的第一行
*	::first-letter选择文本块的第一个字母
*	::before和::after
这两个主要用来给元素的前面或后面插入内容，这两个常用"content"配合使用，见过最多的就是清除浮动

```css
.clearfix:before,
.clearfix:after {
	content: ".";
	display: block;
	height: 0;
	visibility: hidden;
}
.clearfix:after {clear: both;}
.clearfix {zoom: 1;}
```
*	::selection用来改变浏览网页选中文的默认效果

---
## 后代选择器
`.main h2{color:blue}`
## 子代选择器
`.main>h2{color:blue}`
## 兄弟选择器
`h2+p{color:blue}`匹配紧跟h2的p元素
`h2~p{color:blue}`匹配在h2后的所有p元素

---
## 选择器优先级
*	计算方法
	a=行内样式     
	b=ID选择器的数量
	c=类、伪类和属性选择器的数量
	d=标签选择器和伪元素选择器的数量
	**权重value=a*1000+b*100+c*10+d**
<img src="/blogImg/selector.png" alt="示例图">

---
## CSS继承
*	什么是CSS继承？
*CSS样式表继承指的是特定的CSS属性向下传递到子孙元素。*
举个例子：
```html
<p>
	CSS样式表<em>继承特性</em>的演示代码	
</p>
```
当指定p的样式时：
```css
p{
	color:red;
}
```
p和em中的字都变为红色，我们并没有设定em的字体颜色，但em继承了它的父元素p的样式特性。这就是继承。

当然并不是所有的样式都可以继承。接下来我们来进行归纳。
*	可继承属性如下：
azimuth, border-collapse, border-spacing,
caption-side, color, cursor, direction, elevation,
empty-cells, font-family, font-size, font-style,
font-variant, font-weight, font, letter-spacing,
line-height, list-style-image, list-style-position,
list-style-type, list-style, orphans, pitch-range,
pitch, quotes, richness, speak-header, speaknumeral,
speak-punctuation, speak, speechrate,
stress, text-align, text-indent, texttransform,
visibility, voice-family, volume, whitespace,
widows, word-spacing

其中，文本属性相关：
 `font-family`, `font-size`, `font-style`,
`font-variant`, `font-weight`, `font`, 
`letter-spacing`,`line-height`, `text-align`,
 `text-indent`, `texttransform`,`word-spacing`
列表相关属性：
`list-style-image`, `list-style-position`,
`list-style-type`, `list-style`
还有一个比较重要的属性：
`color`

---
## CSS层叠
*	相同属性会覆盖
	优先级
	后面覆盖前面
*	不同属性会合并

---
## 改变CSS优先级
*	改变先后顺序
*	提升选择器优先级
如`.temp{}`改为`p.temp{}`
*	！import




