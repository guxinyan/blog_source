---
title: HTML标签
date: 2016-04-25 18:44:03
tags:
		- HTML
		- 前端
categories:
		- 总结		
---

在代码中经常用会各种HTML标签，每个标签有不同的语义化特点和属性，本篇对**常用标签**进行了整理。

---
## before

在具体介绍标签之前，我们先对标签有一个大概的了解。

对标签的书写我们要有一定的**规范要求**：*小写*、*属性值双引号*、*嵌套缩进* 。

同时我们介绍一些标签的**常用属性**：
<!--more-->
1. id
	`<div id="ext"></div>`					id唯一			
2. class	
	`<span class="style1"></span>`			class可多个，一般用于样式
3. style
	`<div style="display:none"></div>`		一般不建议使用，不利于样式覆盖
4. title
 	`<a title="喜欢"></a>`			可用于显示不完全的标签

---
## 文档声明<!DOCTYPE>

<!DOCTYPE>声明必须是HTML的第一行，具体而言并不是HTML标签，**告诉浏览器用哪个标签解析文档**。
在HTML4.01中有三种<!DOCTYPE>声明：
*	**HTML 4.01 Strict**
	该 DTD 包含所有 HTML 元素和属性，但不包括展示性的和弃用的元素（比如 font）。不允许框架集（Framesets）。
	`<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">`
*	**HTML 4.01 Transitional**
	该 DTD 包含所有 HTML 元素和属性，包括展示性的和弃用的元素（比如 font）。不允许框架集（Framesets）。
	`<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">`
*	**HTML 4.01 Frameset**
	该 DTD 等同于 HTML 4.01 Transitional，但允许框架集内容。
	`<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Frameset//EN" "http://www.w3.org/TR/html4/frameset.dtd">`

在HTML5中只有一种<!DOCTYPE>声明：
	`<!DOCTYPE html>`

---
## 头部标签
*	**head**
	定义文档头部，是所有头部标签的容器
*	**title**
	是head标签中唯一必须包含的标签，定义标题
*	**meta**
	在这里不作具体描述，以几个常用示例做描述
	`<meta charset="utf-8"></meta>`
	`<meta name="keywords" content="HTML,ASP,PHP,SQL">`
	`<meta http-equiv="expires" content="31 Dec 2008">`
	`<meta name="viewport" content="width=device-width, initial-scale=1.0">`
*	**base**
	为页面上的所有链接规定默认地址或默认目标。属性值为href、target
	`<base href="http://www.w3school.com.cn/i/" />`
	`<base target="_blank" />`
*	**link**
	link标签可引入外部css样式，也可引入网页icon
	`<link rel="shortcut icon" href="http://www.mysite.com/myicon.ico">。`
*	**style**
	内部css样式
*	**script**
	引入js
*	**noscript**
	用来定义在脚本未被执行时的替代内容（文本）。

---
## 文档章节相关标签
*	**body标签**
*	**HTML5新增**
	<img src="/blogImg/1.png" alt="">
*	**标题标签**
	h1~h6

---
## 文本标签
`<a></a>`:创建链接|创建锚点|mailto
`<em></em>`、`<strong></strong>`:做强调
`<span></span>`
`<br>`
`<cite></cite>`、`<q></q`>：引用
`<code></code>`：代码
`<b></b>`、`<i></i>`格式化

---
## 组合标签
`<div></div>`
`<p></p>`
列表标签：无序`<ul></ul>`、有序`<ol></ol>`、自定义`<dl></dl>`
`<pre></pre>`：已格式化的内容
`<blockquote cite=""></blockquote>`：组合引用

---
## 表格标签

	```HTML
	<table>
		<caption>标题</caption>
		<thead>
			<tr>
				<th colspan="2"></th>
				<th></th>
			</tr>
		</thead>
		<tbody>
			<tr>
				<td rowspan="2"></td>
				<td></td>
			</tr>
		</tbody>
		<tfoot></tfoot>
	</table>
	```
---
## 表单标签
*	**form**
	用于创建html表单。
*	**fieldset**
	可将表单内容打包成一组。`<legend></legend>`为fieldset设定标题
