---
title: 面向对象编程与函数式编程
date: 2021-04-20 11:19:02
tags: oop
---

OOP与FP是不同的编程风格，了解各自的优缺点结合使用
<!--more-->

# 面向对象编程object oriented programming

把计算机程序视为一组对象的合集，每个对象都可以接收其他对象发过来的消息，并处理这些消息，计算机程序的执行就是一系列消息在各个对象之间的传递。
> 强调`状态property`与`操作method`的封装，设计思想：抽象出Class，根据Class创建Instance
## 主要特征
- 封装：把客观事物封装成抽象的类。保护私有属性与方法
- 继承：从父类获得属性和方法，实现代码复用
- 多态：同一事物的多种形态，多个对象有相同的使用方法
    - 重写overriding：子类有与父类一样的方法名，但是有自己的逻辑，子类与父类的多态性
    - 重载overloading：一个类有多个一样的方法名，但是参数不同，一个类的多态性
## 优缺点
- 优点：易维护、复用、扩展。可设计出低耦合、更灵活的系统。
## JS实现OOP
在js中使用OOP有四种方式
- 工场函数
- Function constructor构造器函数
- Object.create()
- ES6 classes
### 工场函数
创建接收参数返回对象的函数。
```js
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
### Function constructor
使用`new`关键词初始化的函数即为构造器函数，并且最好使用大写首字母
```js
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
### Object.create()
`Object.create()`可以使用一个已经存在的对象作为一个新对象的原型
```js
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
### ES6 classes
es6的class关键词可以创建oop，是一种语法糖本质依旧是原型继承
```js
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


# 函数式编程functional programming