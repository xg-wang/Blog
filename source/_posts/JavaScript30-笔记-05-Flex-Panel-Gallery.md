---
title: '[JavaScript30 笔记] 05 - Flex Panel Gallery'
date: 2017-01-11 23:33:18
tags: [JavaScript30, JavaScript]
---

# 写在系列开篇
在学习了基本的 HTML / CSS / JavaScript 之后没有啥 side project 就直接开始学 Angular 做项目。遇到最近很火的[JavaScript30](https://JavaScript30.com)这个项目后决心好好实践一下 Vanilla JavaScript。

对于每个挑战，我会自己在看过视频了解原理后自己实现一遍并在 [blog](https://xg-wang.github.io/tags/JavaScript30/) 里记录过程、想法和相关资料。源码放在我的 [Github](https://github.com/xg-wang/JavaScript30)上，demo 通过 Github Pages 部署，点击[这里](https://xg-wang.github.io/JavaScript30/)或 Github 的 readme 中连接可以访问。

# Objective
利用 Flexbox 实现一个 Gallery，点击一个 panel 后可放大，并飞入上下的文字。
<!-- more -->
![demo](./JavaScript30-笔记-05-Flex-Panel-Gallery/05-Flex-Panel-Gallery.gif)

> - [Demo](https://xg-wang.github.io/JavaScript30/05%20-%20Flex%20Panel%20Gallery/)
> - [Source](https://github.com/xg-wang/JavaScript30/blob/master/05%20-%20Flex%20Panel%20Gallery/index.html)

# Steps
1. 为包裹 5 个子 panel 的 div 添加 flex 相关的属性:
    ```
    flex: 1;
    display: flex;
    flex-direction: column;
    justify-content: center; 
    ```
    并为每个子 panel 设置 flex-grow 为 1 使占满屏幕宽度.
2. 居中子 panel 的所有 p tag 内容
    ```
    display: flex;
    justify-content: center;
    align-items: center;
    ```
3. 将子 panel 上下文字移出：
    ```
    .panel p:first-child { transform: translateY(-100%) }
    .panel p:last-child { transform: translateY(100%) }
    ```
4. 准备点击后添加的类，设置点击 panel 的 flex 为 5，并设置 translateY(0) 移入上下文字。
5. JavaScript 部分添加对应的 EventListener。

# Things learned
## Flexbox
flex 已经成为目前 CSS layout 的必备了，1月份 [Bootstrap v4-alpha6](http://blog.getbootstrap.com/2017/01/06/bootstrap-4-alpha-6/) 也已经全面使用 flex 进行布局。

因为之前专门学过 flex，这里进行的还是很轻松，相关的知识点网上有很多[资源](https://css-tricks.com/snippets/css/a-guide-to-flexbox/)讲解。

## Center
panel 里面每个 p tag 内文字需要在各自的区域内居中，这里可以使用 flex 轻松解决。更多的解决方案可以参考[css tricks](https://css-tricks.com/centering-css-complete-guide/)。

## fly-in fly-out
利用 `transform: translateY(-100%)` 将元素向上平移，添加 transitioned 事件的监听器可以移回。

注意原作者 Wes 这里有个 [bug](https://github.com/wesbos/JavaScript30/blob/master/05%20-%20Flex%20Panel%20Gallery/index-FINISHED.html#L136)，当快速双击 panel 时文字会停留在 panel 内。解决方案有两个：
1. 对 tranform 的 transition 设置延时
2. 在 transitioned 事件监听器内使 opened class 的 toggle 依赖于 open class 的存在。
我的[代码](https://github.com/xg-wang/JavaScript30/blob/master/05%20-%20Flex%20Panel%20Gallery/index.html#L133)中使用了方案2。
