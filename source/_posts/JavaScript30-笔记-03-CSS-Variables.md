---
title: '[JavaScript30 笔记] 03-CSS Variables'
date: 2017-01-09 23:31:15
tags: [JavaScript30, JavaScript]
---

# Objective
利用 CSS Variable 和 JS 进行样式属性的控制。
<!-- more -->
![Demo](JavaScript30-笔记-03-CSS-Variables/js30-03.gif)

> - [Demo](https://xg-wang.github.io/JavaScript30/03%20-%20CSS%20Variables/)
> - [Source](https://github.com/xg-wang/JavaScript30/blob/master/03%20-%20CSS%20Variables/index.html)

# Steps
## CSS 部分
1. 定义全局 CSS 变量 `--spacing, --blur, --base`；
2. 将 CSS 变量添加到样式属性上。

## JS 部分
1. 对每个 `type` 类型的 `input` 标签添加监听器，触发事件有 `moutsemove`, `change`；
2. 更新全局的 CSS 属性： `document.documentElement.style.setProperty()`。

# Things learned
## CSS Variable
> [MDN 文档](https://developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_variables)
一般将 CSS 变量定义在 root element 的 pseudo-class 上，以 '--' 开头
```
:root {
    --spacing: 10px;
    --blur: 10px;
    --base: #ffc600;
}
```
在使用时只要用 `var(--blur)` 进行调用即可。

虽然 Sass/Less 中也有变量，但其是在编译期完成绑定，原生 CSS 变量可以在运行时改变，功能更强大。

## `<input>` tag
> [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input) The HTML `<input>` element is used to create interactive controls for web-based forms in order to accept data from the user. How an `<input>` works varies considerably depending on the value of its type attribute.

`<input>` 标签的基本作用是进行用户交互输入数据，根据 "type" 属性制定输入类型，种类非常丰富：
- type=button: link, menuitem, menuitemcheckbox, menuitemradio, radio, switch, tab
- type=checkbox: button, menuitemcheckbox, option, switch
- type=image: link, menuitem, menuitemcheckbox, menuitemradio, radio, switch
- type=radio: menuitemradio
- type=color|date|datetime|email|file|hidden: None
- type=month|number|password|range|research: None
- type=search|submit|tel|text|url|week: None

这个demo中用到了：
- color: 根据 OS 唤出取色器
- range: 一般用于具体数值不重要，可以根据比例估算的输入，默认值为：
    - min: 0
    - max: 100
    - value: min + (max - min) / 2, min 如果 max < min
    - step: 1
