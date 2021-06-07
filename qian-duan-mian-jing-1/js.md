# JS

## Q1 - js中的数据类型有哪些

基本数据类型：`Number` `String` `Boolean` `Null` `Undefined` `Symbol`\(ES6唯一标识符，可用作对象的唯一属性名,通过`Symbol()`函数生成\)

引用数据类型（地址访问）：`Object`

```javascript
// Symbol 是唯一值，不相等。
let a = {},
    b = Symbol(0),
    c = Symbol(0);
a[b] = 'hello';
a[c] = 'world';
console.log(a[b]); //hello

// 对象作为键时，都会转化成字符串'[object Object]'
let a = {},
    b = {},
    c = {};
a[b] = 'hello';
a[c] = 'world';
console.log(a[b]); //world

// 对象是引用类型，a，b 中存储的都是同一个地址，改变的是同一个对象。
let a = { m: 1 },
    b = a;
b.m = 2;
console.log(a.m); //2

// 函数传参是值传递，即传递的是地址，根据地址改变对象的属性，改变的是同一个对象。
let a = { m: 1 };
function fn(obj) {
  let a = obj;
  a.m = 3;
}
fn(a);
console.log(a.m); //3
```

## Q2 - 聊一下深拷贝与浅拷贝

基于Object引用类型的特点

* 浅拷贝：只是拷贝了对象的第一层，属性值若仍是引用类型，则只拷贝其地址，因而两个对象仍旧有联系。ES6 里的扩展运算符`...`就是浅拷贝
* 深拷贝： 对于每一层都拷贝，`JSON.parse(JSON.stringify(obj))`可以简单实现，但是有局限性，可以用递归实现，lodash有实现方法

  ```javascript
  function clone(target) { 
    if (typeof target === 'object') { 
        let cloneTarget = Array.isArray(target) ? [] : {}; 
        for (const key in target) { 
            cloneTarget[key] = clone(target[key]); 
        } 
        return cloneTarget; 
    } 
    else { 
        return target; 
    } 
  };
  ```

## Q3 - 说下JS作用域

作用域决定了资源的可见性。ES6之前有`全局作用域`和`函数作用域`，ES6后有了`块级作用域`

* 全局作用域：任何地方都可以访问，比如最外层函数与变量、windows对象
* 函数作用域：函数内声明的变量
* 块级作用域：`{}`中间的语句，通过`let/const`声明，在for循环中定义的变量会将作用域限制在`{}`中，而通过`var`定义的会将作用域作用在for外面

> 作用域链： 一层一层寻找变量取值的关系就是作用域链 执行上下文： 在代码执行阶段创建（作用域在定义时确定）

## Q4 - JS的原型与继承

* 原型：每个对象都有一个内部属性`[[Prototype]]`，通过对象的`__proto__`属性可以访问。
* 原型链：访问一个对象的属性时，会逐级搜索链接对象的原型，直到末尾。

## Q5 - js中的this

* call一个函数时，传入的context

  \`\`\`js

  func\(p1, p2\)

  func.call\(undefined, p1, p2\)

obj.child.method\(p1, p2\) obj.child.method.call\(obj.child, p1, p2\)

func.call\(context, p1, p2\)

```text
- 箭头函数里没有this的定义

## Q6 - js中的call apply bind
```js
fn.call(target, arg1, arg2, ...) 
fn.apply(target, [arg1, arg2, ...])
fn.bind(target)
```

* 都是让target可以调用fn
* call与apply都是立即执行
* bind是返回一个函数
* this指向都是target

## Q7 - ES6 - promise

* 异步操作,接受的参数是函数。
* 本身有状态：pending/fulfilled/rejected
* 包含resolve\(value\)与reject\(reason\)两个方法
* 包含then方法可以链式调用
* API:
  * Promise.all\(\[\]\)： 所有成功
  * Promise.allSetteld\(\[\]\): 所有成功或失败
  * Promise.any\(\[\]\)： 任一成功
  * Promise.race\(\[\]\): 任一成功或失败
* 实现Promise.all

  ```javascript
  Promise.all = function (promiseArr) {
    return new Promise(resolve, reject) {
        let resArr = [];
        for (let i = 0; i < promiseArr.length; i++>) {
            Promise.resolve(promiseArr[i]).then(res => {
                resArr[i] = res;
                if (resArr.length === promiseArr.length) {
                    resolve(resArr);
                }
            }, err => {
                reject(err);
            });
        }
    };
  }
  ```

## Q8 - ES6 Map

* Map 的键可以任何类型，Object 只能是字符串或者 Symbol
* Map 中的 key 是有序的，而 Object 的键是无序的
* map.set\(\)/map.size\(\)

## Q9 - let const 与 var

* let/const 实现块级作用域
* const 声明常量不许更改，如果是object可以更改属性

## Q10 - 为什么0.1+0.2 !== 0.3/ 精度丢失

js数字相加时会转成二进制再运算，无限循环在截断时导致丢失精度。解决方法：

```javascript
const add = (a, b) => {
    return parseFload((a + b).toFixed(2));
}
console.log(add(0.1, 0.2) === 0.3)
```

## Q11 - 实现防抖、节流

参考lodash的`debounce防抖`与`throttle节流`

## Q12 - 聊一下闭包

函数可以使用其父函数内的变量，一个函数返回了一个函数后，其内部的局部变量还被新函数引用

## Q13 - 宏任务与微任务

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

## Q14 - 聊下函数式编程、柯里化

