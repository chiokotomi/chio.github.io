# react & redux

## react

### Q1 - react组件的挂载过程

* 挂载过程：是指创建组件实例并插入到DOM中。

  生命周期调用顺序如下

  * constructor\(\)
  * static getDerivedStateFromProps\(\)
  * render\(\)
  * componentDidMount\(\)

* 额外的更新过程：props或state发生变化时
  * static getDerivedStateFromProps\(\)
  * shouldComponentUpdate\(\)
  * render\(\)
  * getSnapshotBeforeUpdate\(\)
  * componentDidUpdate\(\)
* 组件卸载：
  * componentWillUnmount\(\)
* 错误处理：渲染过程、生命周期、子组件构造函数出现问题时
  * static getDerivedStateFromError\(\)
  * componentDidCatch\(\)

### Q2 - 组件间通信

* 1. 父=&gt;子
     * 通过props： 父向子props中传递数据
     * 通过实例方法：通过使用refs来调用子组件实例的方法
* 1. 子=&gt;父
     * 回调函数：子调用从父组件传递过来的回调函数
* 1. 兄弟组件间 
     * 同一父组件做桥梁
* 1. 不相关组件间
     * Context API
     * redux状态管理库

### Q3 - 高阶组件

* 一个函数，以组件为参数，返回新的组件。
* 抽取公共逻辑，避免重复代码。
* `react-redux`中的connect就是高阶组件，把redux中的状态方法添加到组件的props上去。

### Q4 - setState\(\)方法

* 除了挂载时初始化，setState\(\)是唯一改变状态的方式
* 直接给state赋值不会触发重渲染，setState\(\)才会
* setState\(\)既可以是同步的，也可以是异步的
  * JSX中写的事件都是合成事件，合成事件中是异步
  * 生命周期函数中是异步
  * 原生事件中是同步
  * setTimeout,setInterval等异步执行的代码中是同步
* setState\(\)第二个参数，是一个回调函数，会在页面重新渲染之后调用，这样就能获取到真正的状态

### Q5 - hooks初探

* hooks只在函数组件中能用，让函数组件完成类组件的功能
* 解决状态逻辑复用难的问题
* 常用hooks
  * `useState()`：状态维护
  * `useEffect(func, null|[]|[data])`: 副作用钩子
    * 可用来放fetch请求
    * 让代码在合适时间执行，模拟生命周期函数
    * 第一参数若return了一个函数，则在组件willUnmount阶段会执行return中的函数
    * 第二参数为null时，第一参数在每次渲染（挂载与更新）都会执行
    * 第二个参数有值时，会在update阶段对比state中参数是否有变化，有变化时才会执行func
  * `useContext()`：组件间共享状态钩子
  * `useReducer()`: action钩子

### Q6 - key

列表结构的每一项需要加上key属性

* diff算法：当state/props更新时对比新旧虚拟DOM树的差别
* 列表头部新加项目时会使整个列表unmount后再mount，引入key解决这个问题

### Q7 - react中的错误捕捉

* 错误边界，避免因ui的js错误引起应用崩溃。
* 设置错误边界：`static getDeriverdStateFromError()`/`componentDidCatch()`封装高阶组件
* 错误边界不能捕获的异常
  * 事件handler处理内部的错误：使用try/catch
  * 异步的代码如setTimeout或requestAnimationFrame：使用window.addEventListener\('error', func\)
  * 服务端渲染

### Q8 - 受控组件与非受控组件

组件状态是否受我们控制

```javascript
<input /> // 非受控
<input value={val} onChange={handleChange} /> // 受控
```

* 获取非受控组件的状态使用ref

  ```javascript
  this.input = React.createRef();
  <input type="text" ref={this.input} />
  ```

* 使用useRef Hook

  ```javascript
  const inputEl = useRef(null);
  <input ref={inputEl} />
  ```

### Q9 - Fiber架构

* 解决复杂复合组件最上层`state`更改后，调用栈过长造成阻塞主线程的问题。
* Fiber本质是一个虚拟堆栈帧，调度器按照优先级调度这些帧。将之前的同步渲染改为异步渲染。不影响体验的情况下，分段计算更新。

### Q10 - 组件渲染性能优化与Immutable.js与PureComponent

* Immutable.js提供了简洁高效的判断数据是否变化的方法，与`shouldComponentUpdate`配合使用，极大的性能提升

  ```javascript
  shouldComponentUpdate: (nextProps = {}, nextState = {}) => {
    for (const key in nextState) {
        if (thisState[key] !== nextState[key] || ！is(thisState[key], nextState[key])) {
            return true;
        }
    }
  }
  ```

* PureComponent在shouldComponentUpdate中做了简单的数据判断

  **Q11 - diff算法**

* 树对比           tree diff： 分层比较
* 同类组件对比      component diff
* 同层级的子节点    element diff： key区分比较

### Q12 - 类组件与函数组件

* 类组件提供生命周期钩子
* 函数组件配合Hooks使用做到生命周期钩子的效果
* 类组件=&gt;函数组件，拥抱函数式编程思想

### Q13 - 虚拟DOM

虚拟DOM是react与vue的设计核心。用js数据描述真实dom内容包含元素标签、属性、子项。通过操作虚拟dom后再批量更新到真实dom，比直接操作真实dom性能高很多。

## redux

### Q1 - 如何理解redux

* react的state状态管理方案
* 包含三个部分
  * state store：单一数据源，只读getState\(\)
  * action对象: 即将执行什么数据变化 {type: 'ADD', payload: 2}，发起action通过`dispatch`
  * reducer: 根据action.type真正执行操作，并返回新的state
* redux基本原则
  * 单一数据源
  * state只读不可直接更改
  * 使用纯函数修改

    > 纯函数：返回结果只依赖参数，执行过程没有副作用

### Q2 - redux middleware中间件

嵌入在action发起后，到达reducer之前的代码

常用：进行日志记录、错误处理，redux-logger,redux-thunk

