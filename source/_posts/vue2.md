---
title: vue2学习笔记
date: 2021-04-22 21:24:52
tags: vue2
---

> **Vue是一个用于构建用户界面的渐进式框架**，对于已存在的服务端应用，可将vue作为应用的一部分嵌入；如果将更多的业务逻辑放在前端，也可以用vue的核心库与生态系统来实现。
Vue核心库只关注视图层。
<!-- more -->

# 安装介绍
## 不支持IE8

实现`响应式系统`是使用`Object.defineProperty`来遍历`data`选项把所有的`property`转化为`getter/setter`

`Object.defineProperty`是ES5中一个无法shim的特性，这就是Vue2不支持IE8的原因。

## 版本发布遵循语义化版本控制

遵循[语义化版本控制](https://semver.org/lang/zh-CN/)

## 构建版本
- 编译器： 用来将模板字符串编译成为JavaScript渲染函数的代码
```js
// 需要编译器
new Vue({
    template: '<div>{{ hi }}</div>'
})
```
- 运行时： 用来创建Vue实例、渲染并处理虚拟DOM等的代码。基本上就是除去编译器的其他一切。**运行时版本比完整版体积小大约30%**
```js
// 不需要编译器
new Vue({
    render (h) {
        return h('div', this.hi);
    }
})
```
- 完整版： 编译器+运行时

# 非侵入性的响应式系统

**响应式系统：**修改数据模型(普通的js对象)时，视图会进行更新。这使得状态管理非常简单直接。

## 如何追踪变化

> 针对Vue实例的`data`选项（一般都传入普通的js对象）

Vue2遍历此对象的所有`property`，使用`Object.defineProperty`把所有的`property`转化为`getter/setter`。
```js
// vue2代码
export function proxy (target: Object, sourceKey: string, key: string) {
  sharedPropertyDefinition.get = function proxyGetter () {
    return this[sourceKey][key]
  }
  sharedPropertyDefinition.set = function proxySetter (val) {
    this[sourceKey][key] = val
  }
  Object.defineProperty(target, key, sharedPropertyDefinition)
}
```
这些`getter/setter`对用户不可见，在内部可以让Vue能追踪依赖，在property被访问和修改时通知变更。
每个组件实例对应一个`watcher`实例，会在组件渲染的过程中把接触过的数据`property`记录为依赖，之后当依赖项的`setter`触发时，会通知`watcher`，从而使他关联的组件重新渲染。

![](vue-data-watcher-render.jpg)

需要注意的事：
- 由于js的限制，vue不能检测数组与对象的变化，使用`Vue.set/vm.$set`方法更改`data`中对象或数组的值保证他们的响应性。

- data根级响应式的property不可以动态添加，必须在初始化实例时声明所有的响应式property。主要目的时消除在依赖项跟踪系统中的一类边界情况。同时提高代码的可维护性，`data`对象就像组件状态的`结构(schema)`

- 使用Object.freeze()会阻止修改现有的property，响应系统将无法追踪变化
```js
  var obj = {foo: 'bar'};
  Object.freeze(obj);
  new Vue({
    el: '#app',
    data: obj
  })
  <div id="app">
    <p>{{ foo }}</p>
    <!-- 这里的 'foo' 不会更新 -->
    <button v-on:click="foo = 'baz'">Change it</button>
  </div>
```

## 异步更新队列

Vue 在更新 DOM 时是异步执行的。只要侦听到数据变化，Vue 将开启一个队列，并缓冲在同一事件循环中发生的所有数据变更。如果同一个 watcher 被多次触发，只会被推入到队列中一次。这种在缓冲时去除重复数据对于避免不必要的计算和 DOM 操作是非常重要的。然后，在下一个的事件循环“tick”中，Vue 刷新队列并执行实际 (已去重的) 工作。Vue 在内部对异步队列尝试使用原生的 `Promise.then`、`MutationObserver` 和 `setImmediate`，如果执行环境不支持，则会采用 `setTimeout(fn, 0)` 代替。
`Vue,nextTick(callback)`回调函数将在DOM更新完成后被调用
```js
// 使用方法
Vue.component('example', {
  template: '<span>{{ message }}</span>',
  data: function () {
    return {
      message: '未更新'
    }
  },
  methods: {
    updateMessage: function () {
      this.message = '已更新'
      console.log(this.$el.textContent) // => '未更新'
      this.$nextTick(function () {
        console.log(this.$el.textContent) // => '已更新'
      })
    }
  }
})
// 或使用ES2017 async/await
methods: {
  updateMessage: async function () {
    this.message = '已更新'
    console.log(this.$el.textContent) // => '未更新'
    await this.$nextTick()
    console.log(this.$el.textContent) // => '已更新'
  }
}
```
# 过渡效果系统

Vue插入、更新、移除元素时自动应用过渡效果

# Vue将模板编译成虚拟DOM渲染函数
# 组件化应用构建

组件的本质：一个拥有预定义选项的Vue实例
大型应用中，将整个应用程序划分为组件，使开发更易管理。
## Vue组件与自定义元素（custom elements）的关系
- 自定义元素：[web组件（Web Components）规范](https://developer.mozilla.org/zh-CN/docs/Web/Web_Components)的一部分
- 相同点：
  - Vue组件语法参考了web组件规范（例如slot API 与 `is` attribute）
- 不同点：
  - web components规范已通过，但并未被所有浏览器原生实现。然而vue组件不需要任何polyfill，并支持ie9及以上。必要时，Vue组件也可以包装于原生自定义元素之内。
  - vue组件提供了自定义元素所不具备的一些功能，例如`跨组件数据流`、`自定义事件通信`、`构建工具集成`


# Vue实例生命周期图示

![](lifecycle.png)

# 父子组件单向数据流

所有的`prop`使得父子`prop`之间形成了一个`单向下行绑定`。以防止子组件意外更改父组件的状态，从而导致应用的数据流向难以理解。