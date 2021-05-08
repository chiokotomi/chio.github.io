---
title: 作用域与闭包与原型继承
date: 2021-05-08 13:49:08
tags: prototype
---

# 作用域scope
作用域决定了代码区块中变量和其他资源的可见性。作用域最大的用处是隔离变量，不同作用域下同名变量不会冲突。

ES6之前JS只有`全局作用域`和`函数作用域`,ES6之后有了`块级作用域`
## 全局作用域
代码中任何地方都能访问的对象有全局作用域
- 最外层函数和最外层函数外面定义的变量
```js
var outVariable = '最外层变量';
function outFun() { // 最外层函数
    // ... 
}
```
- 未定义直接赋值的变量自动声明拥有全局作用域
```js
function outFun() {
    variable = '未定义直接赋值的变量'; // 全局
    var inVariable2 = '内层变量';
}
```
- 所有window对象的属性拥有全局作用域
## 函数作用域
声明在函数内部的变量拥有局部作用域，这样函数里面的变量不会外泄暴露，不会污染到外面，例如jquery,zepto等库源码代码会放在`(function(){})()`中。
> 作用域是分层的，内层作用域可以访问外层作用域的变量，反之不行。
## 块级作用域
> 大括号`{}`中间的语句是块语句，如`if switch for while`

块级作用域可以通过新增命令`let/const`声明，所声明变量在指定块的作用域外无法访问，声明变量不会提升到当前代码块的顶部，禁止重复声明
```js
function getValue(condition) {
    if (condition) {
        let value = 'red';
        return value;
    }
    else {
        // value在此处不可用
        return null
    }
}
```
for循环的块级作用域最为必要，常见情况
```html
<button>测试1</button>
<button>测试2</button>
<button>测试3</button>
<script type="text/javascript">
    var btns = document.getElementsByTagName('button');
    // var定义的i是全局变量，执行到点击事件时i为3，点击任意按钮都弹出“第4个”
    for (var i = 0; i < btns.length; i++) {
        btns[i].onclick = function () {
            console.log('第' + (i + 1) + '个');
        }
    }
    // let声明i是块级作用域
    for (let i = 0; i < btns.length; i++) {
        btns[i].onclick = function () {
            console.log('第' + (i + 1) + '个');
        }
    }
</script>
```
## 作用域链
一层一层寻找变量取值的关系就是作用域链
## 静态作用域
如何寻找变量的取值：要到创建函数的作用域中找，即为静态作用域
```js
var a = 10;
function fn() {
    var b = 20; // b取值为fn函数作用域20，最终输出结果为30
    function bar() {
        console.log(a + b); //30
    }
    return bar;
}
var x = fn(),
    b = 200;
x(); //bar()
```
## 作用域与执行上下文

# 闭包

闭包的概念基于两点：
- 1.`Function`是js里的第一公民`first-class citizen`(支持所有操作包括：作为函数参数、函数返回值、直接赋值给变量)
<!-- more -->
```js
// 函数表达式，即将函数赋值给一个变量
const funcSing = name => {
    return `${name} sings: La la lalala`;
};
// 函数作为参数 
const letPerson = (name, fn) => {
    return fn(name);
};
letPerson('Peter', funcSing);
// 函数作为返回值
const myName = name => {
    return () => {
        return `My name is ${name}`;
    };
};
const whatsMyName = myName('Peter'); // 返回一个函数
whatsMyName();
```
- 2.js有静态作用域`lexical scope/static scope`