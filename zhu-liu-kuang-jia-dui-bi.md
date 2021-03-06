---
title: 主流框架对比
date: '2021-04-23T20:13:47.000Z'
tags: null
---

# 主流框架对比

对比vue & react & angular 共同特征与不同设计

## React vs Vue

### 定位

* react：从开发模式上提出突破性的新方向，ui开发的新思路，rethinking best pratices
* vue：降低开发门槛，快速上手开发

  **共同点**

* 使用虚拟Dom（Virtual Dom）：代码操作虚拟dom比操作真实dom效率更高渲染效率更快
* 提供响应式（Reactive）和组件化（Composable）的视图组件：数据驱动UI变化，原子组件组成世界的概念
* 将注意力集中保持在核心库，而将其他功能（路由、全局状态管理）交给相关库。

### 不同点

#### 运行时性能优化

* React

  当某个组件的状态发生变化时，它会以该组件为根，重新渲染整个组件子树。

  避免不必要的子组件重渲染，需要尽可能使用`PureComponent`，或是手动实现`shouldComponentUpdate`

* Vue

  组件的依赖在渲染过程中自动追踪，不需要开发者自己考虑此类优化。

  **HTML & CSS**

* React

  一切都是jsx：可以使用js来构建视图页面，开发工具\(linting、类型检查等\)支持更先进

* vue

  推荐模板：更符合HTML开发习惯

#### 规模

* React

  把路由库与状态库交给社区维护，创建一个更分散的生态系统，相对更繁荣。

* Vue

  路由跟状态库由官方支持维护与核心库同步更新。

  提供功能更全的CLI脚手架

#### 原生渲染

* React

  React Native 使用相同的组件模型编写有本地渲染能力的APP，可以跨多平台

* Vue

  Weex与阿里合作，发起跨多平台用户界面开发框架

## Vue vs Angular1

### 数据绑定

* Angular1 双向绑定
* Vue 单向数据流，数据流向更清晰

  **指令与组件**

* Angular1 所有都是指令，组件是特殊的指令
* vue 指令跟组件区分清晰，指令只封装dom操作，组件代表一个独立单元有自己的数据逻辑与视图

### 运行时性能

* Angular1 使用watcher，越多越慢，使用脏检查循环\(digest cycle\)
* vue 基于依赖追踪观察系统，并且异步队列更新，数据变化独立触发没有明确依赖

## Vue vs Angular2

* Angular2

  必须要用ts开发

  使用tree-shaking后体积依旧比vue规模大很多

