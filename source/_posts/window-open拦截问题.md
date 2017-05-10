---
title: window.open拦截问题
date: 2017-05-10 15:50:22
tags:
    - Angular
    - 前端
categories:
    - problem solving
---

# 场景
<img src="/blogImg/window-open-scene.png" alt="场景示意图" width="400px;">
在一次偶然的需求中，遇到了这样一种情况：输入对应的查询条件之后，点击按钮，向后端发送请求，由后端进行验证（前端无法验证一些情况），假设后端返回true，则打开一个新窗口，否则进行错误提示。
<!--more -->

---

## 刚开始的使用方式
按钮有点击事件，大意如下：
```js
function handelClick(){
    Service.valid(params).then(function(data){
        if(!data)return;
        var url = 'xxxx';
        window.open(url);
    });
}
```
但是这样的情况下，新开的窗口会被大多数浏览器拦截。
第一反应，我觉得可能是在异步请求回调函数中打开窗口导致的，所以进行了以下试验：
```js
function handelClick(){
    var newWin = false;
    var url = 'xxxx';
    Service.valid(params).then(function(data){
        if(!data)return;
        newWin = true;
    });
    newWin && (window.open(url));
}
```
这样并没有任何反应，这个锅呢可能就要promise来背了。
> promise中then指定的回调函数会在当前脚本所有同步任务执行完才会执行。

所以，流程是先执行newWin && (window.open(url))这行代码，然后才执行newWin = true。
下意识呢，马上，我讲下面的部分代码改成了异步模式
```js
function handelClick(){
    var newWin = false;
    var url = 'xxxx';
    Service.valid(params).then(function(data){
        if(!data)return;
        newWin = true;
    });
    var timer = setInterval(function(){
        if(newWin){
            window.open(url);
            clearInterval(timer);
        }
    },300);
}
```
然而，弹窗是出现了，还是被拦截了。当然，我还没死心，也试过了promise的方式去控制弹窗，也没有起作用。但是如果，我不是通过service改变newWin的值，直接设置为newWin为true的话，窗口并没有被拦截。
所以：
> 依赖于异步请求结果改变某些值的改变来控制开启弹窗，都会被浏览器拦截。

---
## 我所尝试的大部分的解决方案
### 重定向
```js
function handelClick(){
    var newWin = window.open();
    var url = 'xxxx';
    Service.valid(params).then(function(data){
        if(!data)return;
        newWin.location = url;
    });
}
```
这个方法并不太适合当前的使用场景，因为在后端返回false的时候，会有一个空白弹窗，我们也可以用newWin.close()去控制，但是在交互体验上并不提倡。但是在一些针对不同后端返回结果，打开不同页面的需求，这个方法还是很合适的。

### 模拟a标签
模拟a标签有两种方式：动态生成a标签和在页面中写一个隐藏a标签，触发a标签的点击事件。然而这两种方式都不太适合异步请求回调的场景。

### 模拟form表单
同a标签

### 利用事件冒泡
```html
<a ng-click="newWindow()">
    <button ng-click="handelClick($event)"></button>
</a>
```
```js
function handelClick(event){
    if(参数不对){
        event.stopPropagation();
        return;
    }
    Service.valid(params).then(function(data){
        if(!data){
            event.stopPropagation();
            return;
        }
    });
}
funtion newWindow(){
    var url = 'xxxx';
    window.open(url); 
}
```
在这段代码中，我加了一个前端自身的表单验证。
*   当验证不通过的时候，也的确是先触发了里层的handleClick事件，并阻止了冒泡行为。
*   当验证通过且后端返回true的时候，并没有按我理想状态走。而是先触发了newWindow事件，再走的回调函数。这时候，promise又得背一回锅了。我的又一个方案失效了。

---
## 最终的解决方案：设置同步
这里以jquery举例
```js
$.ajax({
    async:false,  //同步
    url:'xxxx'
});
```
这个方法也的确是生效的。但是由于项目基础封装原因，项目调用了angular的$http去实现接口请求。（无奈脸）我也就止步于此了，只能找交互大大改交互了。

如果有大神有好方法的话，请多多指教哦。







