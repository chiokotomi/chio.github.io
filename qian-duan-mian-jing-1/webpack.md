# webpack

## Q1 - 基本原理

* 从入口分析模块依赖
  * 使用@babel/parser作为js解析器读取内容，转为AST语法树
  * 使用@babel/traverse解析AST语法树提取依赖关系
  * 使用@babel/preset-env预设编译环境，使用@babel/core.transformFromAst转译代码，适配所有浏览器
  * 对依赖模块进行以上分析，形成依赖关系网
* 打包流程

  * 初始化webpack.config.js
  * loader解析
    * rules规则优化，指定include与exclude
    * 缓存cache-loader、thread-loader
  * 搜索依赖文件
    * 配置resolve缩短搜索时间
      * resolve.module
      * resolve.alias
      * resolve.extension
  * 打包生成chunk

    \*\*\*\*

  \*\*\*\*

## **Q2 - 优化**

* 优化分析
  * speed-measure-webpack-plugin 速度分析
  * webpack-bundle-analyzer 分析包大小
* 代码压缩
  * UglifyJSPlugin多进程压缩js文
  * mode不同模式dev/pro下webpack自动优化
* tree shaking
  * 模块级别，基于模块间的引用关系，将不被用到的模块删除
* 分包策略：提取公共代码库
  * 4.x版本前 `commonsChunkPlugin：` 多页面应用程序使用async模式

    ```javascript
    // node_modules里的包
    new webpack.optimize.CommonsChunkPlugin({
      name: 'vendor',
      minChunks: function (module, count) {
      return (
          module.resource &&
          /\.js$/.test(module.resource) &&
          module.resource.indexOf(path.join(__dirname, './node_modules')) === 0
      )
      },
    }),
    // 被3个入口chunk共享的模块
    new webpack.optimize.CommonsChunkPlugin({
      name: 'common',
      chunks: 'initial',
      minChunks: 2,
    }),
    ```

  * 4后`optimization.splitChunks`：production模式下自动开启代码分割
  * DLLPlugin 动态链接库,不常更改的代码单独编译
* cdn加速
  * 分离基础包：使用html-webpack-externals-plugin，分离出来的包通过CDN引入
* 多线程打包happypack
* 按需加载/懒加载：`require.ensure`
* 图片 webp格式

## Q3 - tree-shaking

* 原理
  * 依赖ES2015的`import/export`，在编译的静态分析阶段找到未被引入的模块打上标记，压缩阶段利用uglify-js等压缩工具删除对应代码。
* 具体配置： 
  * 代码必须具名导入

    ```javascript
    import { debounce } from 'lodash';
    ```

  * webpack配置

    ```javascript
    const config = {
    // 必须产品模式
    mode: 'production',
    optimization: {
      // 开启usedExports才会做标记
      usedExports: true,
      minimizer: [
          // 压缩器
          new TerserPlugin({...})
      ]
    }
    };
    ```

