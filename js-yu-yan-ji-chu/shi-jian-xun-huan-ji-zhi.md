---
title: 事件循环机制Event Loop
date: '2021-05-23T18:32:05.000Z'
tags: null
---

# 事件循环机制

事件循环机制从整体上告诉了我们js代码的执行顺序。

> 已备知识：
>
> * 执行上下文
> * 函数调用栈
> * 队列数据结构
> * Promise

js的主要特点：单线程，拥有唯一的一个事件循环。 js执行过程中，靠`函数调用栈`来搞定函数的执行顺序，靠`任务队列(task queue)`来搞定另外一些代码的执行。

* 一个线程中，事件循环是唯一的，任务队列是多个的
* 任务队列分为：`宏任务`与`微任务`
* `宏任务`包括：script整体代码、setTimeout、setInterval、setImmediate、I/O、UI rendering
* `微任务`包括：process.nextTick、promise、MutationObserver
* setTimeout/promise等称之为任务源，由他们指定的具体执行任务会进入任务队列。
* 不同任务源的任务会进入到不同的任务队列，setTimeout与setInterval是同源的。
* `Promise`构造函数中的第一个参数（函数）是在new时立即执行的，.then会被分发到微任务队列里
* 宏任务与微任务都是借助函数调用栈来完成
* 事件循环的顺序：
  * 从script整体代码开始第一次循环，全局上下文进入函数调用栈，直到调用栈清空
  * 开始执行所有的`微任务`，执行完毕后
  * 循环从`宏任务`开始，找到其中一个任务队列执行完毕
  * 再执行所有的`微任务`，进入循环：`所有微任务`-`任一宏任务队列`
