# node

## Q1 - koa与express

* 与express区别：回调写法不同，koa async await
* 中间件： 

  koa使用async await，可以等待异步操作；

  Express 中间件实现是基于 Callback 回调函数同步的，它不会去等待异步（Promise）完成

* 响应机制：

  Koa 中数据的响应是通过 ctx.body 进行设置，在响应之前是有一些预留操作空间的

  Express 中我们直接操作的是 res 对象，直接 res.send\(\) 之后就立即响应了

  \*\*\*\*

## **Q2 - TS**

是js的超集，可以编译成js

* 支持强类型或静态类型特性
* 支持面向对象的编程思想：类、接口、继承、泛型
* 编译时强调错误

  \*\*\*\*

## **Q3 - Node服务相关概念**

* node stream
* 异步队列
* pm2 启停进程等
* 服务管理
* 压测？
* 进程间通信
* 微服务
* npm 包管理器
* fs读写文件文件
* 单线程、事件驱动、非阻塞I/O

异步函数会被安排在事件循环的下一次迭代中运行，从而解除了其余代码的阻塞

## Q4 - Node基础知识

* 错误优先回调函数

是指回调函数的第一个参数永远是一个错误对象，用于检查程序是否发生了错误

```javascript
fs.readFile(filePath,  (err, data) => {})
```

* 避免回调地狱

模块化、Promise、yield

* 事件循环

Node是单线程处理机制，node中事件循环的实现是依靠的libuv引擎，事件循环使nodeJs具有异步特性。

一个循环五个阶段

```text
- 第一阶段运行预定的setTimeout/setInterval，
- 第二个阶段运行计划在当前迭代上运行的IO回调，
- 第三个阶段轮询将在下一次迭代中执行的事件；
- 第四个阶段运行setImmediate回调；
- 第五阶段所有close回调
```

* 单线程

`child_process`并行多个进程： 可以生成和派生子进程，用自己的内存空间运行自己的进程 `worker`运行多个线程：worker thread，一个进程中的线程，可以与主线程共享内存

## Q5 - Nginx

* 反向代理

以代理服务器接受来自internet上的请求，后转发给内部网络上的服务器，将服务器返回结果发送给客户端。

## Q6 - python/shell

## Q7 - REST API

URL定位资源， 用GET,POST,DELETE,DETC表示操作

## Q8 - node常用模块

* fs
* path
* http
* os

