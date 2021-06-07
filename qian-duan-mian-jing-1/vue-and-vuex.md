# vue & vuex

## vue

### Q1 - Vue 有了数据劫持为什么还要 DOM diff？

数据劫持需要对数据绑定一个 Watcher，如果粒度太细会产生较大开销，因此 Vue 选择在组件级别 Watcher, 而组件内部采用 DOM Diff

### Q2 - this.$nextTick\(\)

* 在下次 DOM 更新循环结束之后执行延迟回调，用于获得更新后的 DOM。

### Q3 - 生命周期

* `beforeCreate()`：实例创建之初，属性生效之前
* `created()`：组件实例已创建、属性绑定，未生成真实dom
* `beforeMount()`：挂载开始之前被调用，render首次调用
* `mounted`：el被新创建的vm.$el替换，挂载到实例之后调用mounted
* `beforeUpdate()`：更新前被调用，在虚拟DOM打补丁之前
* `updated()`：组件数据更新之后
* `beforeDestroy()`：组件销毁前调用
* `destroyed()`：组件销毁后调用

### Q4 - v-model双向绑定与数据劫持

* 数据劫持原理
  * vue2通过Object.defineProperty\(\)设置set/get，只能监听对象第一层，不能监听数组
  * vue3使用proxy

    ```javascript
    const handler = {
    get(target, key) {
      if (typeof target[key] === 'object') {
          return new Proxy(target[key], handler)
      }
      return Reflect.get(target, key);
    },
    set(target, key, value) {
      // 对比oldValue newValue，通知view更新
      return Reflect.set(target, key, value)
    }
    };
    new Proxy(data, handler)
    ```
* 双向绑定原理：发布-订阅模式
  * 初始化数据劫持getter/setter
  * 每个组件watcher实例记录渲染时需要的数据，数据setter时，通知watcher，后重新渲染

### Q5 - 虚拟DOM

* 操作`真实DOM`代价较大：10次dom操作，每一次都从头执行一遍构建DOM树-&gt;解析CSS样式表-&gt;组合DOM树与样式表-&gt;确定坐标-&gt;绘制
* `虚拟DOM`：对真实DOM的抽象描述。将10次更新的diff内容保存在内存对应的js对象中，虚拟对象更新完成后再映射为真实DOM
* 虚拟DOM到真实DOM经过：VNode的create、diff、patch

  \`\`\`js

  // 真实DOM

  * Item 1
  * Item 2
  * Item 3

// 虚拟DOM new Element\(tagName, props, children\) el\('ul', {id: 'list'}, \[ el\('li', { class: 'item' }, \['Item 1'\]\), \]\)

// 渲染为真实DOM function render\(\) { const el = document.createElement\(tagName\); el.append\(childEl\) document.body.appendChild\(el\) }

```text
## Q6 - 虚拟DOM diff 

对比两个虚拟dom树，对同一层级的元素进行对比，第二层级跟第二层级对比，算法复杂度O(n)
- 深度优先遍历，记录有差异的节点
- 查看差异类型：节点替换、顺序变更、属性改变、文本改变
- 将差异应用到真正的DOM树
    - 深度优先遍历真实DOM树，找到对应的有差异的节点进行修改
- 三个阶段: create、diff、patch
## Q7 - Vue3的新特性
- 项目包管理
    - Vue3使用monorepo：模块拆分细化，职责明确，依赖关系明确
- 开发语言：
    - Vue2：Flow：facebook的js静态类型检查器
    - Vue3：TypeScript
- 性能优化：
    - 构建体积：移除冷门feature、引入tree-shaking
    - 数据劫持优化：Object.defineProperty => new Proxy
- API优化
    - Composition API：简单组件Options API（），复杂组件Composition API按逻辑关注点整理代码到一个函数中。

## Q8 - Computed和watch

- Computed: 计算属性
    - 被访问时触发计算，有变化时触发重渲染
    - 适用于模板中
- watch：监听属性
    - 主动订阅属性变化，有变化时执行回调函数
    - 适用于处理一段复杂的业务逻辑

## Q9 - 插槽slot

模板复用`<slot></slot>`的部分会在组件使用时对应部分替换

## Q10 - 路由$route与$router
- $route路由信息对象：包含path/params/hash/query...
- $router路由实例对象：包含路由跳转方法、钩子函数等
- pram 与 query的区别

## Q11 - keep-alive

<keep-alive></keep-alive>组件包裹的组件会被缓存
## Q12 - data属性为什么是函数

- 组件复用时，都会返回一个新data，而不会共享同一个data

## Q13 - 组件间通信方式

- 父->子：props
- 子->父：$emit/ref
- 兄弟：eventBus
- 复杂情况： vuex

## Q14 - vue-router

- 实现：
    - hash: hash change
    - history()： popstate： 刷新子路由会404需要后端配合
- API： router.go .push .replace .back
- 懒加载：
    - vue 异步组件
    ```js
    new Rouer({
        routes: [{
            path: 'home',
            component: resolve => require(['@components/home'], resolve)
        }]
    })
```

* ES6 import\(\)

  ```javascript
  component: () => import('@/components/home'),
  ```

* webpack require.ensure\(\)

  ```javascript
  component: (resolve) => require.ensure([], () => resolve(require('@/components/home')), 'home')
  ```

### Q6 - vue-router的query与param的区别

* query要用path来引入，params要用name来引入
* 接收参数都是类似的，分别是`this.route.query.id`和`this. route.query.id`和`this.route.query.id`和`this.route.params.id`
* query在浏览器地址栏中显示参数，params则不显示

## Vuex

### Q1 - 作用

* vue的状态管理插件
* 基本元素

  ```javascript
  // store.js
  Vue.use(Vuex)
  const store = new Vuex.Store({
    state: {}, // 改变state只能通过commit mutation
    getters: {}, // 基于state的派生属性，类似computed
    mutations: {
        // this.$store.commit('SET_NUMBER', 10)提交
        SET_NUMBER(state, data){ 
            // 必须是同步函数
            state.number=data;
        }
    },
    actions: {
        // this.$store.dispatch('ACTION_NAME',data)提交
        SET_NUMBER({state, commit, rootState, rootGetters}, data){
            // 可以是异步方法
            return new Promise();
        }
    },
    // 多模块的store通过modules引入
    modules: {
        moduleA: {stateA, gettersA, mutationA, actionsA}
    }
  });
  ```

* 批量使用vuex里的数据

  ```javascript
  computed: {
    ...mapState(['a'. 'b']) // 批量使用state里的
    ...mapGetters(['total', 'discountTotal']) // 批量使用getter里的
  }
  ```

### Q2 - Vuex 插件

```javascript
// 定义插件
export default function createPlugin(param) {
    return store => {
        // 监听所有mutation
        store.subscribe((mutation, state) => {
            console.log(mutation.type)//是那个mutation
            console.log(mutation.payload)
            console.log(state)
        })

        store.subscribeAction({
            before: (action, state) => {//提交action之前
                console.log(`before action ${action.type}`)
            },
            after: (action, state) => {//提交action之后
                console.log(`after action ${action.type}`)
            }
        })
    }
}

// 使用插件
const myPlugin = createPlugin()
const store = new Vuex.Store({
  // ...
  plugins: [myPlugin]
})
```

