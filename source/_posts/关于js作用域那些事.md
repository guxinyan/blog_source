---
title: 关于js作用域那些事
date: 2017-01-12 14:39:29
tags:
    - javaScript
    - 前端
categories:
    - 学习笔记
---

---
# 什么是作用域
## 解释型语言与编译型语言
*   编译型语言在程序执行之前，有一个单独的编译过程，将程序翻译成机器语言的文件，运行时不需要重新翻译。如c、c++。
*   解释型语言是在运行的时候将程序翻译成机器语言，相对速度要比编译型语言慢。如典型的basic

其实，之前本人对编译型语言和解释型语言的界限简单定位是否有中间文件，但是注意！！有些情况下， 我们很难简单地去区分语言到底属于编译型还是解释型， 比如业界比较有争议性的java。还有， 我们比较熟悉的javascript。

> 尽管通常将JavaScript归类为“动态”或“解释执行”语言，但事实上它是一门编译语言。这个事实对你来说可能显而易见，也可能你闻所未闻，取决于你接触过多少编程语言，具有多少经验。但与传统的编译语言不同，它不是提前编译的，编译结果也不能在分布式系统中进行移植。
尽管如此，JavaScript引擎进行编译的步骤和传统的编译语言非常相似，在某些环节可能比预想的要复杂。

以上引自《你不知道javaScript》，第一次看到这样的说法，稍作记录，有待考证。
<!--more-->
## 作用域、引擎、编译器三兄弟
* 引擎：从头到尾负责整个JavaScript程序的编译和执行过程，[了解更多](http://web.jobbole.com/84351/)
* 编译器：引擎的好朋友之一，负责语法分析以及代码生成等
* 作用域：引擎的另一位好朋友，负责收集并维护由所有声明的标识符（变量）组成的一系列查询，并实施一套非常严格的规范，确定当前执行的代码对这些标识符的访问权限。

> 以var a = 2为例，会分成两步完成
1、遇到var a,编译器会询问作用域是否已经有一个该名称的变量存在同一作用域，如果是，编译器会忽略该声明；否则它会要求作用域在当前作用域声明一个新的变量，并命名为a.
2、代码运行时，引擎会首先询问作用域，在当前作用域中是否存在一个叫做a的变量，如果是，引擎就会引用这个变量；如果否，引擎会继续查找该变量。找到a变量，就会将2赋值给它，否则引擎就会抛出一个异常。

<img src="/blogImg/作用域.jpeg" alt="作用域">
## LHS && RHS
简单来说，RHS查询就是简单地查找某个变量的值，而LHS查询则是试图找到变量的容器并对它赋值。
RHS查询和LHS查询都会从当前作用域中开始进行查找，如果没有，则会继续向外层作用域继续查找，一直到达最外层作用域（全局作用域）。
> 如果RHS查询在所有嵌套的作用域中遍寻不到所需的变量，引擎就会抛出ReferenceError异常。
如果RHS查询找到了一个变量，但是你尝试对这个变量的值进行不合理的操作，比如试图对一个非函数类型的值进行函数调用，或者引用null或undefined类型的值中的属性，那么引擎会抛出另外一种类型的异常，叫作TypeError。

> 相较之下，当引擎执行LHS查询时，如果在顶层（全局作用域）中也无法找到目标变量，全局作用域中就会创建一个具有该名称的变量，并将其返还给引擎，前提是程序运行在非“严格模式”下。

> ES5中引入了“严格模式”。同正常模式，或者说宽松/懒惰模式相比，严格模式在行为上有很多不同。其中一个不同的行为是严格模式禁止自动或隐式地创建全局变量。因此，在严格模式中LHS查询失败时，并不会创建并返回一个全局变量，引擎会抛出同RHS查询失败时类似的ReferenceError异常。

---
# 词法作用域vs动态作用域
## 概念
*   静态作用域：又称词法作用域，由程序定义的位置决定，一般采用嵌套作用域的方式解析。
*   动态作用域：由程序执行顺序决定

以一个简单地例子来说明:
```javaScript
var x = 10;
function foo(){
    console.log(x);
}
function bar(){
    var x = 20;
    foo();
}
bar();
```
*   静态作用域分析
<img src="/blogImg/静态作用域.jpeg" alt="静态作用域">
对静态作用域而言，作用域由程序定义的位置决定，在foo函数中没有x,词法作用域让foo()中的x通过RHS引用到了全局作用域中的x,因此会输出10。

*   动态作用域并不关心函数和作用域是如何声明以及在何处声明的，只关心它们从何处调用，基于调用栈的。因此，在动态作用域中，最终会输出20。因为foo()无法找到x的变量引用时，会顺着调用栈在调用foo()的地方查找x,即bar函数中x(20)。

## 词法环境管理
当javaScript引擎开始执行（进入）一段可执行代码之后，会生成一个执行环境（Execution Context),或执行上下文。javascript引擎通过一个环境栈（ Stack ）来维护执行环境，当进入一个执行环境，则将当前运行的执行环境压入到这个栈的顶部。而函数执行之后，栈将环境弹出，把控制权返回给之前的执行环境。
<img src="/blogImg/词法环境.jpeg" alt="词法环境">
执行环境由词法环境(Lexical Environments)、变量环境(Variable Environment)、this关键字(This Binding)绑定三部分组成。
*   注意：词法环境的复数形式，我们可以简单理解为一个执行环境之中，有由一组相互关联的词法环境组成的”词法环境数组“。
*   变量环境可以简单理解为，与词法环境（Lexical Environments）刚开始是一样的，只是词法环境在执行过程中可能会变化，但是变量环境不会。

