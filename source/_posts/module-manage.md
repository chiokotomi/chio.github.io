---
title: 前端模块化
date: 2021-04-20 11:29:51
tags: module
---


模块化的开发方式可以提高代码**复用率**，方便代码管理。一般一**个文件就是一个模块，有自己的作用域，只向外暴露特定的变量和函数**。模块化开发的核心问题就是引入js文件的**顺序问题**以及**异步加载问题**。


<!-- more -->
模块化的开发方式可以提高代码**复用率**，方便代码管理。一般一**个文件就是一个模块，有自己的作用域，只向外暴露特定的变量和函数**。模块化开发的核心问题就是引入js文件的**顺序问题**以及**异步加载问题**。

ES6之前，原生不支持模块化，所以出现了第三方的解决方案，分别是`AMD`与`CMD`，NodeJs诞生后使用`CommonJs`的模块化规范。

# AMD 

异步模块定义`Asynchronous Module Definition`，是`RequireJs`在推广过程中对模块定义的规范化的产出。采用**异步方式加载模块**，模块的加载不影响后面的语句运行。所有依赖这个模块的语句，都定义在回调函数中。

用require.js实现AMD规范的模块化：用`require.config()`指定引用路径，用`define()`定义模块，用`require()`加载模块。

```js
// 网页中引入require.js与main.js(配置require.config()规定用到的基础模块)
<script src="js/require.js" data-main="js/main"></script>

// main.js 入口文件
require.config({
    baseUrl: 'js/lib',
    paths: {
        "jquery": "jquery.min",
        "underscore": "underscore.min"
    }
});
```

引入模块时，将模块名放在`[]`中作为`require()`的第一参数。
```js
// 定义math.js模块
define(function () {
    var basicNum = 0;
    var add = function (x, y) {
        return x + y;
    };
    return {
        add: add,
        basicNum: basicNum
    };
});

// 引入模块
require(['jquery', 'math'], function($, math) {
    return sum = math.add(2, 3);
    $("$sum").htm(sum);
});
```

若定义的模块依赖其他模块，则需要放在`[]`中作为`define`的第一个参数
```js
define(['underscore'], function (_) {
    var classify = function (list) {
        _.countBy(list, function(num) {
            return num > 30 ? 'old' : 'young';
        });
    };
    return {classify: classify};
});
```

# CMD

通用模块加载规范`Common Module Definition`，是`Sea.js`实现的规范。

AMD实现者require.js在申明依赖的模块时，会第一时间加载并执行模块内代码。
```js
// AMD写法
// 声明并初始化所有要用到的模块
define(['a', 'b', 'c'], function (a, b, c) {
    if (false) {
        // 即使未用到模块b，b依旧会提前执行 **这就是CMD要优化的地方**
        b.foo();
    }
});
```
CMD与AMD类似，不同点在于：**AMD推崇依赖前置，提前执行，CMD推崇依赖就近，延迟执行**
```js
// CMD写法
define(function (require, export, module) {
    var a = require('./a');
    a.doSomething();
    if (false) {
        var b = require('./b');
        b.doSomething();
    }
});

// sea.js
// 定义模块math.js
define(function (require, export, module) {
    var $ = require('jquery.js');
    var add = function (x, y) {
        return x + y;
    };
    export.add = add;
});
// 加载模块
seajs.use(['math.js'], function (math) {
    var sum = math.add(2 + 3);
});
```
# CommonJS

NodeJS是主要实践者，需要四个重要的环境变量为模块化的实现提供支持`module`、`exports`、`require`、`global`。
`module.exports`定义对外输出的接口，`require`加载模块。
> 问题：
> CommonJS用**同步**的方式加载模块。在服务器端，模块文件存放本地磁盘，读取非常快。**但在浏览器端，限于网络原因，更合理的方案是使用异步加载**。

```js
// 自定义模块math.js
var basicNum = 0;
function add(a, b) {
    return a + b;
}
module.exports = {
    add: add,
    basicNum: basicNum
}


// 引入自定义模块
var math = require('./math');
math.add(2, 3);
```

# UMD

UMD是AMD和CommonJs的统一规范，支持两种规范，写一套代码，可用于多种场景。
- 写法最复杂
- 前后端通用
- UMD更像是一种配置多个模块系统的模式
- UMD在使用诸如Rollup/Webpack之类的bundler时通常用作备用模块

```js
(function (root, factory) {
    if (typeof define === 'function' && define.amd) {
        // AMD
        define(['jquery]', factory);
    }
    else if (typeof exports === 'object') {
        // Node CommonJs
        module.exports = factory(require('jquery'));
    }
    else {
        // 浏览器全局变量
        root.returnExports = factory(root.jquery);
    }
})(this, function ($) {
    // 定义方法
    function myFunc () {

    }
    // 暴露公共方法
    return myFunc;
});
```


# ESM / ES6 Module

> ES6在语言标准层面上，实现了模块功能。成为浏览器和服务器通用的模块解决方案。
> ES6模块不是对象，`import`命令会被js引擎静态分析，在编译时就引入模块代码，而且不在运行时加载，所以无法实现条件加载。

主要包含两个命令：`export`规定对外接口，`import`输入其他模块。

```js
// 定义math.js
var basicNum = 0;
var add = (a, b) => { return a + b; };
export default {basicNum, add};

// 引入
import {basicNum, add} from './math';
function test (ele) {
    ele.textContent =  add(99 + basicNum);
}
```

# ES6模块与CommonJS模块差异
## CommonJs模块输出的是一个值的拷贝，ES6模块输出的是值的引用
- CommonJs模块输出的是一个**值的拷贝**，一旦输出一个值，模块内部的变化影响不到这个值
```js
// a.js
let a = 1;
let b = {num: 1};
setTimeout(() => {
    a = 2;
    b = {num: 2};
}, 200);
module.exports = {a, b};

// main.js
let {a, b} = require('./a');
console.log(a);   // 1
console.log(b);   // {num: 1}
setTimeout(() => {
    console.log(a);  // 1
    console.log(b);   // {num: 1}
}, 500);
```
**输出拷贝`exports`对象是模块内外的唯一关联，CommonJs输出的内容，就是`exports`对象的属性，模块运行结束，属性就确定了。**
- ES6模块的运行机制：JS引擎对脚本静态分析时，遇到`import`会生成一个**只读引用**。在脚本真正执行时，再根据引用到被加载的模块里去取值。ES6模块是动态引用。
```js
// a.mjs
let a = 1;
let b = {num: 1};
setTimeout(() => {
    a = 2;
    b = {num: 2};
}, 200);
export {
    a,
    b
};

// main.mjs
import {a, b} from './a';
console.log(a);  // 1
console.log(b);  // {num: 1}
setTimeout(() => {
    console.log(a);  // 2
    console.log(b);  // {num: 2}
}, 500);
```

## CommonJs模块时运行时加载，ES6模块时编译时输出
- 运行时加载： 输入时先加载整个模块，生成一个对象，再从这个对象上读取方法。
- 编译时加载： `import`时指定加载某个输出值，而不是加载整个模块。模块内部引用的变化，会反应在外部。