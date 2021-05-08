---
title: 面向对象编程与函数式编程
date: 2021-04-20 11:19:02
tags: oop
---

# 面向对象编程与函数式编程
[oop vs fp](https://dev.to/bhaveshdaswani93/oop-vs-fp-with-javascript-39jf)


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
# 函数式编程functional programming