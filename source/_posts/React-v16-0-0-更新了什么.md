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
* React elements。
* Booleans or null。
* Arrays(v16.0.0) and fragments(v16.2.0)。
  ```js
  render() {
    return [
      <li key="A">First item</li>,
      <li key="B">Second item</li>,
      <li key="C">Third item</li>,
    ];
  }
  需要给每个item添加key。
  ```
  ```js
  const Fragment = React.Fragment;

  render(){
      return (
        <Fragment>
          <ChildA />
          <ChildB />
          <ChildC />
        </Fragment>
      );
  }

  存在一种缩写方式
  render(){
      return (
        <>
          <ChildA />
          <ChildB />
          <ChildC />
        </>
      );
  }
  但是，缩写方式的标签不能有属性值。
  ```
* Portals(v16.0.0)。
 具体看下文的portal。
* String and numbers(v16.0.0)。
 可以直接返回text node了。


---
# portal
*v16.0.0更新*
## 用法
```js
ReactDOM.createPortal(child, container)
child 见上面的render 返回
container dom元素，作为容器
```
  
## 主要应用场景
最常用的场景就是弹窗Modal。

### 以前使用方式
在以前我的概念里，弹窗的使用场景有两种：
* 触发了某种条件，产生了弹窗。
[【我是一个小栗子】](https://codepen.io/guxinyan/pen/MBGNxz?editors=1011)
dom元素分离。用createLayer 封装了蒙层。弹窗是在handle函数里面直接调用的，弹窗什么时候关，由弹窗自己控制。父组件与弹窗的交互，通过参数option传递。参数无法更新。
* 通过父组件state中的弹窗控制参数showAlert控制弹窗，弹窗中的数据和父组件息息相关，并随父组件数据改变，从而改变。
[【又来一个小栗子】](https://codepen.io/guxinyan/pen/BPPKvV?editors=1011)
dom元素嵌套在父元素之中。


### 现在使用方式
```js
const modalRoot = document.getElementById('modal-root');

class Modal extends React.Component {
  constructor(props) {
    super(props);
    this.el = document.createElement('div');
  }

  componentDidMount() {
    modalRoot.appendChild(this.el);
  }

  componentWillUnmount() {
    modalRoot.removeChild(this.el);
  }

  render() {
    return ReactDOM.createPortal(
      this.props.children,
      this.el,
    );
  }
}

class Parent extends React.Component {
  constructor(props){
    super(props);
  }
  
  render() {
    return (
      <div>
          ...
          <Modal>{填一些我们想要的内容}</Modal>
          ...
      </div>
    );
  }
}
```
和容错组件一个概念，把弹窗这件事，提出来，封成一个公用组件。是以前两种弹窗使用方式的结合体。在dom结构上，脱离了父组件的元素嵌套，但又可受父组件的state控制。

**支持事件冒泡**：
现在的这种弹窗形式，在弹窗上的点击事件，是可以冒泡到Parent上去捕获的。这是比较神奇的点，dom元素逃离了原来的结构，但是事件依旧可以捕获。具体你可以写个小栗子试一下~


---
# 标签自定义属性
## before 16.0.0
react 维护了一份支持属性的白名单，遇到不在白名单中的属性，会自动忽略。这种当时，让属性更新起来比较麻烦，滞后。
## after 16.0.0
* 容许自定义标签存在
* 标准的attr的属性，依然要遵守驼峰写法

---
# 新的Context API
*v16.3.0更新*
Context提供了一种在组件树中传递数据的新的方式，而不用依赖于组件的props层层传递。
在v16.3之前，react就提供了Context的api，只不过不怎么建议使用。

## 什么时候该使用
可以被认为全局的配置，我们可以放在Context中使用，比如：用户认证信息，框架主题风， 还有我们比较熟悉的redux中的store。balabla。
[用法小demo](https://codepen.io/guxinyan/pen/WKvqxB)

## api
### React.createContext
```js
const {Provider, Consumer} = React.createContext(defaultValue);
```
Provider和Consumer是配对使用的
Consumer只有在组件树中找不到对应的Provider时，才会读取defaultValue。
[show the code](https://codepen.io/guxinyan/pen/MBwMxJ)

### Provider
```js
<Provider value={/* some value */}>
```
* 一个Provider可以对应多个Consumer。
* Provider可以多层嵌套。内层可以覆盖外层的值。

### Consumer
```js
<Consumer>
  {value => /* render something based on the context value */}
</Consumer>
```
Consumer子元素必须是一个函数，函数返回React node（同render要求）
[examples](https://reactjs.org/docs/context.html#examples)

---
# 新的ref API
## createRef API
** 什么时候使用ref **
三种应用场景
* 获取焦点，文本选择，媒体播放
* 触发动画
* 集成第三方库的时候

** 新的api用法 **
```js
class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    this.myRef = React.createRef();
  }
  render() {
    return <div ref={this.myRef} />;
  }
}
```
```js
const node = this.myRef.current;
```
node的值取决于ref设置在什么哪里：
* ref作为HTML element的属性值设置，则node拿到的是DOM element
* ref作为自定义组件的属性值设置，则node拿到的是组件的实例

**functional components 不能使用ref属性，因为functional components没有实例， 但是在functional components组件内部可以使用Ref**

## forwardRef API
Ref forwarding提供了一种新方式，能将ref传递给组件的子元素。
简单小栗子：
```js
const FancyButton = React.forwardRef((props, ref) => (
  <button ref={ref} className="FancyButton">
    {props.children}
  </button>
));

const ref = React.createRef();
<FancyButton ref={ref}>Click me!</FancyButton>;

const BtnNode = ref.current;
```
FancyButton组件必须要用React.forwardRef(props, ref)包一层。

---
# 严格模式组件
在v16.3.0 React新增了一种严格模式组件，可以帮助我们更好的预防bug。当然，只影响开发环境。
用法有点类似Fragment
```js
function ExampleApplication() {
  return (
    <div>
      <React.StrictMode>
        <div>
          <ComponentOne />
          <ComponentTwo />
        </div>
      </React.StrictMode>
    </div>
  );
}
```
React只会检测<StrictMode></StrictMode>包含的内容。
严格模式下，
* 鉴定不安全的生命周期函数
* 对ref的字符串用法进行警告处理
* 对老的context api的用法进行警告处理
* 检测不可预期的副作用（结合上文的异步渲染，生命周期中的一些操作导致的问题）

---
# 更强的服务端渲染
* v16之前严格对比客户端render函数执行与服务端渲染出来的字符。v16放宽了这个限制。
* 新的函数：ReactDOM.hydrate：不要求服务器端和客户端之间产生的dom完全一模一样，寻找最大重合dom。
* 不再到处检查process.env.NODE_ENV=production，只在入口检查
* v16之后可以尝试不用React做服务端渲染
* 传说性能的改进，是以前的3倍！
* html的流式推送方式

---
# 其他
* Reduced file size
* MIT licensed
* New core architecture
