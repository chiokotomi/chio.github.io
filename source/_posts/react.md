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

