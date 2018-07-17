---
title: React v16.0.0+更新了什么
date: 2018-07-16 20:16:29
tags:
    - React
    - 前端
categories:
    - 学习笔记
---
 
React 16之后，不仅仅是切换到MIT协议，还有了很多突破性改变和新功能，本文主要是对react 16及之后的版本更新有个大致的整理。

---
# Fiber架构
*v16.0.0更新*
## Stack Reconciler存在什么问题呢
在原有的React中，更新过程是同步的。什么意思呢？
React挂载或者更新的时候，递归调用各个组件生命周期，对比virtual DOM，更新DOM树，整个过程一旦开始，不可被打断。这样的设计，在组件层级比较深，dom树比较庞大的时候，就会有问题出现了。
<!-- more -->
* 整个浏览器主线程被渲染过程占用比较长的时间，导致进程阻塞，让界面失去响应。
> 因为JavaScript在浏览器的主线程上运行，恰好与样式计算、布局以及许多情况下的绘制一起运行。如果JavaScript运行时间过长，就会阻塞这些其他工作，可能导致掉帧。
* 渲染过程没有优先级可言，不同状态变化是需要有优先级的，比如动画的状态变化，避免动画卡帧一般优先级要高一些。

这里有一张图可以表示这个过程：
<img src="/blogImg/stackR.jpg" width="600px;">

## Fiber Reconciler
Fiber是React核心算法的重构，旨在为了更好的扩大在布局，动画，手势方面的适应性。Fiber把渲染过程分成了一个个chunk， 变同步为异步，进一步提升用户感知性能。
<img src="/blogImg/fiberR.jpg" width="600px;">
Fiber的关键特性：增量渲染(将渲染任务拆分成块，匀到多帧)。这种情况下，渲染任务每次只做一小段，做完之后就把控制权交还给主线程，而不像之前那样一条路走到黑的占用主线程到底。
好处呢：
* 渲染过程中有机会将主线程交给react以外的js调用（比如一些用户操作）
* 渲染有了优先级，能让高优先级的工作抢先做。

## Fiber reconciler的两个阶段
Fiber之后，React将更新分为了两个部分。
<img src="/blogImg/reactPhase.png" width="600px;">
* phase1: render/reconciliation<span style="color:red">(可中断)</span>
<img src="/blogImg/fiberPhase1.png" width="500px;">
左边灰色是一个fiber tree，每一个fiber是一个工作单元，自顶向下逐节点构建workInprogress tree（所谓的中间态）。
一个工作单元判断更新，更新节点状态（props，state, context），调用`render`，向上归并effect list（简单理解为更新点）。
> 构建workInProgress tree的过程就是diff的过程，通过requestIdleCallback来调度执行一组任务，每完成一个任务后回来看看有没有插队的（更紧急的），每完成一组任务，把时间控制权交还给主线程，直到下一次requestIdleCallback回调再继续构建workInProgress tree。
所有的工作单元工作完毕之后，就进入了pendingCommit状态。

* phase2: commit<span style="color:red">(不可中断)</span>
处理effect list（包括3种处理：更新DOM树、调用组件生命周期函数以及更新ref等内部状态）。
将所有变更一次性更新到DOM树上。

被划分的生命周期：
* phase 1: 
```js
componentWillMount
componentWillReceiveProps
shouldComponentUpdate
componentWillUpdate
```

* phase 2:
```js
componentDidMount
componentDidUpdate
componentWillUnmount
```

第一阶段的生命周期会随时被打断，因为增加了所谓优先级的概念。假设组件A在阶段1过程中，时间片用完了，把控制权交给主线程之后，发现有更高优先级的事情要做。这时候，组件A的渲染过程就会被放弃，只能等待下次**从头开始**。
**所以！！！第一阶段的生命周期很有可能会被调用多次！！所以我们建议第一阶段的生命周期函数最好都是纯函数。**

