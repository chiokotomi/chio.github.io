---
title: React源码学习
date: 2021-05-22 21:42:05
tags:
---

## react学习分层

- `掌握术语、基本实现思路`：
    - fiber架构、虚拟dom、diff算法(协调reconciliation)
    - 了解视图更新的实现思路
    - 参考[build-your-own-react](https://pomb.us/build-your-own-react/)
- `掌握整体工作流程、局部细节`: 
    - 源码整体工作流程：
        - schedule调度模块 - scheduler
        - render协调模块 - reconciler - 与视图dom无关，仅操作fiber
        - commit渲染模块 - renderer - 仅此阶段与视图平台相关涉及ReactDOM、ReactNative、ReactArt
    - 局部细节：
        - diff算法
        - hooks
- `掌握关键流程的细节`：
    - view视图 - 软件应用

      生命周期函数         -> 符合默认认知的在hooks上一层的抽象
      
      hooks    - 操作系统
      
      React底层运行流程  - 硬件
    - scheduler调度器模块：优先级队列、小顶堆数据结构、浏览器渲染原理、message channel API、宏任务与微任务的区别、event loop参考[深入核心，详解事件循环机制](https://mp.weixin.qq.com/s/m3a6vjp8-c9a2EYj0cDMmg)
    - 协调模块：dfs深度优先遍历、update单向（有环）链表数据结构、lane模型、二进制掩码、fiber并发模型(原生提供的generator)
- `掌握思想`：
    - hooks之前classComponent面向对象、functionComponent函数式编程
    - state到view的变化，纯函数更好表达，编译时优化更有优势
    - 代数效应理念处理副作用 hooks实现

## React理念

构建快速响应的大型web应用，解决两个主要瓶颈
- CPU瓶颈：计算量大、操作多、设备性能低时的卡顿
- IO瓶颈：网络请求数据等待，导致下一步不能快速响应

### CPU瓶颈

- 主流浏览器刷新频率为60HZ，1000ms/60HZ，16.6ms刷新一次。

- JS线程、JS操作DOM、GUI渲染线程 互斥

  JS脚本、样式布局  、样式绘制   不能同时执行

- React解决方案：将`同步更新`变为`可中断的异步更新`
    - `时间切片time slice`： 将长任务拆分到每一帧中。每一帧预留一些时间（5ms）react来更新组件，剩下的时间交还给浏览器渲染UI。
    - `Concurrent Mode`：
        ```js
        ReactDOM.render(<App/>, rootEl);
        ReactDOM.unstable_createRoot(rootEl).render(<App />)
        ```

### IO瓶颈

在`网络延迟`存在的客观情况下，减少用户的感知。
- `Suspense`实现与配套的`hook-useDeferedValue`

## React16新架构

- react15中协调器`reconciler`使用递归来创建虚拟DOM，不能中断，占用多时间
- react16为了改为`异步可中断更新`，采用了`Fiber架构`

## Fiber架构

- 静态数据包含：对应的React element的组件的类型、DOM节点信息等
- 动态工作单元包含：本次更新中组件改变的状态、要执行的工作（被删除/被插入页面中/被更新），调度优先级相关的信息