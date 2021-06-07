---
title: Webpack工具链
date: '2021-04-12T12:01:18.000Z'
tags: null
---

# webpack

`webpack`是模块打包工具，管理模块依赖，并编译输出模块们所需的静态文件。  可管理所有静态资源文件：html、js、css、图片、字体等。

不同类型资源使用不同的`loader`

webpack自动分析模块间`依赖关系`，最后生成优化合并后的静态资源。

`插件`提供了处理各种文件过程中的生命周期钩子，可以开发者自定义功能。

## 核心打包原理

### 主要流程

1. 读取入口文件内容
2. 分析入口文件，递归查找模块依赖文件内容，生成AST语法树
3. 根据AST语法树，生成浏览器可执行代码

### 过程步骤

1. 读取模块内容
2. 分析模块
3. 收集依赖
4. ES6转成ES5（AST）
5. 递归获取所有依赖
6. 处理两个关键字：import exports
7. 输出打包结果

### 步骤实现

1. 获取主模块内容
2. 分析模块
   * 安装`@babel/parser`转AST
3. 对模块内容进行处理
   * `@babel/travers`包遍历AST收集依赖
   * `@babel/core`和`@babel/preset-env`包ES6转ES5
4. 递归处理所有模块
5. 生成最终代码

### 代码实现

```javascript
/**
 * 手写webpack核心打包流程
 */
// 获取主入口文件
const fs = require('fs');
const path = require('path');
const parser = require('@babel/parser');
const traverse = require('@babel/traverse').default;
const babel = require('@babel/core');

// 解析单个文件，获取文件路径，依赖文件列表，编译成es5的代码
const getModuleInfo = (file) => {
    const body = fs.readFileSync(path.resolve(__dirname, file), 'utf-8');
    // 新增代码
    const ast = parser.parse(body, {
        sourceType: 'module' // 表示我们要解析的是ES模块
    });

    // 新增代码
    const deps = {};
    traverse(ast, {
        ImportDeclaration({node}) {
            const dirname = path.dirname(file);
            const abspath = './' + path.join(dirname, node.source.value);
            deps[node.source.value] = abspath;
        }
    });
    const {code} = babel.transformFromAst(ast, null, {
        presets: ['@babel/preset-env']
    });
    const moduleInfo = {file, deps, code};
    return moduleInfo;
};

// 编译入口文件，非递归遍历AST依赖树，将所有文件解析后生成平铺的映射对象
const parseModules = (file) => {
    const entry = getModuleInfo(file);
    const temp = [entry];
    const depsGraph = {};
    for (let i = 0; i < temp.length; i++) {
        const deps = temp[i].deps;
        if (deps) {
            for (const key in deps) {
                if (deps.hasOwnProperty(key)) {
                    temp.push(getModuleInfo(deps[key]));
                }
            }
        }
    }

    temp.forEach(moduleInfo => {
        depsGraph[moduleInfo.file] = {
            deps: moduleInfo.deps,
            code: moduleInfo.code
        };
    });
    return depsGraph;
};

// 打包，生成递归遍历执行依赖关系树的执行器和AST依赖关系树
const bundle = file => {
    const depsGraph = JSON.stringify(parseModules(file), null, 4);
     return `;(function (graph) {
        function require(file) {
            function absRequire(relPath) {
                return require(graph[file].deps[relPath])
            }
            var exports = {};
            (function (require, exports, code) {
                eval(code);
            })(absRequire, exports, graph[file].code);
            return exports;
        }
        require('${file}');
    })(${depsGraph});`;
}；

const entry = './src/index.js';
const content = bundle(entry);

if (!fs.existsSync('./dist')) {
    fs.mkdirSync('./dist');
}
fs.writeFileSync('./dist/bundle.js', content);
```

