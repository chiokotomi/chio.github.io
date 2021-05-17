---
title: vue2的规模化
date: 2021-04-22 21:24:52
tags: vue2
---

**Vue是一个用于构建用户界面的渐进式框架**，对于已存在的服务端应用，可将vue作为应用的一部分嵌入；如果将更多的业务逻辑放在前端，也可以用vue的核心库与生态系统来实现。
<!-- more -->
Vue核心库只关注视图层。


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


## Vue实例生命周期图示

![](lifecycle.png)

## 父子组件单向数据流

所有的`prop`使得父子`prop`之间形成了一个`单向下行绑定`。以防止子组件意外更改父组件的状态，从而导致应用的数据流向难以理解。双向绑定会带来维护上的问题，因为子组件可以变更父组件，且在父组件和子组件都没有明显的变更来源

## 过渡效果系统

Vue插入、更新、移除元素时自动应用过渡效果

# 可复用性&组合
- mixin
  ```js
    let myMixin = {
      created: function () {},
      methods: {
        hello: function() {}
      }
    };
    let Component = Vue.extend({
      mixins: [myMixin]
    });
    let component = new Component();
  ```

- 自定义指令`Vue.directive()`
- 渲染函数`render`
  ```js
  Vue.component('xxx-xxx', {
    render: function(createElement) {
      return createElement(
        'h' + this.level,
        this.$slots.default
      ),
      props: {
        level: {
          type: Number,
          required: true
        }
      }
    }
  });
  ```
  通过`babel插件@vue/babel-preset-jsx`可以使用jsx语法，接近于模板的语法上，Vue的模板实际上也是被编译成了渲染函数`Vue.compile`
  ```js
  new Vue({
    el: '#demo',
    render: function (h) {
      return (
        <AnchoredHeading level={1}>
          <span>Hello</span> world!
        </AnchoredHeading>
      )
    }
  })
  ```
- 虚拟DOM：Vue通过建立一个虚拟DOM来追踪自己要如何改变真实DOM
  `createElement`返回一个虚拟节点`virtual node/VNode`包含信息告诉Vue页面上需要渲染什么样的节点，包括及其子节点的信息描述。
  
 
# Vue插件

插件通常用来为 Vue 添加全局功能，如`vue-loader`
## 使用插件

`Vue.use()`在调用`new Vue()`启动应用之前
```js
  Vue.use(MyPlugin)
  new Vue({
    // ...
  })
```

# 规模化
## 路由
```js
const NotFound = { template: '<p>Page not found</p>' }
const Home = { template: '<p>home page</p>' }
const About = { template: '<p>about page</p>' }

const routes = {
  '/': Home,
  '/about': About
}

new Vue({
  el: '#app',
  data: {
    currentRoute: window.location.pathname
  },
  computed: {
    ViewComponent () {
      return routes[this.currentRoute] || NotFound
    }
  },
  render (h) { return h(this.ViewComponent) }
})
```

## 状态管理
类Flux架构，store模式，集中式状态管理，组件不允许直接变更属于 store 实例的 state，而应执行 action 来分发 (dispatch) 事件通知 store 去改变，这样能够记录所有 store 中发生的 state 变更，同时实现能做到记录变更、保存状态快照、历史回滚/时光旅行的先进的调试工具。
![](./state.png)
## 服务端渲染SSR
将一个组件渲染为服务端的HTML字符串，直接发送到浏览器，再将静态标记“激活”为客户端上可以交互的应用程序
使用SSR可以
- 更好的SEO：搜索引擎爬虫工具可以直接查看完全渲染的页面
- 更快的内容到达时间（time-to-content）
### 基本用法
使用`vue-server-renderer`渲染一个Vue实例
```js
const renderer = require('vue-server-renderer').createRenderer({
  template,
});

const context = {
    title: 'vue ssr',
    metas: `
        <meta name="keyword" content="vue,ssr">
        <meta name="description" content="vue srr demo">
    `,
};

server.get('*', (req, res) => {
  const app = new Vue({
    data: {
      url: req.url
    },
    template: `<div>访问的 URL 是： {{ url }}</div>`,
  });

  renderer
  .renderToString(app, context, (err, html) => {
    console.log(html);
    if (err) {
      res.status(500).end('Internal Server Error')
      return;
    }
    res.end(html);
  });
})

server.listen(8080);
```
### 通用代码
> 运行在服务器与客户端的代码
- 服务器上默认情况下禁用响应式数据
- 生命周期只有`beforeCreate`与`created`会在服务端渲染过程中被调用，避免在期间产生有全局副作用的代码
- 通用代码不接受特定平台的API，如`window`或`document`

### 源码结构
- 使用webpack对client与node端代码进行构建
```js
// 通用app.js
import Vue from 'vue'
import App from './App.vue'

// 导出一个工厂函数，用于创建新的
// 应用程序、router 和 store 实例
export function createApp () {
  const app = new Vue({
    // 根实例简单的渲染应用程序组件。
    render: h => h(App)
  })
  return { app }
}


// entry-client：将app挂载到dom上进行渲染执行
import { createApp } from './app'

const { app } = createApp()

app.$mount('#app')


// entry-server：ssr中直接调用app，后续再执行
// 服务器端路由匹配 server-side route matching
// 和 数据预取逻辑 data pre-fetching logic
import { createApp } from './app'

export default context => {
  const { app } = createApp()
  return app
}
```

### 路由与代码分割
- 路由
- 惰性加载（懒加载）：减少初始渲染中下载的资源体积，改善大体积bundle的可交互时间（time-to-interactive）
  ```js
    import Foo from './Foo.vue'
    // 改为
    const Foo = () => import('./Foo.vue')
  ```

### 数据预取和状态
![数据预取和状态基于vuex](https://ssr.vuejs.org/zh/guide/data.html)

### 客户端激活

 Vue 在浏览器端接管由服务端发送的静态 HTML，使其变为由 Vue 管理的动态 DOM 的过程。
 ```js
  <div id="app" data-server-rendered="true"></div>
  // 强制使用应用程序的激活模式
  app.$mount('#app', true)
 ```
## 预渲染Prerendering
改善少数页面的SEO使用预渲染
在构建时简单地生成针对特定路由的静态HTML文件