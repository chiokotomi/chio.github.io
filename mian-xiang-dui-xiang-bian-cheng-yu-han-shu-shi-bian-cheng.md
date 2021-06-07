---
title: 面向对象编程与函数式编程
date: '2021-04-20T11:19:02.000Z'
tags: oop
---

# 面向对象编程与函数式编程

OOP与FP是不同的编程风格，了解各自的优缺点结合使用 

## 面向对象编程object oriented programming

把计算机程序视为一组对象的合集，每个对象都可以接收其他对象发过来的消息，并处理这些消息，计算机程序的执行就是一系列消息在各个对象之间的传递。

> 强调`状态property`与`操作method`的封装，设计思想：抽象出Class，根据Class创建Instance
>
> ### 主要特征
>
> * 封装：把客观事物封装成抽象的类。保护私有属性与方法
> * 继承：从父类获得属性和方法，实现代码复用
> * 多态：同一事物的多种形态，多个对象有相同的使用方法
>   * 重写overriding：子类有与父类一样的方法名，但是有自己的逻辑，子类与父类的多态性
>   * 重载overloading：一个类有多个一样的方法名，但是参数不同，一个类的多态性

### 优缺点

* 优点：易维护、复用、扩展。可设计出低耦合、更灵活的系统。

### JS实现OOP

在js中使用OOP有四种方式

* 工场函数
* Function constructor构造器函数
* Object.create\(\)
* ES6 classes

#### 工场函数

创建接收参数返回对象的函数。

```javascript
let personA = {
    firstName: 'lorem',
    lastName: 'ipsum',
    getFullName() {
        return `Full name is: ${this.firstName} ${this.lastName}`;
    }
};
let personB = {
    firstName: 'retrum',
    lastName: 'massa',
    getFullName() {
        return `Full name is: ${this.firstName} ${this.lastName}`;
    }
};
let person = function (firstName, lastName) {
    return {
        firstName,
        lastName,
        getFullName() {
            return `Full name is ${this.firstName} ${this.lastName}`;
        }
    };
};
let personC = person('erat', 'luctus');
```

如上使用工场函数可以避免代码重复，但是因为每次使用`person`函数时，`getFullName`都被重复创建，使用Function constructor方式可以提高内存使用效率。

#### Function constructor

使用`new`关键词初始化的函数即为构造器函数，并且最好使用大写首字母

```javascript
let Person = function (firstName, lastName) {
    this.firstName = firstName;
    this.lastName = lastName
};
Person.prototype.getFullName = function() {
    return `Full name is ${this.firstName} ${this.lastName}`;
};
let personA = new Person('lorem','ipsum');
let personB = new Person('felis','ullamcorper');

personA.getFullName();
```

使用构造器与prototype实现了代码复用，`personA`与`personB`两个实例有各自不同的firstName与lastName，但是在原型上有共同的函数`getFullName`，通过原型链使得构造器函数更高效的利用了内存。

#### Object.create\(\)

`Object.create()`可以使用一个已经存在的对象作为一个新对象的原型

```javascript
let person = {
    firstName: 'retrum',
    lastName: 'massa',
    getFullName() {
        return `Full name is ${this.firstName} ${this.lastName}`;
    }
};
let personA = Object.create(person);
personA.firstName = 'ullam';
personA.lastName = 'ipsum';
personA.getFullName();
```

#### ES6 classes

es6的class关键词可以创建oop，是一种语法糖本质依旧是原型继承

```javascript
class Person {
    // 每个实例都不同的属性需要放在constructor里，所有实例共享的方法在外面
    constructor(firstName, lastName) {
        this.firstName = firstName;
        this.lastName = lastName
    }
    getFullName() {
        return `Full name is ${this.firstName} ${this.lastName}`;
    }
}
```

## 函数式编程functional programming

所有的程序里两个核心的东西是**数据data**与**行为behavior**。

函数式编程主张数据与行为分离。将代码分成不同的部分，可以保证每一个部分都能被很好的组织。简言之，应该传递数据给函数，函数操作后返回新对象。

