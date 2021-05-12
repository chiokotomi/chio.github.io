---
title: react规模
date: 2021-05-12 11:53:01
tags:
---


# 核心概念
## props & state & 生命周期
### props：由父组件传入，固定的数据，子组件不可修改
### state
组件内维护的动态数据，使ui具备交互功能

- 检查相应数据是否属于 state：
    - 该数据是否是由父组件通过 props 传递而来的？如果是，那它应该不是 state。
    - 该数据是否随时间的推移而保持不变？如果是，那它应该也不是 state。
    - 你能否根据其他 state 或 props 计算出该数据的值？如果是，那它也不是 state。
```js
class Clock extends React.Component {
 // 不写constructor就没有state
 // 写了constructor一定要super，否贼thhis不会指向组件自己
 // 当想在constructor中使用this.props的时候，super需要加入(props)
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```
- 更新state
直接修改`this.state.xxx = 'xxx'`不会重新渲染组件，使用`this.setState({xxx: 'xxx'})`才会触发渲染
state的更新可能是异步的


### 生命周期：`componentWillUnmount`、`componentDidMount`

## key
在兄弟节点之间必须唯一

## 状态提升
多个组件共享的state向上移动到父组件，数据源唯一，自上而下的数据流
## 组合 vs 继承
> 推荐使用组合来实现组件间的代码复用

# 代码分割 与 懒加载

当 Webpack 解析到该语法时，会自动进行代码分割
```js
import { add } from './math';

console.log(add(16, 26));

//代码分割

import("./math").then(math => {
  console.log(math.add(16, 26));
});
```

`React.lazy()`与`Suspense`组件实现懒加载
```js
import React, { Suspense } from 'react';

const OtherComponent = React.lazy(() => import('./OtherComponent'));
const AnotherComponent = React.lazy(() => import('./AnotherComponent'));

function MyComponent() {
  return (
    <div>
      <Suspense fallback={<div>Loading...</div>}>
        <section>
          <OtherComponent />
          <AnotherComponent />
        </section>
      </Suspense>
    </div>
  );
}
```
# diffing算法

state或props更新时，相同的`render()`方法返回不同的元素树，react需要判断如何高效的更新UI
当对比两棵树时，React 首先比较两棵树的根节点。不同类型的根节点元素会有不同的形态。
- 对比不同类型的元素
  当根节点为不同类型的元素时，React 会拆卸原有的树并且建立起新的树，从 <Button> 变成 <div> 会触发一个完整的重建流程，当卸载一棵树时，对应的 DOM 节点也会被销毁。组件实例将执行 componentWillUnmount() 方法
  
- 对比同一类型的元素
  当对比两个相同类型的 React 元素时，React 会保留 DOM 节点，仅比对及更新有改变的属性。
  ```js
  <div className="before" title="stuff"></div>

  <div className="after" title="stuff"></div>
  ```
  处理完当前节点之后，React 继续对子节点进行递归。

- 对比同类型的组件元素
  
- 对子节点进行递归
  默认情况下，当递归 DOM 节点的子元素时，React 会同时遍历两个子元素的列表；当产生差异时，生成一个 mutation
  在子元素列表末尾新增元素时，更新开销比较小。如果只是简单的将新增元素插入到表头，那么更新开销会比较大。会重建每一个子元素。这种情况会带来性能问题，当子元素拥有 key 时，React 使用 key 来匹配原有树上的子元素以及最新树上的子元素。以下示例在新增 key 之后，使得树的转换效率得以提高
# 高阶组件 HOC

