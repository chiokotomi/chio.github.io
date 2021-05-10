---
title: Web Components
date: 2021-04-26 12:01:18
tags:
---

**Web Components** 可以创建可复用的定制元素，包含三项主要技术。
<!-- more -->
1. **Custom elements （自定义元素）：**JS API，可以定义custom elements及其行为，在用户界面中按需求使用。
2. **Shadow DOM （影子DOM）：**JS API，用于将封装的影子DOM树附加到元素（与主文档DOM分开呈现）并控制其关联功能。这样可以保持元素的功能私有，可以被脚本化和样式化，不会与文档的其他部分发生冲突。
3. **HTML templates（HTML模板）：** `<template>`和`<slot>`元素可以编写不在页面中显示的标记模板，然后他们可以作为自定义元素结构的基础被多次复用。



> FF63+、Chrome、Opera支持，Safari支持部分web组件特性，Edge正在开发一个实现，ie不支持

# Custom elements

共有两种custom elements
- Autonomous custom elements： 是独立元素，不继承自其他内建的HTML元素。可以直接写成HTML标签的形式，在页面上使用。例如`<popup-info>`或者是`document.createElement("popup-info")`
- Customized built-in elements： 继承自基本的HTML元素。在创建时，必须指定所需扩展的元素。使用时，需要先写出基本的元素标签，通过`is`属性指定custom element的名称。例如`<p is="word-count">`或者`document.createElement("p", {is: "word-count"})`。

## 基本使用

注册一个custom element使用`CustomElementRegistry.define()`，接收三个参数：
- 元素名称，符合[DOMString](https://developer.mozilla.org/zh-CN/docs/Web/API/DOMString)标准的字符串，必须包含短横线。
- 用于定义元素行为的[类](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes)
- `可选参数`：一个包含`extends`属性的配置对象，指定所创建的元素继承自哪个内置元素。

```js
// Autonomous custom element
class PopUpInfo extends HTMLElement {
    constructor() {
        super();
        // ...
    }
}
customElements.define('popup-info', PopUpInfo);

<popup-info img="img/alt.png" text="some info"></popup-info>

// Customized built-in element
class ExpandingList extends HTMLUListElement {
    constructor() {
        super();
        // ...
    }
}

customElements.define('expanding-list', ExpandingList, {extends: 'ul'});

<ul is="expanding-list"></ul>
```
## 声命周期回调函数

在custom element的构造函数中，可以指定多个不同的回调函数，在元素的不同生命时期被调用：
- `connectedCallback`: 首次被插入文档DOM时，被调用
- `disconnectedCallback`： 从文档DOM中删除时，被调用
- `adoptedCallback`： 被移动到新的文档时，被调用
- `attributeChangedCallback`： 增加、删除、修改自身属性时，被调用

# shadow DOM

Shadow DOM可以将一个隐藏的、独立的DOM附加到常规的DOM树中。它以shadow root节点为起始根节点，在根节点的下方，可以是任意元素，和普通的DOM元素一样。

![](shadow-dom.png)
- **Shadow host:** 一个常规的DOM节点，Shadow DOM会被附加到这个节点上
- **Shadow tree:** Shadow DOM内部的DOM树
- **Shadow boundary:** Shadow DOM结束的地方，也是常规DOM开始的地方
- **Shadow root:** Shadow tree根节点

## 用法

`Element.attachShadow({mode: 'open'|'closed' })`将一个shadow root附加到任何一个元素上。
`open`表示可以通过页面内的js方法来获取Shadow DOM，如`Element.shadowRoot`
`closed`不可以从外部获取Shadow DOM，如浏览器内置元素如`<video></video>`

> 常规[DOM（文档对象模型）](https://developer.mozilla.org/zh-CN/docs/Web/API/Document_Object_Model/Introduction)
# templates and slots

## templates

模板可以复用相同的结构。

```js
<template id="my-paragraph">
    <p>My paragraph</p>
</template>
```
上面代码不会出现在页面中，直到使用js获取它的引用，然后添加到DOM中
```js
let template = document.getElementById('my-paragraph');
let templateContent = template.content;
document.body.appendChild(templateContent);
```
## slots

插槽能在单个实例中通过声明式的语法展示不同的文本。
slots由`name`属性标识，并且在模板中定义占位符。在实例中使用slot标记时，占位符可以填充为所需的任何HTML标记片段。

```js
<html>
    <head></head>
    <body>
        <template id="my-paragraph">
            <p>
                <slot name="my-text">
                    My default text
                </slot>
            </p>
        </template>

        <my-paragraph>
            <span slot="my-text">Let's have some different text</span>
        </my-paragraph>

        <my-paragraph>
            <ul slot="my-text">
                <li>list 1</li>
                <li>list 2</li>
            </ul>
        </my-paragraph>
    </body>
</html>
```