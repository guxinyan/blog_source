---
title: React Component  Lifecycle
date: 2017-02-23 16:12:58
tags:
    - React
    - 前端
categories:
    - 总结
---

年后开始真正开始接手公司的wap项目了，从之前的管理后台常驻人员将眼界开向了用户端。用了半年多的Angular，再重新拾起react做需求的时候，就遇到了声明周期这个绊脚石，决心整理一波~
> React通过React.createClass(object)方法来创建组件，每个组件都是一个状态机。React为每个组件提供了生命周期钩子函数去响应不同的时刻——创建时、存在期以及销毁期。
那么接下来，我们从以下几个一一探索钩子函数的奥妙。 

---
# 一、初始化
首先，每个组件渲染都有一次初始化（也称组件挂载），在这个阶段，会`依次`执行下图中的函数。这些函数（除render外），`在组件的声明周期内，只会(only)执行一次`。
<img src="/blogImg/initialization_lifecycle.png" alt="初始化" style="width:60%">
<!--more-->
下面是一个很简单的组件：
```js
var A = React.createClass({
    getDefaultProps: function(){
        return {};
    },
    getInitialState: function(){
        return {};
    },
    componentWillMount: function(){
        console.log('component will mount');
    },
    componentDidMount: function(){
        console.log('component did mount');
    },
    /**
    * 在render函数中，假设不想返回任何dom结构，可以返回 null 或者 false
    *               返回dom结构的时候，只能返回一个顶级元素（single child component）
    */
    render: function(){
        console.log('render');
        return false;
    }
})

ReactDOM.render(
  <A/>,
  document.getElementById('id0')
)

这时候控制台的输出为：
"component will mount"
"render"
"component did mount"
```

---
## componentWillMount
componentWillMount是render前最后一次修改state的机会, 在此函数对state的修改完毕之后触发组件的Render。
```js
var A = React.createClass({
    getInitialState: function(){
        return {num:1};
    },
    componentWillMount: function(){
        this.setState({
            num:2
        });
        console.log('component will mount:'+this.state.num);
    },
    componentDidMount: function(){
        console.log('component did mount:'+this.state.num);
    },
    render: function(){
        console.log('render:'+this.state.num);
        return false;
    }  
})

控制台输出：
"component will mount:1"
"render:2"
"component did mount:2"
```
但是很奇怪的，在componentWillMount函数中输出的num值为1，而并不是我们所以为的2。其实，
> setState并不是同步的， 我们可以理解为异步。
```js
setState(nextState, callback)
```
callback会在新的state设置成功之后执行。因此，如果我们想在state更新之后再获取,可以在callback中获取。我们将上面代码改成：
```js
var A = React.createClass({
    getInitialState: function(){
        return {num:1};
    },
    componentWillMount: function(){
        this.setState({
            num:2
        },function(){
        console.log("callback:"+this.state.num);
        });
        console.log('component will mount:'+this.state.num);
    },
    componentDidMount: function(){
        console.log('component did mount:'+this.state.num);
    },
    render: function(){
        console.log('render:'+this.state.num);
        return false;
    }  
})

控制台输出：
"component will mount:1"
"render:2"
"component did mount:2"
"callback:2"
```

---
## componentDidMount
componentDidMount中修改state会触发re-rendering。在这个函数中，我们也可以进行DOM操作。
```js
var A = React.createClass({
    getInitialState: function(){
        return {num:1};
    },
    componentWillMount: function(){
        console.log('component will mount:'+this.state.num);
    },
    componentDidMount: function(){
        this.setState({
            num:3
        })
        console.log('component did mount:'+this.state.num);
    },
    render: function(){
        console.log('render:'+this.state.num);
        return false;
    }  
})

控制台输出：
"component will mount:1"
"render:1"
"component did mount:1"
"render:3"
```