一个函数应该满足以下特点： 1. 单任务：函数尽量的小执行单一的任务 2. 纯函数：函数不应该有副作用，相同的输入有相同的输出 3. 函数应该有返回`return statement` 4. 函数应该是可组合的`compose-able` 5. 函数应该是不变的`immutable`：函数应该返回一个数据的copy，而不应该改变原始数据 6. 函数应该是可测的`predictable`

### 纯函数

保证以下几点的是纯函数

* 1.相同的输入，永远会得到相同的输出，无论函数被执行多少次

  \`\`\`js

  function addTwoNumber\(num1, num2\) {

    return num1 + num2;

  }

addTwoNmber\(3, 4\); // will return 7 no matter how many times is called

function multiplyWithCurrentTime\(num\) { return num \* new Date\(\).getTime\(\); }

multiplyWithCurrentTime\(3\);// will provide new output each time we call the funciton

```text
- 2.计算返回值时不会产生副作用(side effect)
```js
// In this example i want to clear out the difference between function with side effect and a function without side effect
let arr = [1,2,3];
function removeLastItem(input) {
    input.pop();
}

removeLastItem(arr); 
console.log(arr); // output [1,2] (this function changes the orignal variable from [1,2,3] -> [1,2])
// above execution has side effect as it mutate arr which belong to the outside world.

function immutablyRemoveLastItem(input) {
    const newArray = [].concat(input); // concat method copies the value to new variable.
    return newArray.pop();
}

let newArr = [1,2,3];

console.log(immutablyRemoveLastItem(newArr)); // output [1,2]
console.log(newArr); // output [1,2,3] -> the above function does not have side effect as it not modify the orignal input instead it copies
// and alter the copied array.
```

### 引用透明

一个表达式可以被他的值代替并不会造成其他影响

```javascript
let a = (num1,num2) => {
    return num1+num2;
}

let b = (num) => {
    return num*2;
}

console.log(b(a(3,4))) //output will be 14
// here i can replace a(3,4) expression with value 7 value and this will not effect to the result of the program because its return
// value is 7
// so i can replace  console.log(b(a(3,4))) to console.log(b(7)) as a function is refrencially transparent

let c = (num1,num2) => {
    console.log(`Value of num1:${num1} and value of num2:${num2}`);
    return num1+num2;
}

console.log(b(c(3,4))) //output will be 14
// here i cannot replace expression c(3,4) with value 7 as it effect the result of the program
// function c has console.log() which is one type of side effect so it is not referentially transparent
```

### 幂等性

一个函数相同输入有相同输出，那么函数就是幂等的。不同于纯函数，幂等性允许函数有副作用。

```javascript
// 非幂等，每次输出都不同
function notIdempotenceFn(num) {
    return Math.random(num);
}

notIdempotenceFn(5);
notIdempotenceFn(5);

// 幂等但非纯函数
function idempotentFn(num) {
   console.log(num);
}
// 幂等
function getAbsolute(x) {
  return Math.abs(x)
}
getAbsolute(getAbsolute(getAbsolute(-50))) //50
```

### 命令式vs声明式

命令式Imperative：告诉做什么、怎么做 声明式Declarative：告诉做什么、什么需要被完成

```javascript
// Imerative命令式
for(i=1; i<=5; i++) { // 怎么做：定义变量i，直到5向上++
  console.log(i); // 做什么：console.log(i)
}
// Declarative声明式
// 做什么：console.log(item)
[1, 2, 3, 4, 5].forEach(item=>console.log(item));
```

函数式编程通过使用`compose`让我们更加偏向声明式，`compose`告诉我们程序要做什么而不是怎么去做

### Immutability不变性

不更改原始状态，而是复制一个新的状态把改变应用到新状态上后返回。

