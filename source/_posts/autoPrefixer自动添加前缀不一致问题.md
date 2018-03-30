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
搜了（相关资料之后）[https://docs.npmjs.com/getting-started/using-a-package.json]
> npm uses Semantic Versioning, or, as we often refer to it, SemVer, to manage versions and ranges of versions of packages.
If you have a package.json file in your directory and you run npm install, npm will look at the dependencies that are listed in that file and download the latest versions, using semantic versioning.

当我们输入npm install之后， 发生了什么？
npm包有个大小版本的概念， 直接npm install之后，依赖package.json中的大版本控制，比如1.0.2，会去搜寻这个`1.0`版本集中最新的那个npm包，也许是1.9.9哈哈。

附上重要链接：
(autoprefixer)[https://github.com/postcss/autoprefixer#options]
(Browserslist)[https://github.com/browserslist/browserslist#queries]