然后我做了个小小的实验，将代码进行了小小的修改，将state的num在render函数中渲染出来：
```js
var A = React.createClass({
    getInitialState: function(){
        return {num:1};
    },
    componentWillMount: function(){
        console.log('component will mount:'+this.state.num);
    },
    componentDidMount: function(){
        this.setState({
            num:3
        })
        console.log('component did mount:'+this.state.num);
    },
    render: function(){
        console.log('render:'+this.state.num);
        return <div>{this.state.num}</div>;
    }  
})
```
肉眼无法发现dom结构从1-3的变化，直接我们看到的就是3。

---
# 二、state改变
更新方法只会在组件初始化渲染完成后且触发了重新渲染的条件才会执行，按照以下顺序执行图中函数。
<img src="/blogImg/state_changes_lifecycle.png" alt="state_changes" style="width:60%">

按照上面代码，我们在componentDidMount函数中对state进行了修改：
```js
var A = React.createClass({
  getInitialState: function(){
    return {num:1};
  },
  componentDidMount: function(){
     this.setState({
      num:3
    })
  },
  shouldComponentUpdate: function(){
    console.log("shouldComponentUpdate:"+this.state.num);
    return true;
  },
  componentWillUpdate: function(){
    console.log("componentWillUpdate:"+this.state.num);
  },
  componentDidUpdate: function(){
    console.log("componentDidUpdate:"+this.state.num);
  },
  render: function(){
     console.log('render:'+this.state.num);
    return <div>{this.state.num}</div>;
  }  
})

控制台输出：
"render:1"
"shouldComponentUpdate:1"
"componentWillUpdate:1"
"render:3"
"componentDidUpdate:3"
```
个人觉得很神奇的是，在shouldComponentUpdate和componentWillUpdate取到的num值都是1。于是，我很幼稚地刷了很多次，在shouldComponentUpdate和componentWillUpdate中永远取到的this.state.num一直都是1，而在render中总能正确获取2。大胆假设一下，react让setState这个函数的行为结果一定完成在componentWillUpdate后，render函数前。根据前面的理解，将setState理解为一个异步行为，我觉得有点偏差，setState这个行为是一个可以预测时间（可以决定在什么时候执行完成）的函数。因此，大胆假设一下，
> 将setState函数理解成是一个“人工异步”。

---
## shouldComponentUpdate
shouldComponentUpdate函数决定是否re-rendering，默认返回true。
```js
var A = React.createClass({
  getInitialState: function(){
    return {num:1};
  },
  componentDidMount: function(){
     this.setState({
      num:1
    })
  },
  componentWillUpdate: function(){
    console.log("componentWillUpdate:"+this.state.num);
  },
  componentDidUpdate: function(){
    console.log("componentDidUpdate:"+this.state.num);
  },
  render: function(){
     console.log('render:'+this.state.num);
    return <div>{this.state.num}</div>;
  }  
})

控制台输出：
"render:1"
"componentWillUpdate:1"
"render:1"
"componentDidUpdate:1"
```
在React.createClass传入的对象中没用定义shouldComponentUpdate时，默认shouldComponentUpdate钩子函数返回true。令我失望的，原以为react会在此函数中做一个基础的对比，state的`无效变化`不会导致re-rendering。然而，事实是，只要调用了setState，不管state是否“真正”有变化，shouldComponentUpdate默认都返回true。在这样的情况，也会导致一次重复渲染。因此，`建议在shouldComponentUpdate函数中，对state和props的变化进行一次对比，避免重复渲染的情况`。
当传入对象定义了shouldComponentUpdate函数时，必须返回一个boolean（没有返回值的情况下默认返回false）。那么，在shouldComponentUpdate返回false的时候，又会发生什么呢。
```js
var A = React.createClass({
  getInitialState: function(){
    return {num:1};
  },
  componentDidMount: function(){
     this.setState({
      num:2
    })
  },
  shouldComponentUpdate: function(){
    console.log("shouldComponentUpdate:"+this.state.num);
    return false;
  },
  componentWillUpdate: function(){
    console.log("componentWillUpdate:"+this.state.num);
  },
  componentDidUpdate: function(){
    console.log("componentDidUpdate:"+this.state.num);
  },
  handleClick: function(){
    console.log(this.state.num);
  },
  render: function(){
     console.log('render:'+this.state.num);
    return <button onClick={this.handleClick}>{this.state.num}</button>;
  }  
})

控制台输出：
"render:1"
"shouldComponentUpdate:1"

点击button之后，输出：2
```
shouldComponentUpdate返回false之后，componentWillUpdate、render、componentDidUpdate将都不会执行。
但是这样的情况之下，我们可以发现，不管shouldComponentUpdate返回了什么，组件的state还是发生了变化。如果这时候，我们再在操作函数中对state进行一些处理，可能获取的不是我们想要的state。`因此，建议，尽量不要在state有变化的时候在shouldComponentUpdate函数中返回false。`

