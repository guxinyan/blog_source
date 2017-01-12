---
title: 关于js异步的一些概念理解
date: 2017-01-03 11:53:01
tags:
    - javaScript
    - 前端
categories:
    - 学习笔记
---

最近在看《你不知道的javaScript》,对js的异步概念有了一些比较清晰的理解。异步编程的核心就是处理程序中现在运行部分和将来运行部分之间的关系。

---
# 异步

```js
function now(){
    return 21;
}
function later(){
    answer = answer * 2;
    console.log('Meaning of life:', answer);
}
var answer = new now();
setTimeout(later, 1000);
```
<!--more-->
这个程序中，分为现在执行的部分和将来执行的部分。
现在：
```js
function now(){
    return 21;
}
function later(){...}
var answer = new now();
setTimeout(later, 1000);
```
将来：
```js
answer = answer * 2;
console.log('Meaning of life:', answer);
```
setTimeout设置了一个定时事件在将来执行，所以现在的代码块会立即执行，函数later的部分会在之后的某个时间（从现在起的1000毫秒之后）执行。
任何时候，只要把一段代码包装成一个函数，并制定它在响应某个时间（定时器、鼠标点击、Ajax事件）时执行，就是在代码中创建了一个将来执行的代码的块，也就是引入了*异步机制*。

---
# 异步控制台
```js
var a={
    index:1
}
console.log(a);
a.index++;
```
正常情况下，我们会认为输出的是a对象的快照，打印出来{index:1}。但是有可能会先执行a.index++，再打印这样的情况的(虽然一般情况不会发生)。那么，为什么有这样的发生呢？
因为console.\*方法并不是javaScript正式的一部分，是属于宿主环境添加到javaScript中的，因此不用浏览器可以按照自己的意愿来实现。可能，在某些条件下面，某些浏览器并不会马上把传入的内容console.log()出来。*在很多程序中，I/O是非常低速的阻塞部分，所以，浏览器可能会在后台异步处理控制台I/O来提高性能*。

---
# 事件循环
javaScript引擎并不是独立运行的，它运行在宿主环境中，对我们多数前端开发而言通常宿主就是Web浏览器（当然还有其他环境，比如通过Node.js这样的工具进入服务器领域）。但这些所有环境，都有一个共同“点”，即都提供了一种机制来处理程序中多个块的执行，且执行每块调用javaScript引擎，这种机制被称为“事件循环”。
通过一段伪代码来了解下概念：
```js
//eventLoop是一个用作队列的数组
//先进先出
var eventLoop = [];
var event;
//‘永远’执行
while(true){
    //一次tick
    if(eventLoop.length > 0){
        //拿到队列中下一个事件
        event = eventLoop.shift();

        //现在，执行下一个事件
        try{
            event();
        }
        catch(err){
            reportError(err);
        }
    }
}
```
循环的每一轮称为一个tick。如果队列中有等待事件，就会从队列中摘下一个事件并执行，这些事件就是我们常用的回调事件。

setTimeout()并没有马上把回调函数挂在事件循环队列中，而是设置了一个定时器，到底时间之后将回调函数放在事件循环中，这样，在未来某个时间tick会执行这个回调函数。所以，*定时器的精度可能并不高*，只能确保回调函数不会发生在指定时间间隔之前，可能发生在那个时刻，也可能发生在那个时刻之后，由事件队列的状态而定。

---
# 并行线程
异步是关于现在和将来的时间间隔，而并行是关于能够同时发生的事情。接下来来举例说明：
```js
var a=20;
function foo(){
    a = a + 1;
}
function bar(){
    a = a * 2;
}
ajax('url1', foo);
ajax('url2', bar);
```
在单线程环境中，只会出现两种情况，foo运行在前，那么结果便是42;bar运行在前，结果便是41。
但是，假设javaScript之间并行执行的话， 会出现什么情况呢？
```
线程1（X和Y是临时内存地址）：
foo()
a.把a的值加载到X
b.把1保存在Y
c.执行X加Y，把结果保存在X
d.把X的值保存在a
```
```
线程2（X和Y是临时内存地址）：
bar()
a.把a的值加载到X
b.把2保存在Y
c.执行X*Y，把结果保存在X
d.把X的值保存在a
```
假设两个线程并行执行，因为共享了临时内存地址X和Y,将会发生很多种情况。比如：
```
1a (把a的值加载到X        ===> 20)
2a (把a的值加载到X        ===> 20)
2b (把2保存在Y           ===> 2)
1b (把1保存在Y           ===> 1)
1c (执行X加Y，把结果保存在X ===> 21)
2c (执行X*Y，把结果保存在X ===> 21)
2d (把X的值保存在a       ===> 21)
1d (把X的值保存在a       ===> 21)
```
a的结果将是21。
可以看出，在多线程编程中，不确定性会在语句顺序级别；而javaScript中的不确定性只在foo函数和bar函数的先后执行顺序上。一旦foo函数开始执行，将不会被bar函数打断。这称为*完整运行特性*。
函数顺序的不确定性就是通常所说的*竞态条件*，bar()与foo()互相竞争，看谁先运行。

---
# 并发
并发指的是在同一时间间隔发生，并行指的是同一时刻。
单线程事件循环是并发的一种形式。
例如：网站中向下滚动加载列表，正确实现这一功能，就需要至少两个独立“进程”同时进行：向下滚动触发请求；接受ajax响应，加载dom。如果用户滚动足够快的话，在第一个响应被处理的时候也可能发生多个滚动事件被触发。（虽然最终执行都是一个一个执行）
## 非交互
两个或多个“进程”在程序内并发的交替运行事件，彼此不相关。那么，不确定性可被接受。
## 交互
```js
var res = [];
function response(data){
    res.push(data);
}
ajax('url1', response);
ajax('url2', response);
```
这里并发的“进程”运行的先后顺序会直接影响到res数组。这种不确定性就是一种竞态条件bug。
所以可以通过协调交互顺序来处理：
```js
function response(data){
   if(data.url === 'url1'){
        res[0] = data;
   }else if(data.url === 'url2'){
        res[1] = data;
   }
}
```
## 协作
一个长期运行的“进程”，将其分割成为多个步骤或多批任务，使得其他并发“进程”有机会将自己的运算插入到事件循环队列中交替运行。
```js
var res = [];
function response(data){
    res = res.concat(
        data.map(function(val){
            return val*2;
        })
    );
};
ajax('url1', response);
ajax('url2', response);
```
在记录少的情况之下，性能不会有什么影响，但如果url1取得的结果有上千万条数据的话，可能运行一段时间了，这样会导致阻塞状态出现。所以，要创建一个协作性更强且不会霸占事件循环队列的并发系统。
比如：
```js
function response(data){
    //一次处理1000个
    var chunk = data.splice(0, 1000);
    //添加到已有数组中去
    res = res.concat(
        chunk.map(function(val){
            return val*2;
        })
    );

    //还有剩下的处理？
    if(data.length > 0){
        //异步调度下一批次处理
        setTimeout(function(data){
            response(data);
        },0);
    }
};
```
事件循环队列的交替运行会提高站点/APP的响应（性能）。

---
# 任务
任务队列：挂在事件循环队列的每个tick之后的一个队列。


