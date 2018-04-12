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
connect实现也是一步步柯里化的进程。从我们对connect的调用我们也可以看出来，`connect(mapStateToProps, mapActionsToProps)(MyComponent)`。
<img src="/blogImg/connect.png" alt="connect执行的流程图">
我们实际的最终调用其实就是对应图中`2`步骤，返回了`hoistStatics(Connect, WrappedComponent)`;
hoistStatics是什么？简单来说， 就是将WrappedComponent的非React属性拷贝到Connect上。具体可以参考[hoist-non-react-statics](https://github.com/mridgway/hoist-non-react-statics)。

那么Connect又是什么呢？？这才是我们的重点。
Connect是一个React组件。
```js
constructor(props, context) {
  super(props, context)

  this.version = version
  this.state = {}
  this.renderCount = 0
  //在connect api介绍中， 提供了两种模式， store可以以props的形式传给connect组件， 否则就默认从context中取。
  //一般情况下， 只有测试的时候可能会用第一种方式。
  this.store = props[storeKey] || context[storeKey]  //我们在这里拿到了store
  this.propsMode = Boolean(props[storeKey]) //模式标识

  this.setWrappedInstance = this.setWrappedInstance.bind(this)

  ......

  //一些初始化的东西
  this.initSelector()
  this.initSubscription()
}
```

在构造函数里面，进行了两项初始化, initSelector 和 initSubscription。
<b>initSelector</b>里面三行代码：
```js
//一个初始的selector， selectorFactory：内置的selector工厂， 接下来会细说。
const sourceSelector = selectorFactory(this.store.dispatch, selectorFactoryOptions)

//makeSelectorStateful返回了一个带有run方法的对象
this.selector = makeSelectorStateful(sourceSelector, this.store)
this.selector.run(this.props) //对props的真正初始化
```
那么接下来， 我们来搞懂关于Seletor的两件事~
第一件事：makeSelectorStateful到底做了什么妖？
```js
function makeSelectorStateful(sourceSelector, store) {
  //将sourceSelector执行返回结果挂在对象的props中，有了自己的局部状态， 在每一次run函数执行中方便对比。
  //哈哈包括了一层马上有了自己的状态维护， 一个可以借鉴的思想
  const selector = {
    run: function runComponentSelector(props) {
      try {
        //执行了selectorFactory中返回的selector, 进行真正的props合并
        const nextProps = sourceSelector(store.getState(), props) 
        if (nextProps !== selector.props || selector.error) { //当前selector中的props不同更新
          selector.shouldComponentUpdate = true 
          selector.props = nextProps
          selector.error = null
        }
      } catch (error) {
        selector.shouldComponentUpdate = true
        selector.error = error
      }
    }
  }

  return selector
}
```

第二件事：selectorFactory造出的sourceSelector是什么呢？
我用这幅图来简单概括一下：
<img src="/blogImg/selectorFactory.png" alt="selectorFactory执行的流程图">
那么我们先来说说， seletor.run()是怎么获取合并的props的呢？
selectorFactory有两种情况，主要是看connect api中传入的配置options对象中pure属性，默认true。 
true的时候执行pureFinalPropsSelectorFactory，返回函数pureFinalPropsSelector；
false执行impureFinalPropsSelectorFactory， 返回函数impureFinalPropsSelector。
不管执行哪一步，都是暂时返回一个函数finalPropsSeletor。
这个被返回的finalPropsSeletor最终呢就是在makeSelectorStateful中的run方法中执行的：
`const nextProps = sourceSelector(store.getState(), props) `还记得不？

那么我们再来看看这个finalPropsSeletor是怎么合并的props。
* pureFinalPropsSelector(nextState, nextOwnProps) 
  这里还要做一个判断， 是否是第一次执行(hasRunAtLeastOnce===false)，我刚来， 第一次走这个流程，ok，那就简单记录下就可以了。
  ```js
  function handleFirstCall(firstState, firstOwnProps) {
    state = firstState //记录初始state
    ownProps = firstOwnProps //记录C初始props
    stateProps = mapStateToProps(state, ownProps) //获取stateProps
    dispatchProps = mapDispatchToProps(dispatch, ownProps) //获取dispatchProps
    mergedProps = mergeProps(stateProps, dispatchProps, ownProps) //合并props
    hasRunAtLeastOnce = true //标识一下， 我来过了， 不是新人了
    return mergedProps //返回合并的props
  }

  注： mapStateToProps mapDispatchToProps mergeProps可以简单理解为 使用connnect api中传入的配参
  ```
  以后不是新人了， 该怎么走呢？
  ```js
  function handleSubsequentCalls(nextState, nextOwnProps) {
    const propsChanged = !areOwnPropsEqual(nextOwnProps, ownProps)
    const stateChanged = !areStatesEqual(nextState, state)
    state = nextState
    ownProps = nextOwnProps

    if (propsChanged && stateChanged) return handleNewPropsAndNewState()
    if (propsChanged) return handleNewProps()
    if (stateChanged) return handleNewState()
    return mergedProps
  }
  注：areOwnPropsEqual areStatesEqual 即connect api暴露出来的option方法中的属性值
  ```
  很明显， 去判断传入的props 和 当前store中的state是否变化， 去分别执行不同的合并函数。
  当props和state都发生了变化的时候：
  ```js
  function handleNewPropsAndNewState() {
    stateProps = mapStateToProps(state, ownProps) //缓存新的stateProps值

    if (mapDispatchToProps.dependsOnOwnProps) //dependsOnOwnProps主要是判断mapDispatchToProps是否依赖ownProps的值
      dispatchProps = mapDispatchToProps(dispatch, ownProps) //若是， 则更新dispatchProps

    mergedProps = mergeProps(stateProps, dispatchProps, ownProps)//合并最新的props并返回
    return mergedProps
  }
  ```

  仅当props发生变化时：
  ```js
  function handleNewProps() {
    //分别取判断mapStateToProps和mapDispatchToProps是否和ownProps有关
    if (mapStateToProps.dependsOnOwnProps)
      stateProps = mapStateToProps(state, ownProps)

    if (mapDispatchToProps.dependsOnOwnProps)
      dispatchProps = mapDispatchToProps(dispatch, ownProps)

    mergedProps = mergeProps(stateProps, dispatchProps, ownProps)
    return mergedProps
  }
  ```
  仅当state发生变化时：
  ```js
  function handleNewState() {
    const nextStateProps = mapStateToProps(state, ownProps)//缓存新的stateProps
    const statePropsChanged = !areStatePropsEqual(nextStateProps, stateProps)//判断stateProps与原来相比是否发生变化
    stateProps = nextStateProps
    
    if (statePropsChanged)//仅当stateProps发生变化的时候，才更新mergeProps
      mergedProps = mergeProps(stateProps, dispatchProps, ownProps)

    return mergedProps
  }
  ```
  当pure设置为true的时候， 我们可以根据options中设定的相等判断条件（areStatePropsEqual、 areOwnPropsEqual、areStatesEqual）去判断对应的变量是否发生变化， 从而决定是否更新props。这就是其中的性能优化的重要点。

* impureFinalPropsSelector(nextState, nextOwnProps)
  理解了pureFinalPropsSelector之后， impureFinalPropsSelector很好理解， 不做任何比较，直接返回了mergeProps。
  ```js
  function impureFinalPropsSelector(state, ownProps) {
    return mergeProps(
      mapStateToProps(state, ownProps),
      mapDispatchToProps(dispatch, ownProps),
      ownProps
    )
  }
  ```

讲了一大通， 终于把Selector的两件事讲完了。那么这个selector暴露出的两个参数供我们Connect来使用：
1、selector.shouldComponentUpdate 是否应该更新组件了
2、selector.run() 获取nextProps

搞明白了Selector之后， 我们再来看看<b>initSubscription</b>函数里做了什么。
```js
...
//我有一个监听者 监听store中的变化， 有变化时就执行onStateChange
//具体判断逻辑我们可以不看， 我们不考虑复杂的store通过props传入的情况
this.subscription = new Subscription(this.store, parentSub, this.onStateChange.bind(this))
...
//通知子组件监听的方法
this.notifyNestedSubs = this.subscription.notifyNestedSubs.bind(this.subscription)
```
讲别的之前呢，Subscription先了解一下？
[Subscription类源码链接](https://github.com/reactjs/react-redux/blob/master/src/utils/Subscription.js)
同时呢，这里有一篇比较好的博客[庖丁解牛React-Redux(一): connectAdvanced](https://github.com/MrErHu/blog/issues/17)里面讲述了Subscription的具体作用。

onStateChange里面干了什么呢？
onStateChange是store发生变化的回调函数。
```js
onStateChange() {
  this.selector.run(this.props)

  if (!this.selector.shouldComponentUpdate) {
    //不需要更新 近需要通知子组件更新
    this.notifyNestedSubs()
  } else {
    //自身更新完成之后， 通知子组件更新
    this.componentDidUpdate = this.notifyNestedSubsOnComponentDidUpdate
    this.setState(dummyState) //dummyState即{} 通过这种方式 强制connect组件自身更新
  }
}

notifyNestedSubsOnComponentDidUpdate() {
  this.componentDidUpdate = undefined
  this.notifyNestedSubs()
}
```

Connect类构造函数里面做的事情我们大概有了了解， 本身Connect就是一个React组件， 那么我们再来看看这个组件本身的生命周期函数。
```js
render() {
  const selector = this.selector
  //更新一波之后， 将更新flag设置为false
  selector.shouldComponentUpdate = false 

  if (selector.error) {
    throw selector.error
  } else {
    //开始渲染 addExtraProps就不细讲了 顾名思义 就是添加一些额外的props（版本号 实例等等）
    return createElement(WrappedComponent, this.addExtraProps(selector.props))
  }
}

componentDidMount() {
  if (!shouldHandleStateChanges) return
  //开始监听变化
  this.subscription.trySubscribe()
  this.selector.run(this.props)
  if (this.selector.shouldComponentUpdate) this.forceUpdate()
}

componentWillReceiveProps(nextProps) {
  this.selector.run(nextProps)
}

shouldComponentUpdate() {
  return this.selector.shouldComponentUpdate
}

componentWillUnmount() {
  if (this.subscription) this.subscription.tryUnsubscribe()
  this.subscription = null
  this.notifyNestedSubs = noop
  this.store = null
  this.selector.run = noop
  this.selector.shouldComponentUpdate = false
}
```
所有的生命周期函数还是利用了this.selector.run去执行获取新的props， this.selector.shouldComponentUpdate去决定该不该更新。

乱七八糟地讲述完了自己看源码的整个经历。可能比较晦涩， 希望自己能反复看看自己的文章， 多看多改，加强博客的撰写能力。

<span style="color:red">全文基于react-redux 5.0.6</span>




