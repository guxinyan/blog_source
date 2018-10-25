---
title: Javascript的执行上下文
date: 2018-10-24 15:41:04
tags:
    - javaScript
    - 前端
categories:
    - 学习笔记
---

重新翻到执行上下文这个概念，发现当年阅读的时候，只关注了作用域链这件事，忽略了这个比较有意思的知识点。那么今天来稍稍整理下。

# 执行上下文栈(ESC)
所谓的执行上下文栈，也就是执行环境栈。
每个函数都有自己的执行上下文，执行每个函数的时候，都会进入函数的执行环境。那么在执行过程中，引擎怎么去管理这么多执行环境呢。因此有了这样的一个执行环境栈的概念。“栈”， 就是数据结构中的概念，一个后进先出的表。
这可以理解为我们平时经常用的递归。

<!-- more -->
## 举个栗子
高程上的一个经典栗子：
```js
var color = 'blue';
function changeColor (){
  var anotherColor = 'red';
   
  function swapColor(){
    var tempColor = anotherColor;
    anotherColor = color;
    color = tempColor;
  }

  swapColor();
}

changeColor();
```
首先刚开始执行的时候，我们的环境栈为：
```js
ESCStack = [
  globalContext
];
```
到14行changeColor()时， changeColor的执行环境入栈：
```js
ESCStack = [
  changeColorContext,
  globalContext
];
```
进入changeColor之后，11行swapColor()时，swapColor的执行环境入栈：
```js
ESCStack = [
  swapColorContext,
  changeColorContext,
  globalContext
];
```
然后随着代码依次执行完成，swapColorContext， changeColorContext，globalContext依次出栈。
一个简单的示意图：
<img src="/blogImg/execution_context.png" alt="">

## 执行环境有哪几种
* 全局环境：最外围的一个执行环境， 根据js实现所在宿主环境不同，表示执行环境的对象也不一样。全局执行环境直到应用程序退出时才会被销毁。
* 函数环境：每个函数都有自己的执行环境，当执行流进入一个函数时，函数的环境就会被推入环境栈。
* eval：eval内部的文本被执行的时候

___

# 执行上下文
讲完了执行上下文栈，那么再来说说什么每一个组成--执行上下文。
## 什么是执行上下文
执行上下文可以抽象成以下对象：
```js
executionContextObj = {
    scopeChain: [], //作用域链
    variableObject: {}, //变量对象 || 活动对象
    this: {}  //
}
```
作用域链和this其实是一个老生常谈的话题了，那么变量对象是什么呢？
每个执行环境中定义的所有*变量*和*函数*都保存在这个对象中。
全局环境的变量对象就是全局对象。那么在函数里呢，我们用活动对象来作为变量对象。

稍微提一嘴作用域，
执行上下文的作用域的前端，始终都是当前执行环境的变量对象，下一个变量对象来自当前环境的外部环境，依次类推，一直延续到全局对象。这就是作用域链的组成。

## 函数执行过程
每次调用函数，都会创建新的执行上下文，而调用执行上下文分为两个阶段，创建和激活。
### 创建的时候
当函数被调用，但是还未执行任何内部代码之前， 会做以下三件事：
1、创建作用域链，即设置scopeChain的值
2、创建变量VO
  * 创建arguments对象，初始化参数名称和值并创建引用的复制
  * 扫描上下文的函数声明：
    找到函数声明，将函数名和函数的引用存入VO中；若VO中已存在同名函数，*覆盖*。
  * 扫描上下文的变量声明：
    找到变量声明，将变量名存入VO中，并初始化为undefined；若VO中已存在同名属性(变量，函数，形参)，则*不做任何操作继续往下扫描*。

3、求this的值

### 执行的时候
在代码执行阶段，会根据代码的顺序，一行一行执行，改变变量对象的值。

### 总结
那么从函数创建和执行的过程，我们有了一些变量提升的思考。
```js
变量提升的值是undefined
函数提升的是整个声明
function test(){
  console.log(a); // undefined
  console.log(f); // ƒ f(){}

  var a = 'red';
  function f(){}
}

test();
```

看看变量声明和函数声明的优先级
```js
变量已存在同名变量的时候，不会覆盖，依旧打印出函数，而不是undefined
函数已存在同名函数的时候，直接覆盖
function test(){
  console.log(f); //ƒ f(){
                  //我不一样 
                // }
  function f(){}
  function f(){
    //我不一样 
  }
  var f = 'color';
}

test();
```

看看变量声明和形参的优先级
```js
function test(f){
  console.log(f); //red
  var f = 'color';
}

test('red');
```
再加个函数声明看看
```js
function test(f){
  console.log(f); // f f(){}
  var f = 'color';
  function f(){}
}

test('red');
```
*以上，我们可以悄咪咪得出这样的创建时变量提升的优先级： 函数 > 形参 > 变量

```js
这个例子是想说明，变量提升的优先级只存在于创建时，至于函数执行时候的变量覆盖，完全看代码执行到哪一行了。
function test(){
  console.log(f); //ƒ f(){
                  //我不一样 
                // }
  var f = 'color';
  console.log(f); // color
  function f(){}
  function f(){
    //我不一样 
  }
}

test();
```


参考链接：
[JavaScript深入之变量对象](https://github.com/mqyqingfeng/Blog/issues/5)
[了解JavaScript的执行上下文](https://yanhaijing.com/javascript/2014/04/29/what-is-the-execution-context-in-javascript/)