```javascript
let mutateObj = {
    first_name:'lorem',
    last_name:'ipsum'
};

let immutateObj = {
    first_name:'irum',
    last_name:'egestas'
};

const mutatingState = (obj) => {
    obj.first_name = 'ullamcorper';
    return obj;
}

const immutatingState = (obj) => {
    const copiedObj = Object.assign({},obj);
    copiedObj.first_name = 'facilisis';
    return copiedObj;
}

mutatingState(mutateObj);
console.log(mutateObj);
/*
above output {
  first_name:'ullamcorper',
  last_name:'ipsum'
};
in the above function we have mutated the orignal state
*/
const newObj = immutatingState(immutateObj);
console.log(immutateObj);
/*
newObj will be 
{
  first_name:'facilisis',
  last_name:'egestas'
};
and  immutateObj will not be changed and it value is
{
  first_name:'irum',
  last_name:'egestas'
};
here the orignal state is not change
*/
```

函数式编程建议immutability，因为immutability给我们代码带来稳定性与保护性。

### 高阶函数high order function

一个函数接收函数作为参数，或者返回是一个函数。

```javascript
const hocFn = function() {
  return function() {
    return 5;
  }
}
hocFn() // will return function
hocFn()() // will return 5
// hocFn return function so it is HOC

const fn = function(x) {
  return x*2;
}

const hocFn2 = function(fn) {
  return fn(5);
}

hocFn2(fn); // will return 10
// hocFn2 accept function as argument it is an HOC
```

### 柯里化

柯里化是把一个接收很多参数的函数，转成很多接收一个参数的函数的技巧

```javascript
const multiply = (a, b) => a * b;
multiply(4, 6);

const curring = a => b => a * b;
curring(5)(4);
```

### 局部应用

partially applying a function

```javascript
const multiply = (a,b,c) => a * b * c;

//Partial Application
const partiallyMultiplyBy5 = multiply.bind(null, 5); // 局部应用5作为第一个参数
partiallyMultiplyBy5(4, 10) //200  // 此时执行时传入剩余参数
```

### 记忆化Memoization

缓存的一种特殊形式

### 组合compose与管道pipe

`组合compose`是一种描述函数间关系的设计规则。`管道pipe`与组合类似，区别在于调用上。组合从右向左执行，管道从左向右执行。

```javascript
const multiplyWith3  = x => x * 3;
const getAbsouleOfNum = x => Math.abs(x);
// 正常实现给一个值乘3后取模
let x = 15;
let xAfterMultiply = multiplyWith3(x);
let xAfterAbsoulte = getAbsouleOfNum(xAfterMultiply);

// 通过组合的形式：安排两个函数的调用顺序
const compose = (f, g) => data => f(g(data));
const multiplyBy3andGetAbsolute = compose(multiplyWith3, getAbsouleOfNum);
multiplyBy3andGetAbsolute(-15); //45

// 通过管道的形式
const pipe = (f, g) => data => g(f(data));
const multiplyBy3andGetAbsolutePipe = pipe(multiplyWith3, getAbsouleOfNum);
multiplyBy3andGetAbsolutePipe(-15);
```

### arity参数数量

函数化编程建议使用更改少的函数参数（建议1-2个），以让函数更好用。

## OOP vs FP

OOP强调`data`与`operation`的封装，具有封装、继承、多态的主要特点。FP强调`data`与`operation`的分离，实现纯函数，避免副作用，通过函数组合实现更强的功能。

### 共同性

OOP与FP都是为了让我们的代码更可控的设计模式，代码可控意味着：

* 易读性
* 易扩展
* 易维护
* 内存高效：oop-继承，fp-闭包
* DRY（don't repeat yourself）：避免代码重复

  **不同点**

* FP 擅于处理在很多操作中修改数据；OOP 擅长处理在少操作中修改共同数据
* FP 是无状态的，不会影响程序的其他状态；OOP是有状态的，会改变属性的状态
* FP 有无副作用的纯函数；OOP因为修改自己的状态，所有有副作用
* FP 是声明式的，专注于该完成什么；OOP是命令式的，专注于如何去做

### 使用选择

* 操作多，高性能是用FP
* 角色对象多操作少的情况用OOP