---
## componentWillUpdate
> Note that you cannot call this.setState() here. If you need to update state in response to a prop change, use componentWillReceiveProps() instead.
官网上是这么阐述的，但是实际情况，我们可以在这个函数中setState，并且，与在componentWillMount中改变state中不一样的是，state不仅更改成功了，还触发了re-rendering，因此导致浏览器进入了一种死循环。
> 所以，在componentWillUpdate中`不要`setState！！！

---
# 三、props改变
<img src="/blogImg/props_change_lifecycle.png" alt="props_change" style="width:60%">
```js
var A = React.createClass({
  getInitialState: function(){
    return {
      num: this.props.num || 0
    }
  },
  componentWillMount: function(){
    console.log('componentWillMount');
  },
  componentWillUpdate: function(){
    console.log('componentWillUpdate');
  },
  render: function(){
     console.log('render:'+this.state.num);
    return (<div>{this.state.num}</div>);
  }  
})

var B = React.createClass({
  getInitialState: function(){
    return {num:1};
  },
  componentDidMount: function(){
    this.setState({
      num:2
    })
  },
  render:function(){
    return <A />
  }
})

控制台输出：
componentWillMount"
"render:0"
"componentWillUpdate"
"render:0"
```
非常有意思的是，子组件A渲染了两遍。当父组件B的state发生变化时，会触发一次render函数。此时，不管子组件A的state有没有发生变化。子组件A发生了一次update（默认情况下）。假设子组件是比较纯的情况之下，建议在子组件中的shouldComponentUpdate去判断逻辑，避免重复渲染。当然，刚刚询问了身边的大佬，目前还没有这个渲染瓶颈，在state和props比较复杂的情况之下，可以忽略这些。

---
## componentWillReceiveProps
这是一个非常有用的钩子函数，我们一般会在这里判断props的变化从而去更新子组件的state，是父子组件通信的常用手段。
```js
var A = React.createClass({
  getInitialState: function(){
    return {
      num: this.props.num || 0
    }
  },
  componentWillReceiveProps: function(nextProps){
    if(this.props.num !== nextProps.num){
      this.setState({
        num: nextProps.num
      })
    }
  },
  render: function(){
     console.log('render:'+this.state.num);
    return (<div>{this.state.num}</div>);
  }  
})

var B = React.createClass({
  getInitialState: function(){
    return {num:1};
  },
  componentDidMount: function(){
    this.setState({
      num:2
    })
  },
  render:function(){
    return <A num={this.state.num}/>
  }
})

控制台输出：
"render:1"
"render:2"
```
我们发现，在父组件B更新状态时，子组件A随着B组件只更新了两次。在A组件中的componentWillReceiveProps调用setState并没有触发第三次render，并且直接更新了第二次render的num值。
> componentWillReceiveProps方法可以作为React在props传入后，渲染之前setState的机会，不会导致二次渲染。

---
# 四、移除
<img src="/blogImg/unmount.png" alt="unmount_lifecycle" style="width:60%">