复用组件逻辑的一种高级技巧，是基于React组合特性而形成的设计模式
高阶组件是`参数为组件`，`返回值为新组件`的`函数`
HOC 不会修改传入的组件，也不会使用继承来复制其行为。相反，HOC 通过将组件包装在容器组件中来组成新组件。HOC 是纯函数，没有副作用
# render prop
render prop 是一个用于告知组件需要渲染什么内容的函数 prop
# 性能优化
## shouldComponentUpdate
React 只更新改变了的 DOM 节点，重新渲染仍然花费了一些时间。可以通过覆盖生命周期方法 shouldComponentUpdate 来进行提速。该方法会在重新渲染前被触发。其默认实现总是返回 true，让 React 执行更新
```js
shouldComponentUpdate(nextProps, nextState) {
  return true;
}
```
![](./should-component-update.png)
组件只有当 props.color 或者 state.count 的值改变才需要更新时，你可以使用 shouldComponentUpdate 来进行检查：
```js
class CounterButton extends React.Component {
  constructor(props) {
    super(props);
    this.state = {count: 1};
  }

  shouldComponentUpdate(nextProps, nextState) {
    if (this.props.color !== nextProps.color) {
      return true;
    }
    if (this.state.count !== nextState.count) {
      return true;
    }
    return false;
  }

  render() {
    return (
      <button
        color={this.props.color}
        onClick={() => this.setState(state => ({count: state.count + 1}))}>
        Count: {this.state.count}
      </button>
    );
  }
}
```
React 已经提供了一位好帮手来帮你实现这种常见的模式 - 你只要继承 React.PureComponent 就行了。所以这段代码可以改成以下这种更简洁的形式：
```js
class CounterButton extends React.PureComponent {
  constructor(props) {
    super(props);
    this.state = {count: 1};
  }

  render() {
    return (
      <button
        color={this.props.color}
        onClick={() => this.setState(state => ({count: state.count + 1}))}>
        Count: {this.state.count}
      </button>
    );
  }
}
```

## Profiler组件测量渲染带来的开销

```js
render(
  <App>
    <Profiler id="Navigation" onRender={callback}>
      <Navigation {...props} />
    </Profiler>
    <Main {...props} />
  </App>
);

function onRenderCallback(
  id, // 发生提交的 Profiler 树的 “id”
  phase, // "mount" （如果组件树刚加载） 或者 "update" （如果它重渲染了）之一
  actualDuration, // 本次更新 committed 花费的渲染时间
  baseDuration, // 估计不使用 memoization 的情况下渲染整颗子树需要的时间
  startTime, // 本次更新中 React 开始渲染的时间
  commitTime, // 本次更新中 React committed 的时间
  interactions // 属于本次更新的 interactions 的集合
) {
  // 合计或记录渲染时间。。。
}
```
# HOOK

不编写 class 的情况下使用 state 以及其他的 React 特性。

```js
import React, { useState } from 'react';

function Example() {
  // 声明一个新的叫做 “count” 的 state 变量
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```
Hook 是一些可以让你在函数组件里“钩入” React state 及生命周期等特性的函数。

React 内置了一些像 useState 这样的 Hook


- useEffect
  就是在告诉 React 在完成对 DOM 的更改后运行你的“副作用”函数。由于副作用函数是在组件内声明的，所以它们可以访问到组件的 props 和 state。默认情况下，React 会在每次渲染后调用副作用函数 —— 包括第一次渲染的时候。
  ```js
  import React, { useState, useEffect } from 'react';

  function Example() {
    const [count, setCount] = useState(0);

    // 相当于 componentDidMount 和 componentDidUpdate:
    useEffect(() => {
        // 使用浏览器的 API 更新页面标题
        document.title = `You clicked ${count} times`;
    });

    return (
        <div>
        <p>You clicked {count} times</p>
        <button onClick={() => setCount(count + 1)}>
            Click me
        </button>
        </div>
    );
   }
  ```
  通过返回一个函数来指定如何“清除”副作用
  ```js
  import React, { useState, useEffect } from 'react';

    function FriendStatus(props) {
    const [isOnline, setIsOnline] = useState(null);

    function handleStatusChange(status) {
        setIsOnline(status.isOnline);
    }

    useEffect(() => {
        ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
        return () => {
        ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
        };
    });

    if (isOnline === null) {
        return 'Loading...';
    }
    return isOnline ? 'Online' : 'Offline';
    }
  ```









# 工具链

## package管理器
npm / yarn

## 打包器
webpack 

## 编译器
babel