---
title: '[JavaScript30 笔记] 02-JS + CSS Clock'
date: 2017-01-06 23:29:49
tags: [JavaScript30, JavaScript]
---

# 写在系列开篇
在学习了基本的 HTML / CSS / JavaScript 之后没有啥 side project 就直接开始学 Angular 做项目。遇到最近很火的[JavaScript30](https://JavaScript30.com)这个项目后决心好好实践一下 Vanilla JavaScript。

对于每个挑战，我会自己在看过视频了解原理后自己实现一遍并在 [blog](https://xg-wang.github.io/tags/JavaScript30/) 里记录过程、想法和相关资料。源码放在我的 [Github](https://github.com/xg-wang/JavaScript30)上，demo 通过 Github Pages 部署，点击[这里](https://xg-wang.github.io/JavaScript30/)或 Github 的 readme 中连接可以访问。

# Objective
利用 CSS 和 JS 实现一个时钟界面，每秒更新三个指针的位置。
<!-- more -->
![Demo](JavaScript30-笔记-02-JS-CSS-Clock/js30-02.gif)

> - [Demo](https://xg-wang.github.io/JavaScript30/02%20-%20JS%20+%20CSS%20Clock)
> - [Source](https://github.com/xg-wang/JavaScript30/blob/master/02%20-%20JS%20%2B%20CSS%20Clock/index.html)

# Steps
## CSS 部分
1. 设置指针旋转的原点`transform-origin: right;`
2. 设置 CSS transition 模拟时钟指针抖动；
3. 添加 CSS 样式区别时针秒针。

## JS 部分
1. 获取 Date 对象得到时间并计算需要转动的角度；
2. 设置三个指针的 transform 属性为 `rotate(${xxxDegree}deg)`；
3. 在转动角度归零(90deg)时特殊处理 transition。

# Things learned
## CSS tranform
[MDN document](https://developer.mozilla.org/en-US/docs/Web/CSS/transform)
通过设置 `transform` 为 `translateX/Y` 进行平移，`rotate` 进行旋转，`transform-origin` 指定旋转的圆心。

[transform function](https://developer.mozilla.org/en-US/docs/Web/CSS/transform-function)涵盖了几乎所有几何变换，具体见文档。

## transition-timing-function
一般 `transition-timing-function` 可以设置为 `ease` 之类的。利用 `dev-tools` 可以图形化手动调节曲线的到想要的 `cubic-bezier`。

## 角度归零的处理
以秒针为例，
因为设置了 transition，在秒针从 444 跳动到下一秒时不是 450 度，而是归零为 0 + 90 = 90 度。设置的 transition 实际执行结果是指针逆时针倒转到90度。

视频中 Wes 提到了两种方法：
1. 归零时暂时取消 transition；
2. 角度只增不减。

[Soyaine](https://github.com/soyaine/JavaScript30/tree/master/02%20-%20JS%20%2B%20CSS%20Clock)使用的解法是暂时取消 transition：
```
if (secondDeg === 90) secHand.style.transition = 'all 0s';
else secHand.style.transition = 'all 0.05s';
```

但这样实际结果是那一秒没有 transition 效果，我改进了一下加入两个 `setTimeout` 完成角度的归零。
```
if (secDegree === 444) {
    setTimeout(() => {
        secHand.style.transition = 'all 0s';
        secHand.style.transform = `rotate(${secDegree-360}deg)`;
    }, 100);
    setTimeout(() => {
        secHand.style.transition = 'all .05s';
    }, 500);
}
```

