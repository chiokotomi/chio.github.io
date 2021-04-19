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

# VUE相关设计概念

## 非侵入性的响应式系统
> 响应式系统：修改数据模型(普通的js对象)时，视图会进行更新。这使得状态管理非常简单直接。

### Vue2实现

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

# Next To Learn

- data 属性设定与作用