*	**input**
	属性`type`规定元素的类型：`button`、`checkbox`、`file`、`hidden`、`image`、`password`、`radio`、`reset`、`submit`、`text`(默认)。
	HTML5新增了一些type类型：`number`、`email`、`url`、`color`、`seach`等等。
	`readonly`只读属性。
	`placeholder`规定帮助用户填写输入字段的提示。
*	**select**
	创建多选菜单。`<option>`定义菜单中的可用选项。
	属性`disabled`禁止下拉菜单，`selected`默认选定
*	**textarea**
	定义多行的文本输入控件。
*	**label**
	 `for`属性应当与相关元素的 id 属性相同。

**以下是表单综合示例**	 

```HTML
<form action=""  method="post" class="m-form">
	<fieldset>
		<legend>照片选择</legend>
		<label for="file">选择照片</label><input type="file" id="file">
	</fieldset>
	<fieldset>
		<legend>综合设置</legend>
		<div>选择尺寸：</div>
		<div>
			<input  type="checkbox" name="size" id="cb_0" value="5"><label for="cb_0" checked>5寸</label>
			<input  type="checkbox" name="size" id="cb_1" value="6"><label for="cb_1">6寸</label>
		</div>
		<div>选择相纸：</div>
		<div>
			<input type="radio" name="material" id="rd_0" value="fushi"><label for="rd_0" >富士</label>
			<input type="radio" name="material" id="rd_1" value="keda"><label for="rd_1" >柯达</label>
		</div>
		<div>
			<label for="delivery">配送方式：</label>
			<select id="delivery">
				<option value="0">快递</option>
				<option value="1">EMS</option>
				<option value="2" selected>平邮</option>
			</select>
		</div>
		<div>
			<label for="description">商品描述：</label>
			<input type="text" id="description" placeholder="描述">
		</div>
		<div>
			<label for="feedback">意见反馈：</label>
			<textarea name="feedback" rows="4" id="feedback"></textarea>
		</div>
	</fieldset>
	<div>
		<button type="submit">提交</button>
		<button type="reset">重置</button>
	</div>
</form>
```
*以上代码效果如下图所示：*
<img src="/blogImg/form.png" alt="表单代码效果">

---
## 资源标签
*	**img**
	`<img src=""  alt="标题" />`
	写alt属性提高用户体验
*	**iframe**
	iframe 元素会创建包含另外一个文档的内联框架（即行内框架）。嵌入页面与当前页面隔离。
	`<iframe src=""></iframe>` 
*	**object**、**embed**：引入外部插件
```html
	<object type="application/x-shockwavr-flash">
		<param name="movie" value="http://xxx.swf">
		<param name="flashvars" value="http://xxx.pdf">
	</object>
```
```html
	<embed src="http://xxx.pdf" type="application/x-shockwavr-flash" width="640" height="480">
```
*	**video**、**audio**：音视频
```html
<video autoplay loop controls poster=""  >
    <source src="./res/video.mp4" type="video/mp4">
    <track kind="subtitles" src="./res/video.vtt" srclang="cn" label="cn">
    您的浏览器不支持video
</video>
```
`controls`控制条
`poster`视频封面
`track`引入视频字幕
`autoplay`自动播放
`loop`循环播放
audio与video类似

*	**canvas**、**svg**：图
	`canvas`基于像素，提供一些绘制函数，利用脚步绘制图形图像。性能要求高，场景比较复杂通过canvas实现
	`svg`矢量的，提供一些列图形：线型、圆形、矩形
*	**map**、**area**：热点

```html
<img src="planets.jpg" border="0" usemap="#planetmap" alt="Planets" />

<map name="planetmap" id="planetmap">
	<area shape="circle" coords="180,139,14" href ="venus.html" alt="Venus" />
	<area shape="circle" coords="129,161,10" href ="mercur.html" alt="Mercury" />
	<area shape="rect" coords="0,0,110,260" href ="sun.html" alt="Sun" />
</map>
```	

---
## 标签语义化
明白每个标签的用途，用正确的标签来描述页面。
**作用**：
1.	便于SEO
2.	可访问性（屏幕阅读性）
3.	代码可读性	

				