接下来，我们还是以例子来说明词法环境管理。
```javaScript
var x = 10;
function foo(y){
    var z = 30;
    function bar(q){
        return x+y+z+q;
    }
    return bar;
}
var bar = foo(20);
bar(40);
```
随着代码的执行，词法环境一直在变化。
下图是整个词法环境的最终结果示意图：
<img src="/blogImg/词法环境例子.jpeg" alt="词法环境例子">

## 词法环境初始化（声明提升）
词法环境都有一次初始化，也就是我们常说的声明提升。在代码执行之前，引擎都会对javaScript代码进行编译。
> 包括变量和函数在内的所有声明都会在任何代码被执行前首先被处理。var a = 2，会被分为两步：var a;a = 2。var a 会在编译阶段进行的，a = 2会留在原地等待执行阶段。

注意：函数声明和变量声明都会被提升。但是函数会首先提升，然后才是变量。

下图是上面例子代码未执行时的全局词法环境初始化
<img src="/blogImg/变量提升1.jpeg" alt="变量提升1">

## “欺骗"词法
在javaScript中，有两种方式可以在运行时来”修改“词法作用域。
*   eval
> javaScript中的eval()函数可以接受一个字符串作为参数，并将其中的内容视为好像在书写时就存在于程序中这个位置的代码。
在严格模式下，eval()在运行时有自己的词法作用域，意味着无法修改所在作用域。
*   with
> with可以讲一个没有或有多个属性的对象处理为一个完全隔离的词法作用域，因此这个对象的属性也会被处理为定义在这个作用域中的词法标识符。

---
# 函数作用域、块作用域
## 函数作用域
> 函数作用域：属于这个函数的全部变量都可以在整个函数的范围内使用及服用（事实在嵌套的作用域中也可以使用）

说到函数作用域，一般我们都要来顺道提下函数声明和函数表达式。
> 解析器会率先读取函数声明，并使其在执行任何代码之前可用（声明提升）；函数表达式则必须等到解析器执行到它所在的代码行，才会真正被解释执行。

