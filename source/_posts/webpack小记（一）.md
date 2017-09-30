---
title: webpack小记（一）
date: 2017-09-27 16:25:54
tags:
  - 前端
  - 打包
categories:
  - 学习笔记
---

Webpack现在已经成为了前端模块化开发的标配。具体的理论基础介绍呢，这篇文章就不做赘述了。直接从实践入手，一一解释相关。全文基于webpack3.6.0。可能因为版本不一产生部分不同，但大体肯定是一致的。

---
# 安装和使用
## 安装方式
webpack可以作为就全局的npm包安装， 也可以在当前项目中安装
```node
  npm install -g webpack
  npm install --save-dev webpack
```
<!-- more -->

## 使用方式
* 命令行使用： webpack <entry> <output> (entrys是入口文件，output是打包后的文件)
webpack可以有很多选项，具体可以参考[官方文档](http://webpack.github.io/docs/cli.html)
* node api使用：
```node
var webpack = require("webpack");
// returns a Compiler instance
webpack({
    // configuration
}, function(err, stats) {
    // ...
});
```
* 默认使用当前目录的webpack.config.js作为配置文件。如果想要别的配置文件，可以使用webpack --config webpack.test.config.js

---
# 配置文件
接下来我将以demo的形式，结合实例来分析配置文件。
新建一个文件夹webpack-demo, 在命令行该文件下面npm init。
安装webpack： npm install --save-dev webpack
```
文件夹目录为
--- webpack-demo
  --- package.json
  --- nodu_modules
  --- webpack.config.js
  --- dist
  --- src
    --- js
    --- style
  --- index.html
```
```js
在/src/js/文件夹下面添加一个main.js

webpack.config.js 
//这是一个最简单的Webpack配置文件
const path = require('path');
module.exports = {
  entry: './src/js/main.js',        //入口文件
  output: {                         //打包后的文件配置
    path: path.resolve(__dirname, "dist/js"),   //打包后的文件路径
    filename: 'bundle.js'                       //打包后的名称
  }
}
```
我们在package.json文件中的scripts属性定义一个脚本
```js
"webpack": "webpack"
```
当然我们也可以添加一些参数
```js
"webpack": "webpack --progress --display-modules --colors --display-reason"
```
在命令行下运行npm run webpack， 我们可以看到打包成功的界面。同时，在dist文件夹下面会多一个js文件夹，里面有一个bundle.js，即打包后的js文件。
## entry 
入口文件的写法也有三种方式：
* 1、 string 就是上述例子中的用法，单入口文件
* 2、 array 入口是一个数组
  在/src/js文件夹下面添加a.js，修改配置文件为：
  `entry: ['./src/js/main.js', './src/js/a.js']`
  再次打包，可以发现两个入口文件打包成了一个文件bundle.js。`这适用于多个平行入口文件打包为一个文件`。
* 3、 object 入口是一个对象 key为chunk name, value为entry(可以是string或者array)
```js
  entry: {
    'main': './src/js/main.js',
    'a': './src/js/a.js'
  }
  同时，出口也要做配置，否则将会出现覆盖或者报错现象，具体出口配置下面再细说。
  output: {
    path: path.resolve(__dirname, "dist/js")
    filename: '[name].js'
  }
```
现在打包， 在dist/js文件夹下面我们可以看到main.js 和 a.js。 `这适用于多页面应用程序打包`。
[【entry api在此】](http://webpack.github.io/docs/configuration.html#entry)
## output 
output配置只能是一个对象。
```js
output: {
    path: path.resolve(__dirname, "dist/js")
    filename: '[name].js'
    //filename: '[hash].js'
    //filename: '[chunkhash].js'
    //filename: '[name]-[chunkhash].js'
    ///...
  }
```
* path 出口文件路径。这里不能采用相对路径， path为node内置api， `__dirname`为当前根路径
* filename 出口文件名。
  入口文件为单个文件时，出口文件一般写死,如 `bundle.js`。当有多个入口文件时，出口文件就要选择以下方式啦。
  `[name]` 入口文件配置的key
  `[hash]` 此次编译的全局md5加密编码，全部chunk共享
  `[chunkhash]` 每个chunk的编码，chunk之间不相同。
  在文件没有改变的时候，chunkhash值不会改变。
* publicPath 指定输出文件的公用URL地址，通俗来说，就是公用域。
```js
entry: {
    'main': './src/js/main.js',
    'a': './src/js/a.js'
},
output: {
  path: path.resolve(__dirname, "dist/js")
  filename: '[name].js',
  publicPath: 'www.xxx.com'
}
```
在html中引入main或者的a的时候，URL为`www.xxx.com/dist/js/main.js`
[【output api在此】](http://webpack.github.io/docs/configuration.html#output)

---
# 插件系统 --- 以html-webpack-plugin为例
首先安装 npm install --save-dev html-webpack-plugin，在webpack.config.js中增加plugin配置。
> html-webpack-plugin是一个自动生成html文件的插件，插件生成的html文件会自动引入打包生成的文件。特别适用于打包文件配置了hash编码的场景。

```js
const path = require('path');
const htmlWebpackPlugin = require('html-webpack-plugin');
module.exports = {
  entry: {
    'main': './src/js/main.js',
    'a': './src/js/a.js'
  },
  output: {
    path: path.resolve(__dirname, "dist"),
    filename: 'js/[name].js'
  },
  plugins:[
    new htmlWebpackPlugin()
  ]
}
```
在dist目录下我们可以看到一个自动生成的index.html文件， 并且引入了打包后的main.js和a.js。在这里呢，我们可以发现，htmlWebpackPlugin这个插件呢，是直接将生成的html文件打包到output配置的path下面。所以呢，我将js的路径写入了filename中。

[html-webpack-plugin](https://www.npmjs.com/package/html-webpack-plugin)这个插件也可以有很多配置项，大家可以点开链接，看一下插件的npm官网的配置项说明
```js
new htmlWebpackPlugin({
  filename: 'app.html',  //可以设置生成的html的文件名
  inject: 'body',        //设置引入文件放置的位置，这里选择放在<body></body>标签中
});
```

## 传参
在根目录下面新建一个index.html文件做为一个入口文件，文件内容如下：
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>
    <%= htmlWebpackPlugin.options.title %>
  </title>
</head>
<body>
</body>
</html>
```
插件在html文件中以htmlWebpackPlugin参数的形式注入， 在html中使用的是EJS语法
```js
new htmlWebpackPlugin({
  filename: 'app.html',
  inject: 'body',
  template: 'index.html',
  title: '我的标题'
})
```
最后生成的页面我们可以看到，标题已经是'我的标题'了。
我们可以测试htmlWebpackPlugin下面挂载了两个属性： files 和 options。在很多地方都会用到这两个参数。

## 产生多个页面
以上的例子都是动态生成了一个html页面，如果假设在多页面应用中，我们要生成多个入口页面呢？
在js文件夹下面新建一个b.js c.js。我们想要实现的效果是，a b c分别属于三个页面，main是三个页面的共享js。这时候该如何配置呢？
```js
const path = require('path');
const htmlWebpackPlugin = require('html-webpack-plugin');
module.exports = {
  entry: {
    'main': './src/js/main.js',
    'a': './src/js/a.js',
    'b': './src/js/b.js',    
    'c': './src/js/c.js' 
  },
  output: {
    path: path.resolve(__dirname, "dist"),
    filename: 'js/[name].js'
  },
  plugins:[
    new htmlWebpackPlugin({
      filename: 'a.html',
      inject: 'body',
      template: 'index.html',
      title: 'a的标题',
      chunks: ['a', 'main']
    }),
    new htmlWebpackPlugin({
      filename: 'b.html',
      inject: 'body',
      template: 'index.html',
      title: 'b的标题',
      chunks: ['b', 'main']
    }),
    new htmlWebpackPlugin({
      filename: 'c.html',
      inject: 'body',
      template: 'index.html',
      title: 'c的标题',
      chunks: ['c', 'main']
    })
  ]
}
```
在代码中，我们主要利用了htmlWebpackPlugin的chunks属性，设定入口html页面添加的js。
> chunks: Allows you to add only some chunks (e.g. only the unit-test chunk)

## 嵌入js
有时候呢，我们会考虑页面引入的js请求过多，首页的某些数据出来的过慢，这时候呢就希望一些js代码能以内联的方式直接引入在入口页面中。
比如我们例子中的main.js。
1、先把htmlWebpackPlugin中的inject改为false，使不能自动引入js文件
2、该写模板index.html文件
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>
    <%= htmlWebpackPlugin.options.title %>
  </title>
</head>
<body>
<%= compilation.assets[htmlWebpackPlugin.files.chunks.main.entry].source()%>

<% for(var key in htmlWebpackPlugin.files.chunks){%>
  <% if(key != 'main'){ %>
    <script src="htmlWebpackPlugin.files.chunks[key].output"></script>  
  <%}%>
<%}%>
</body>
</html>
```
htmlWebpackPlugin.files.chunks.main.entry即拿到main的入口文件，compilation.assets[].source()即是获取源码。
下面一段代码，遍历chunks，将不是main的js引入进来。

---
# 加载器
我们重新整理文件目录，结合React + es6 + react + sass 来讲述这个例子。     
## 准备工作
```
新建好以下目录文件
--- dist
--- src
  --- js
    --- components
      --- hello
        --- hello.jsx
        --- hello.scss
    --- app.js
--- index.html
--- webpack.config.js
--- package.json
```
首先，webpack配置为（安装webpack及package.json设置就不再说明了。）：
```js
var path = require('path');
var excludePath = path.resolve(__dirname, "node_modules");
var includePath = path.resolve(__dirname, "src");
module.exports = {
  entry: {
    'app':'./src/js/app.js'
  },
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'js/[name].bundle.js'
  }
}
```
入口文件只有一个，就是app.js，并且我们在根目录下面的index.html中直接引入生成后的app.bundle.js，不采用动态生成html的方式了。

## 处理jsx 和 es6
> 下载react => npm install --save react react-dom
安装相关babel的npm包 npm install --save-dev babel-core babel-loader babel-preset-es2015 babel-preset-react

hello.jsx内容为：
```js
import React from 'react';
export default class Hello extends React.Component{
  render(){
  return (
  <div className="hello">
    hello
    <span className="world">world</span>
  </div>);
  }
}
```
在webpack配置文件中添加loader：
```js
module: {
    loaders:[
      {
        test: /\.jsx?$/,
        loader: "babel-loader?presets[]=es2015,presets[]=react",
        include: includePath,
        exclude: excludePath
      }
    ]
  }
```
打包之后，打开index.html我们就能看到helloworld了。

* tips：loader？xxx => xxx是对应的配置参数，也可以用query属性替换，或者在根目录下面新建babel配置文件。
[using loaders](https://webpack.github.io/docs/using-loaders.html)
[babel配置文件](http://babeljs.cn/docs/usage/babelrc/)

## 处理样式
> 下载样式相关loader npm install --save-dev node-sass sass-loader style-loader css-loader 

在这里我采用的是Sass。hello.scss文件为
```css
.hello{
  background: red;
  .world{
    background: green;
  }
}
```
然后在hello.jsx中引入
```js
require('./hello.scss');
```
webpack配置添加loader
```js
{
  test: /\.scss$/,
  loader: "style-loader!css-loader!sass-loader",
  include: includePath,
  exclude: excludePath
}
```
* tips:loader我们也可以理解是一个链式操作，处理顺序*从右往左*。
  style-loader 将模块的导出作为样式添加到 DOM 中
  css-loader 解析 CSS 文件后，使用 import 加载，并且返回 CSS 代码
  sass-loader 加载和转译 SASS/SCSS 文件

## 处理图片
<!-- > npm install --save-dev url-loader

将hello.scss修改为：
```css
.hello{
  background: red;
  width: 200px;
  height: 100px;
  background-image: url('../../../img/1.png');
  .world{
    background: green;
  }
}
```
在webpack配置中添加loader
```js
{
  test: /\.(png|jpg|jpeg|gif)$/,
  loader: "url-loader",
  include: includePath,
  exclude: excludePath
}
```
打包，刷新浏览器，我们就能看到helloworld的背景图已经有填充了。。
给url添加一个参数
```js
query: {
  options: {
    limit: 8192
  }
}
```
再次打包，我们发现，图片被转成base64编码了。
> The url-loader works like the file-loader, but can return a DataURL if the file is smaller than a byte limit

这是npm官网上url-loader的解释，当我们设置了limit的时候，如果图片的大小 -->


---
# 构建本地服务器

---
参考链接
[http://www.imooc.com/learn/802](http://www.imooc.com/learn/802)
