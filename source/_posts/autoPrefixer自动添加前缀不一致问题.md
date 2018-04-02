---
title: autoPrefixer自动添加前缀不一致问题
date: 2018-03-30 20:29:03
tags:
    - 打包
    - 前端
categories:
    - problem solving
---

# 问题描述
今天在工作中遇到这样一个问题，两台机器同一个页面打包出来的页面样式在ios8上的表现非常不一致！
仔细看了页面的样式，发现用了flex布局， 而ios8需要-webkit-前缀。 
暂时先不考虑css写法严不严谨的问题， 但是咱们不是用了autoPrefixer自动处理css样式了吗？？？
这貌似没生效啊？

<!-- more -->

# 原因
自己观察了一下两台机器打包出来的样式， 表现正常的那台样式，既有-ms-前缀，又有-webkit-前缀；而表现异常的那台竟然只有-ms-前缀，为什么不自动添加webkit呢？
一模一样的打包配置，为什么会有两种结果呢？
```js
autoprefixer({ browsers: ["last 2 versions"] })
```
这是一个官网上很正常的配置，去搜寻当前目前各个浏览器稳定的两个版本，查看支持特性，不支持，则相对应加前缀。

> You have different versions of caniuse-db on different machines.

在网上搜了很多资料看到这样一句话，last 2 versions 是依赖本地机器下载的caniuse-db的。
所以，不同机器上下载的caniuse-db不同导致了这样的结果。

到这里， 我又有了一个疑问。我们本地包的下载都是依赖package.json的呀，里面都写死了autoprefixer包的版本号，为什么会产生caniuse-db包的不用呢。
`关于package.json version那些事`：
指定版本：比如，1.1.1；遵循大版本.次要版本.小版本的格式规定
波浪号+指定版本：比如， ~1.1.1；表示安装1.1.x的最新版本， 不低于1.1.1
插入号+指定版本：比如， ^1.1.1；比碍事安装1.x.x的最新版本， 不低于1.1.1
latest: 安装最新版本


附上重要链接：
(autoprefixer)[https://github.com/postcss/autoprefixer#options]
(Browserslist)[https://github.com/browserslist/browserslist#queries]
