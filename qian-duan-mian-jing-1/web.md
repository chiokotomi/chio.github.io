# web

## Q1 - 前端路由实现原理\(react-router/vue-router\)

路由就是URL与组件树的映射关系。监听URL的变化，匹配到对应的组件。

* 基于hash实现
  * window.location.hash
  * window.addEventListener\('hashchange', handler\)
  * url会携带\#，但不会发http请求
* 基于History API实现
  * 没有\#，但是重新刷新会发请求，需要后端配合

```javascript
history.pushState(); // 添加新的状态到历史状态栈
history.replaceState(); // 用新的状态代替当前状态
history.state; // 返回当前状态对象
```

## Q2 - 客户端/浏览器缓存

* 设置静态资源缓存
  * 请求更快，缓存在本地或者服务器\(CDN\)可以提高请求响应速度
  * 节省带宽，缓存本地不需要网络请求
  * 降低服务器压力
* 浏览器缓存通过HTTP cache headers
  * cache-control响应头
    * public任何情况都缓存 / private不可缓存
    * no-cache缓存前使用Etag\(验证令牌\)验证资源是否有变 / no-store不允许浏览器缓存
    * max-age指定资源可悲利用的最大时限=30s即30s内
  * expries 资源过期时间（向后兼容，cache-control优先）
  * last-modified 上次更改时间（向后兼容，cache-control优先）

## Q3 - 从URL到页面展示发生了什么

* 输入url
* DNS解析域名为ip地址
* 浏览器与服务器建立TCP链接，三次握手
* 浏览器向服务器发送 http 请求
* 服务器处理请求，并返回 HTTP 报文\(304 Not Modified

  缓存\)

* 浏览器解析并渲染页面
* 断开连接：四次挥手

## Q4 - MVVM

* Model
* View
* ViewModel： 沟通Model View的桥梁，负责数据与业务处理
* 数据双向绑定：
  * angular脏数据检测：效率低

## Q5 - vue react angular架构对比

* vue vs Angular
  * ng2使用ts开发，vue2 js，vue3 js与ts都可以
  * ng 依赖脏数据检查watcher增多导致性能问题
  * ng 组件也是指令，vue 仅使用指令操作dom
* vue vs react
  * vue 模板，react JSX

## Q6 - SPA性能优化

* 减少入口文件体积 =》 webpack Q2
* 开启静态缓存 =&gt; Q2客户端浏览器缓存
* 服务上开启Gzip压缩
* 使用SSR
* 第三方模块按需引入
* 懒加载
* 用户体验 
  * 骨架屏 ant-design Skeleton
  * PWA渐进式Web应用
    * Service Worker
* SSR
  * node `vue-server-renderer.createRenderer()`
  * node `react ReactDOMServer.renderToStaticMarkup()`

## Q7 - http1 http2 https

* http1 长连接keep-alive
* http2
  * 多路复用多个请求在同一个tcp上完成
  * 报文头部压缩
* https
  * http: IP - TCP - HTTP
  * https: IP - TCP - SSL\(Secure Sockets Layer\) -HTTP

## Q8 - 解决跨域问题

* 1 - JSONP跨域

  本质是

  ```javascript
  var script = document.createElement('script');
    script.type = 'text/javascript';
    // GET请求携带参数与callbackHandler
    script.src = 'http://www.domain2.com:8080/login?user=admin&callback=handleCallback';
    document.head.appendChild(script);
    // 回调执行函数
    function handleCallback(res) {
        alert(JSON.stringify(res));
    }
  ```

* 2 - CROS跨域资源共享，请求头协议

  \`\`\`js

  // 请求头

  GET /cors HTTP/1.1

  // Origin字段

  Origin: [http://api.bob.com](http://api.bob.com)

  Host: api.alice.com

// 响应头 Acess-Control-Allow-Origin

\`\`\`

* 3 - iframe跨域
* 4 - websocket协议跨域\(socket.io.js\)

## Q9 - 常见web攻击原理与预防

* xss
* csrf
* sql注入

