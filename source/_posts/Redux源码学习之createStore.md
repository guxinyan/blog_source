---
title: Redux源码学习之createStore
date: 2018-02-26 16:57:58
tags:
  - 前端
  - redux
categories:
  - 学习笔记
---


之前在项目中，偶尔交互复杂的页面我们项目组会引入了Redux，快速迭代的过程中，我们在网上快速学习了用法，跑了demo，照葫芦画瓢地用进了项目里。当时自己的感受就是：一层套一层，这里要用这个，那里要结合那个，整个redux的引入，合着react-redux（项目用的react），用起来就是一种仿照的迷惘，只知道“要这样用”。现在前端圈可能就会有这种现象，结合的东西太多太复杂，当我们引入新东西的时候可能一下子需要学习的是这个新技术的整个生态圈，学起来反而有些吃力。这时候，我们需要做的是，抛开各种花哨的便利工具，告诉自己最简单的用法怎么整。

看了redux的源码之后，我可以这么说，createStore就是redux的精髓，刚开始，我们只需要这个函数做了什么，根本不用care别的。

<!-- more -->

---
# 简析
createStore的函数整个我们可以缩为以下： 
```js
export default function createStore(reducer, preloadedState, enhancer){
  //enhancer参数判断

  //reducer基础格式校验

  //dispatch({type: ActionTypes.INIT})

  //
  // 返回对象store 
  // {
  //  dispatch,
  //  subscribe,
  //  getState,
  //  replaceReducer,
  //  [$$observable]: observable
  // }
  //
}
```
## 函数签名
它接受三个参数： reducer, initialState, enhancer。 reducer是必传参数，initialState是初始化state状态，enhancer我们可以简单理解为是store的增强器， 返回一个增强功能的store。

## enhancer
```js
//当第二个参数preloadedState是函数且第三个函数没有传的时候，createStore自动将第二个参数默认为enhancer。
  if (typeof preloadedState === 'function' && typeof enhancer === 'undefined') {
    enhancer = preloadedState
    preloadedState = undefined
  }
  
//判断enhancer的格式
  if (typeof enhancer !== 'undefined') {
    if (typeof enhancer !== 'function') {
      throw new Error('Expected the enhancer to be a function.')
    }

//在enhancer有效的情况下，createStore会返回enhancer(createStore)(reducer, preloadedState)。
    return enhancer(createStore)(reducer, preloadedState)
  }

```
enhancer函数的结构一般如下：
```js
function enhancerCreator() {
  return createStore => (...args) => {
    //返回一个增强的store
  }
}
```
其实， 我们可以把middleware也可以理解为是一个enhancer。从applyMiddleware(...middlewares)的返回{...store, dispatch}可以看出， *middleware主要是针对dispatch的增强， 而咱们可以把enhancer理解为store所有暴露函数的增强*。增强的原则是：不能破坏原有的redux工作流。[【参考链接】](https://segmentfault.com/a/1190000012653724)


## reducer格式校验
在enhance无效的情况下，我们继续往下走。
对 reducer是否是函数做了一个基础校验。
```js
if (typeof reducer !== 'function') {
    throw new Error('Expected the reducer to be a function.')
  }
```

## 其他
createStore方法最终生成的store，可以概括为下图， 上面是内部内置变量， 下面是暴露的几个api。
<img src="/blogImg/store.png" alt="store" width="400px;">
那么接下来，来详细地看看这几个api的用法和原理。

---
# 本地变量
* currentReducer： store当前的reducer，支持通过store.replaceReducer热替换reducer
* currentState： store的当前状态，初始值为传进来的initialState
* isDispatching： 标志位，某个action是否处于正在分发的处理过程中
* currentListeners： 当前分发action中拥有的监听器
* nextListeners： 下一个action分发之前注册的监听器

---
# store的api
## getState
```js
 function getState() {
    return currentState
  }
```
很简单，就是取currentState， 初始值是传进来的initialState。

## dispatch
```js
function dispatch(action) {
  //检查 action 是否是一个原生js对象
    if (!isPlainObject(action)) {
      throw new Error(
        'Actions must be plain objects. ' +
        'Use custom middleware for async actions.'
      )
    }
  //检查 action 是否定义type， 所以我们的action都必须要有type的定义
    if (typeof action.type === 'undefined') {
      throw new Error(
        'Actions may not have an undefined "type" property. ' +
        'Have you misspelled a constant?'
      )
    }

  //判断是否处于别的action分发的过程中，避免分发死循环。
    if (isDispatching) {
      throw new Error('Reducers may not dispatch actions.')
    }

    try {
      //进入主流程
      //马上设置标志位
      isDispatching = true
      //更改state, currentReducer为我们传进来的reducer方法
      currentState = currentReducer(currentState, action)
    } finally {
      //分发完毕， 回归标志位
      isDispatching = false
    }

    //得到新的状态之后， 通知所有的监听器
    const listeners = currentListeners = nextListeners
    for (let i = 0; i < listeners.length; i++) {
      const listener = listeners[i]
      listener()
    }

    return action
  }
```

## subscribe
```js
function subscribe(listener) {
  //判断listener是否是函数
    if (typeof listener !== 'function') {
      throw new Error('Expected listener to be a function.')
    }

    let isSubscribed = true

  //检查current和next是否指向同一个数组对象，若是， 
  //则nextListeners = currentListeners.slice() 这也是数组拷贝的一种方式
    ensureCanMutateNextListeners() 
  //把当前listener塞进nextListeners
    nextListeners.push(listener)

  //返回一个函数， 为取消监提供了可能
    return function unsubscribe() {
      if (!isSubscribed) {
        return
      }

      isSubscribed = false

      ensureCanMutateNextListeners()
      const index = nextListeners.indexOf(listener)
      nextListeners.splice(index, 1)
    }
  }
```
整体就是利用内置变量 currentListeners和nextListeners。通过操作nextListeners数组来增加监听和取消监听（操作之前确保对currentListeners不造成负面影响），dispatch某个action时， 通过当前currentListeners来执行监听， 通知监听器发生了变化。

## replaceReducer
该方法主要用于reducer的热替换， 一般我们很少直接用到。
```js
function replaceReducer(nextReducer) {
    if (typeof nextReducer !== 'function') {
      throw new Error('Expected the nextReducer to be a function.')
    }

    currentReducer = nextReducer
    //热替换之后，重新回归最初始状态
    dispatch({ type: ActionTypes.INIT })
  }
```

## [$$observable]
留给 可观察/响应式库的接口。具体就不详细描述了， 有兴趣可以看看RxJS。

---
# 简易用法
一个简易的用法
```js
import { createStore } from 'redux';

const ActionTypes = {
  'INCREMENT': 'INCREMENT',
  'DECREMENT': 'DECREMENT'
};

function reducer(state = 0, action){
  switch(action.type){
    case ActionTypes.INCREMENT: return state+1;
    case ActionTypes.DECREMENT: return state-1;
    default: return state;
  }
}

const store = createStore(reducer);

//手动触发
store.dispatch({type: 'INCREMENT'}); //store: 1
store.dispatch({type: 'INCREMENT'}); //store: 2
store.dispatch({type: 'DECREMENT'}); //store: 1
```

<span style="color:red">全文基于redux 3.7.2</span>

