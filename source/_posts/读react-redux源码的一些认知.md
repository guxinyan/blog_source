---
title: 读react-redux源码的一些认知
date: 2018-03-26 17:46:38
tags:
  - 前端
  - redux
  - react
categories:
  - 学习笔记
---

为什么我们在react项目中， 用到redux的时候，常常会选择react-redux。react-redux到底为我们做了什么便利的事情呢？？？

---
# 不用react-redux时的手动连接
假设有一个组件 <Counter/>，组件现在目前就支持简单的加法（onIncrement），减法（onDecrement）。
首先我们提前做好把redux的基础准备工作做好。
```js
//actions
const Actions = {
  onIncrement: {type: 'INCREMENT'},
  onDecrement: {type: 'DECREMENT'}
}
//reducer
const Reducers = function(state, action){
  switch(action.type){
    case 'INCREMENT': return state+1;
    case 'DECREMENT': return state-1;
    default: return state;
  }
}
const store = createStore(Reducers);
```
<!-- more -->

接下来， 我们该怎么办呢？
我们需要解决的问题是什么？
* 告诉组件我现在的状态（store.getState）是这个样子的， 你该更新就更新吧。
* 子组件因为一些用户操作，导致了状态应该有所变化，通知store更新state。

redux的第一件事，告诉组件我现在的state值。
将state传给最大的父组件， 由父组件一层一层往里传。如下：
```js
function render(){
  ReactDOM.render(
    <Counter {...store.getState()}/>,
    rootEl  
  )
}
//当state变化时通知重新渲染
store.subscribe(render);
```
但是采用这种方式的时候， 跟在最外层父组件上用内部state实现，其实也没什么区别。我们不就是觉得应用复杂了， 层层层传觉得麻烦才考虑用Redux的吗？这样我们为什么还要用Redux呢？？？对，我们不能走老路。

这时候脑子里出现了第二种方法，利用context。偷偷借鉴下Provider的实现原理， 咱们来实现一下。
在最外层的组件中， 将store定义为子组件的context，这样所有的子组件都可以获取到。
```js
class ParentComponent extends React.Component{
  construtor(props, context){
    super(props, context);
    this.store = props.store;
  }
  getChildContext(){
    return {
      store: this.store
    }
  }
  ...
}
```
子组件怎么获取store呢？
```js
class ChildComponent extends React.Componet{
   construtor(props){
    super(props);
    this.state = {};
    this.getCurrentState = this.getCurrentState.bind(this)
  }

  componentWillMout(){
    const { store } = this.context;
    this.getCurrentState();
    store.subscribe( () => this.getCurrentState());
  }

  getCurrentState(){
    const { store } = this.context;
    const state = store.getState();
    this.setState({
      ...state
    })
  }

  ...
}
```
在子组件中，去获取context中的store，放到自身组件的state中，并监听store的变化，实现state的实时更新。
但是， 这样重复的步骤我们要在每一个需要用到状态的子组件中都来一遍？？？？这也太麻烦了吧，感觉也不是那么爽。

第二件事， 子组件因为一些用户操作，导致了状态应该有所变化，通知store更新state
将actions通过子组件的容器组件一层层往下传。
```js
const bindActions = Redux.bindActionCreators(Actions, store.dispatch);

<Container  {...bindActions}/>
```
同样的， 我们需要在所有子组件上重复进行这两项操作。

以上就是手动链接的一些简单幻想，总结一下就是：
* 麻烦。
  手工当然要比自动化织布机麻烦地多。react-redux为我们做了dispatch , subscribe, getState三大重要的步骤。
> 手动连接： component => dispatch(action) => reducer => store.subscribe => store.getState => component
  react-redux: component => ActionCreators => reducer => component
* 没有优化性能
  手动连接，任何state的变化都会导致整个组件的重新渲染。

所以我们建议用react-redux去做react和redux的`桥梁`。

---
# react-redux怎么用
[献上官方链接](https://github.com/reactjs/react-redux/blob/master/docs/api.md#api)

---
# react-redux怎么实现的
## Provider
Provider的实现其实并不用多说， 就是利用context。在使用的时候， 用Provider包裹在最外面。子组件去获取context中的数据， 具体怎么获取呢？那我们就要仔细看看connenct做了什么了。

## connect
connect实现也是一步步柯里化的进程。