```js
函数声明：
function foo(){
    console.log('test');
}

一般函数表达式：
var foo = function(){
    console.log('test');
}

匿名函数表达式：
setTimeOut(function(){
    console.log('test');
},1000);

立即执行函数表达式(IIFE)
(function foo(){
    console.log('test');
})();
```
## 块作用域
*   with
上代码说明：
```javaScript
var a = 1;
var o1 = {
    a:2
}
var o2 = {
    b:3
}

function foo(obj){
    with(obj){
        console.log(a);
    }
}

foo(o1); //2
foo(o2); //1
```
with会将对象自成作用域，对象中的属性称为作用域中的词法标识符；因此o1中的a属性直接覆盖了全局作用域中的a变量。而o2中没有a属性，会向上查找，知道找到全局作用域中的a。
*   try/catch
```javaScript
var foo = 10;
try{
    throw 20;
}catch(foo){
    function f(){
        console.log(foo);
    }
    f(); //20
}
```
按照我们原以为的js中普通{}理解，函数f的声明应该outer是指向全局作用域的foo（10）,但是最终结果为20;说明try/catch会创建一个块作用域。
*   let
let关键字是es6引入的，为其声明的变量隐式劫持在了所在的块作用域。
```javaScript
if(true){
    let bar = 3;
    console.log(bar); //3
}
console.log(bar)//ReferenceError
```
*   const
const也是es6引入的，与let类似，也能创建块作用域变量，不用的是const创建的值是固定的常量。

---
# 闭包
## 什么叫闭包
个人认为，简单理解，满足两种条件的可以称为闭包：1、可以访问其他作用域 2、在自己定义的词法作用域以外的地方执行，依然持有对该作用域的引用。
```js
function foo(){
    var a = 2;
    function bar(){
        console.log(a);
    }
    return bar;
}
var baz = foo();
baz(); //2 ---这就是闭包
```
> 其实，在定时器，事件监听器、Ajax请求、跨窗口通信、Web Workers通信或者任何其他的异步（或者同步）任务中，只要使用了回调函数，实际上就是在使用闭包！

那么现在，我讲拿出我们经典的循环例子来说明----闭包
```js
for(var i = 0; i <= 5; i++){
    setTimeOut(function(){
        console.log(i);
    }, i*1000);
}
结果:每隔一秒打印一个6
原因：for循环声明了6个函数，每隔一秒执行；在执行的时候，引擎会去作用域查找变量i,而实际上5个函数的i都是被定义在全局作用域中共享的，全局作用域中的i为6。
```
```js
以上代码用立即执行函数实现
for(var i = 0; i <= 5; i++){
    (function(j){
        setTimeOut(function(){
            console.log(j);
        },j*1000);
    })(i);  
}
结果：每隔一秒输出[0, 1, 2, 3, 4, 5]
```
## 模块
说到闭包，我们就不得不提下现今非常流行的---模块。
什么叫做模块？(1)为创建内部作用域调用了一个包装函数 (2)包装函数的返回值必须至少包括一个对内部函数的引用，这样就会创建涵盖整个包装函数内部作用域的闭包。
```js
现代的模块机制
var myModules = (function manager(){
    var modules = {};
    function define(name, deps, impl){
        for(var i=0; i< deps.length; i++){
            deps[i] = modules[deps[i]];
        }
        modules[name] = impl.apply(impl, deps);
    }
    function get(name){
        return modules[name];
    }
    return {
        define:define,
        get:get
    }
})();

myModules.define('bar', [], function(){
    function hello(who){
        return 'let me introduce '+who;
    } 
    return {
        hello:hello
    }
});

myModules.define('foo', ['bar'], function(){
   var hungry = 'hippo';
   function awesome(){
        console.log(bar.hello(hungry).toUpperCase());
   } 
   return {
        awesome:awesome
   }
});
var bar = myModules.get('bar');
var foo = myModules.get('foo');
console.log(bar.hello('hippo')); // 'let me introduce hippo'
foo.awesome(); // 'LET ME INTRODUCE HIPPO'
```