现阶段这个异步渲染React还没有默认开放。
 
**参考链接**
[Lin Clark 的演讲视频](https://www.youtube.com/watch?v=ZCuYPiUIONs)
[React Fiber是什么](https://zhuanlan.zhihu.com/p/26027085)
[完全理解React Fiber](http://www.ayqy.net/blog/dive-into-react-fiber/)

---
# 生命周期的改变
*v16.3.0更新*
## 即将移除的
这三个生命周期可能会导致一些不安全的体验，随着异步渲染模式的引进，问题会越来越多。
```js
componentWillMount
componentWillReceiveProps
componentWillUpdate
```
在v16.3.0中，为这三个生命周期函数增加了别名： UNSAFE_componentWillMount, UNSAFE_componentWillReceiveProps, 和UNSAFE_componentWillUpdate。
componentWillMount,componentWillReceiveProps,componentWillUpdate在16.x版本中还是可以使用的，但是React会在开发环境启用warning。在17.0及以后， 会将这三个老的生命周期函数彻底移除，只提供带'UNSAFE'前缀的用法。

## 新增的
同时呢，React新增了以下两个生命周期函数。
```js
static getDerivedStateFromProps(props, state)
```
getDerivedStateFromProps会在render函数之前被调用，在初次挂载阶段和后期的更新阶段都会调用。
需要返回一个object去更新state, 或者返回null表示没有更新。

这个阶段拿不到组件的实例。

```js
getSnapshotBeforeUpdate(prevProps, prevState)
```
在update发生的时候，render函数之后，组件渲染之前被调用。
这个函数的返回值将会作为componentDidUpdate的第三个参数。

---
# 更好的错误处理方式
*v16.0.0更新*
React 16提供了一种新的错误处理的机制，叫“error boundary”。
## 以前的错误处理
其实以前的react有一个秘而不宣的处理错误的方式：
```js
unstable_handleError: function(error){
  this.setState({error: "oops"})
}
```
但是这个处理方式：
* 只能处理render抛出的Error，其他的生命周期函数触发不了。
* 能捕获当前组件本身抛出的Error，但是还是会阻碍接下来的渲染。

## componentDidCatch
React 16之后引入了新的生命周期函数`componentDidCatch(error, info)`。
```js
error就是捕获的错误
info：错误信息栈，例如：
{
  componentStack: "
    in BuggyCounter (created by App)
    in ErrorBoundary (created by App)
    in div (created by App)
    in App"
}

```
用法如下：
```js
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  componentDidCatch(error, info) {
    this.setState({ hasError: true });
    // 上报错误
    logErrorToMyService(error, info);
  }

  render() {
    if (this.state.hasError) {
      return <h1>Something went wrong.</h1>;
    }
    return this.props.children;
  }
}
```
```js
<ErrorBoundary>
  <otherComponent></otherComponent>
</ErrorBoundary>
```
ErrorBoundary作为一个专门处理错误的异常边界组件。
* 能捕获所有子组件，任何生命周期函数抛出的异常
* 只能捕获子组件抛出的error，组件本身的异常无法捕获。
因此我们的异常边界组件不能涉及业务代码，否则将无法捕获自身逻辑错误。

子组件抛出的错误会层层往上传递，直到被捕获，一到捕获之后，就不会再往上传播。
> 注意：
Error boundaries 不能捕获的错误
- 事件处理
- 异步 (e.g. setTimeout or requestAnimationFrame callbacks)
- 服务端渲染
- 组件自身的错误

## 错误处理表现
在React16之后，未被捕获的错误，错误会一直往上抛，直到React将整个React Tree全部卸载。
同时，在开发环境，控制台会输出所有的错误，包括被捕获的错误。

---
 # 新的render返回

---
 # portal

---
 # 标签自定义属性

---
 # 新的Context API

---
 # 新的ref API

---
 # 严格模式组件

 
---
# 更强的服务端渲染

---
 # 其他
