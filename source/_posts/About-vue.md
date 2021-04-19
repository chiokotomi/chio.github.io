---
title: vue学习笔记
date: 2021-04-10 23:13:43
tags: vue
---

# 问题
- vue3 比 vue2 加了什么功能
- vue 设计思想，实现了什么功能，怎么实现的
- vue 支持到的ie版本
- vue 生态其他库的作用是什么
- vue3跟typescript关系是什么，好用在哪里
- 预渲染与服务端渲染如何实现的

# Vue3

## 放弃支持IE11
> 参考vue RFC [proposal for dropping ie11 support in Vue3](https://github.com/vuejs/rfcs/blob/ie11/active-rfcs/0000-vue3-ie11-support.md)
### 原因
1. 浏览器和js的环境发生变化，越来越多的开发者开始使用现代语言特性
2. 微软通过投资Edge推动用户停止对IE的使用，同时在自己的`Microsoft 365`重大项目中放弃使用IE11
3. WordPress放弃支持IE11
4. IE11全球使用率跌破1%以下

### 成本
#### 1. 行为不一致

- 原因：`响应式系统的实现`在`vue2`中基于`ES5的getter/setter`，在`vue3`中基于`ES2015 Proxies`，更高性能更完整的响应式系统。ie11中无法`polyfill`这些特性。

- 好处： `vue3基于Proxy`的响应式系统提供了`近乎完整的`语言特性覆盖：可以检测到许多在ES5中不可行或不可能的操作，如`属性的添加/删除`、`数组索引和长度突变`以及`in操作符的检查等功能`。

- 支持方案：使vue3在ie11版本的开发构建中同时发布`Proxy`和`ES5`两种响应性版本。
  
- 结论： 理论可行，复杂度大，需要将两种实现混合在一起，可能导致***开发和生产之间的行为差异***

#### 2. 长期维护负担

支持ie11就必须考虑整个代码库中使用的语言特性与为发布版本找到合适的polyfill编译策略。

每一个不能在ie11中被polyfill的新特性都会带来新的行为警告

#### 3. 库作者的复杂性

两个响应性实现也会不可避免的影响vue生态其他库作者开发，若决定支持IE11，在编写库时，需要时刻考虑ES5响应系统的一些缺陷。


## 新功能
- composition API
- `<script setup>` 与其他新的`单文件组件(SFC)`特性
- emits 选项
- TS 类型改进
- Vite官方整合

# Vue2

### 不支持IE8

实现`响应式系统`是使用`Object.defineProperty`来遍历`data`选项把所有的`property`转化为`getter/setter`

`Object.defineProperty`是ES5中一个无法shim的特性，这就是Vue2不支持IE8的原因。

# VUE设计 - 非侵入性的响应式系统
> 响应式系统：修改数据模型(普通的js对象)时，视图会进行更新。这使得状态管理非常简单直接。

## Vue2实现
### 如何追踪变化

> 针对Vue实例的`data`选项（一般都传入普通的js对象）

Vue2遍历此对象的所有`property`，使用`Object.defineProperty`把所有的`property`转化为`getter/setter`。
```js
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
> 由于js的限制，vue不能检测数组与对象的变化，使用`Vue.set/vm.$set`方法更改`data`中对象或数组的值保证他们的响应性。

> data根级响应式的property不可以动态添加，必须在初始化实例时声明所有的响应式property。主要目的时消除在依赖项跟踪系统中的一类边界情况。同时提高代码的可维护性，`data`对象就像组件状态的`结构(schema)`
### 异步更新队列

Vue 在更新 DOM 时是异步执行的。只要侦听到数据变化，Vue 将开启一个队列，并缓冲在同一事件循环中发生的所有数据变更。如果同一个 watcher 被多次触发，只会被推入到队列中一次。这种在缓冲时去除重复数据对于避免不必要的计算和 DOM 操作是非常重要的。然后，在下一个的事件循环“tick”中，Vue 刷新队列并执行实际 (已去重的) 工作。Vue 在内部对异步队列尝试使用原生的 `Promise.then`、`MutationObserver` 和 `setImmediate`，如果执行环境不支持，则会采用 `setTimeout(fn, 0)` 代替。
`Vue,nextTick(callback)`回调函数将在DOM更新完成后被调用
```js
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
# Next To Learn

- data 属性设定与作用