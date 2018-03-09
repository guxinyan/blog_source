---
title: Redux源码学习之提供的那些便利api
date: 2018-03-01 15:32:09
tags:
  - 前端
  - redux
categories:
  - 学习笔记
---

本文我们只讲redux中除了createStore的那些api， 我将这些api称作redux中的便利工具，是为了createStore更好的工作。如果对createStore这个核心api更感兴趣的话， 请看这篇[【Redux源码学习之createStore】](/2018/02/26/Redux源码学习之createStore/#more)

---
# compose
compose是函数式编程中的一个概念，通过合成多个函数，最终生成一个把接收函数*从右到左*合成的超级厉害版的函数。

<!-- more -->

> 用法： compose(...func);
举例： compose(f1, f2, f3, f4)  =>  f1(f2(f3(f4(..args))))

看到这个用法，自己可以动手试试compose的实现。
```js
function compose(...funcs){
  return function(...args){
    return funcs.reduceRight(function(current, fn){
      return fn(current);
    }, args)
  };
}
```
这是redux中的源码实现，具体就不赘述了。
```js
export default function compose(...funcs) {
  //注意参数的检查
  if (funcs.length === 0) {
    return arg => arg
  }

  if (funcs.length === 1) {
    return funcs[0]
  }

  return funcs.reduce(
    (a, b) => (...args) => a(b(...args))
  )
}
```

---
# applyMiddleware
> It provides a third-party extension point between dispatching an action, and the moment it reaches the reducer.

这是官网github上对middleware的描述， 主要触发的节点在dispatch action之后， reducer接收action之前。middleware对在不影响redux原有主流程的情况下， 对dispatch方法进行了强化，为异步接收， 延迟分发等等行为提供了可能性。

《深入react技术栈》上有这么一张经典的图，可以形象地概括middleware的理念：
<img src="/blogImg/middleware in redux.png" alt="middleware示意图" width="600px;">
通过串联不同的middleware实现对dispatch功能的增强，每一个midlleware实现不同的业务逻辑， 达到自由组合， 随意插拔的插件机制。
在没有中间件的情况下， action创建函数函数只能同步地返回一个action对象。所以中间件最常用的功能就是定制action的返回格式。

这里以Reduc Thunk这个中间的用法为例， 来理解一下所谓的中间件的用法。Redux Thunk的源码如下：
```js
const thunk = store => next => action => 
  typeof action === 'function' ? 
    action(store.dispatch, store.getState) : next(action)
```

Thunk会判断传入的action是否是一个函数，假设是函数， 则执行action, 传入参数为store的dispatch和getState方法，这也决定了异步action的写法，需要返回一个入参为dispatch和getState的函数，当然这是后话；假设不是函数，则继续传递action到下一个middleware。
middleware的设计就是一个层层包裹的匿名函数，也就是函数式编程中的柯里化(currying)。

这是结合middleware的两种用法：
* 用法一：
`var newStoreCreator = applyMiddleware(Thunk)(createStore);`
然后newStoreCreator就按照createStore的用法继续

* 用法二：
`var store = createStore(reducer, applyMiddleware(Thunk));`
这里就是把applyMiddleware(Thunk)当成enhancer来使用了
有兴趣可以看看[【Redux源码学习之createStore】](/2018/02/26/Redux源码学习之createStore/#more)

```
我们首先创建一个普通的store:
```js
var store = applyMiddleware(...middleware)(createStore)(reducer, initState, enhancer);
```

那么接下来， 我们来看看applyMiddleware的简单而精炼的源码：
```js
export default function applyMiddleware(...middlewares) {
  return (createStore) => (reducer, preloadedState, enhancer) => {
    //利用传入的参数， 创建了store
    const store = createStore(reducer, preloadedState, enhancer)
    let dispatch = store.dispatch
    let chain = []
    //然后将middleware要用的getState和dispatch方法赋值给middlewareAPI（一个新的store）
    const middlewareAPI = {
      getState: store.getState,
      dispatch: (...args) => dispatch(...args)
    }
    //让每个middleware都带这个这个参数执行一遍，回顾一下中间件的函数签名，这时候我们获得的是以
    // next => action => {....}  这样的函数组成的数组
    //每个函数都能访问middlewareAPI这样的store。
    chain = middlewares.map(middleware => middleware(middlewareAPI))
    //这行代码， 就是合成middleware最重要的一步，
    //将所有的middlewares以及初始store的dispatch方法合成
    //形成最终的dispatch方法
    dispatch = compose(...chain)(store.dispatch)

    //将改造后的dispatch方法暴露出去
    return {
      ...store,
      dispatch
    }
  }
}
```

有一个比较有意思的地方
```js
const middlewareAPI = {
  getState: store.getState,
  dispatch: (...args) => dispatch(...args)
}
```
这里的dispatch用匿名函数包了一层， 而且传入的是dispatch，而不是store.dispatch。
为什么呢？因为要保证传给每个middleware的dispatch是最新的，最终合成的。
怎么办的的呢？因为匿名函数行成了闭包，对外层dispatch这个变量一直保持着引用关系， 也保证了dispatch不会被销毁。同时呢，dispatch在后面呢被更新为了`compose(...chain)(store.dispatch)`，也就是我们最终改造的dispatch方法。
这其中呢涉及了一点作用域那些事，哈哈， 想起了以前写的一篇博客，有兴趣也可以看看[【关于js作用域那些事】](/2017/01/12/关于js作用域那些事/#more)。


---
# combineReducers
用法：
```js
const finalReducer = Redux.combineReducers({
  reducer0: reducer0,
  reducer1: reducer1,
  reducer2: reducer2
});
```
> 将独立的reducers整合成一个大家庭，在需要处理处理数据比较多的情况下，比较容易梳理， 并不一定要使用。

源码大概就是：
* 刚开始一些规范性检查：
1.参数检测
2.每个Reducer是否设定初始值
* 将传入的Reducer整体对象赋值通过key遍历， 存储在一个finalReducers对象中
* 返回一个函数（其实就是一个整体Reducer）
1.参数为state, action
2.遍历finalReducers中的所有reducer， 触发对应的action， 复制给新的state。
```js
 let hasChanged = false
    const nextState = {}
    for (let i = 0; i < finalReducerKeys.length; i++) {
      const key = finalReducerKeys[i]
      const reducer = finalReducers[key]
      const previousStateForKey = state[key]
      const nextStateForKey = reducer(previousStateForKey, action)
      if (typeof nextStateForKey === 'undefined') {
        const errorMessage = getUndefinedStateErrorMessage(key, action)
        throw new Error(errorMessage)
      }
      nextState[key] = nextStateForKey
      hasChanged = hasChanged || nextStateForKey !== previousStateForKey  
      //个人觉得这个对比基本对于小reducer返回是对象的没有任何作用
      //由于对象引用经过Reducer,一定会改变
    }
    return hasChanged ? nextState : state
  }
```

> 所以用了combineReducers这个方法时， 我们以为只调用了对应state的reducer， 其实并不是， dispatch action调用了所有的 reducer。



---
# bindActionCreators
用法：
```js
const bindActions = Redux.bindActionCreators(Actions, store.dispatch);
//然后直接出发bindActions的对应方法即可
```
>  处理传入的action, 返回一个拥有同样key, 但是每个方法用dispatch包括的对象。

这样，触发方法的时候，不到到处都可以去写store.dispatch(action), 只要直接调用action即可。

源码我就不贴了， 有兴趣可以去官网看看。大概就是， 遍历对象的key， 然后赋值一个新的对象， 并将所有的方法用dispatch包裹一遍， 返回这个新对象。


<span style="color:red">全文基于redux 3.7.2</span>

