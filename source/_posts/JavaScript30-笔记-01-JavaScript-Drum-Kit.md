---
title: '[JavaScript30 笔记] 01-JavaScript Drum Kit'
date: 2017-01-05 11:28:35
tags: [JavaScript30, JavaScript]
---

# 写在系列开篇
在学习了基本的 HTML / CSS / JavaScript 之后没有啥 side project 就直接开始学 Angular 做项目。遇到最近很火的[JavaScript30](https://JavaScript30.com)这个项目后决心好好实践一下 Vanilla JavaScript。

对于每个挑战，我会自己在看过视频了解原理后自己实现一遍并在 blog 里记录过程、想法和相关资料。源码放在我的 [Github](https://github.com/xg-wang/JavaScript30)上，demo 通过 Github Pages 部署，点击[这里](https://xg-wang.github.io/JavaScript30/)或 Github 的 readme 中连接可以访问。

# Objective
通过预先提供的音频文件和 JavaScript 实现键盘按键打鼓，按下预定的键位后进行高亮并在高亮转换结束后恢复。

# Steps
1. 添加全局的监听器，播放按键相应的音频；
2. 播放音频前对按键 dom 元素高亮；
3. 对所有按键 dom 元素添加监听器，在 transition 结束后恢复原先样式。

> - [Demo](https://xg-wang.github.io/JavaScript30/01%20-%20JavaScript%20Drum%20Kit/)
> - [Source](https://github.com/xg-wang/JavaScript30/blob/master/01%20-%20JavaScript%20Drum%20Kit/index.html)

# Things learned
## data-*
通过 data-* (这里是data-key) 对 dom 元素进行标识并与数据对应起来。这个小技巧在面试 LinkedIn 的前端实习时候也被考到了。
在选取时即可用` const key = document.querySelector(`div[data-key="${e.keyCode}"]`);`来获得元素。

## HTML5 Audio
HTML 部分通过 HTML5 的 `<audio>` 标签嵌入音频
```
<audio data-key="65" src="sounds/clap.wav"></audio>
<audio data-key="83" src="sounds/hihat.wav"></audio>
<audio data-key="68" src="sounds/kick.wav"></audio>
<audio data-key="70" src="sounds/openhat.wav"></audio>
<audio data-key="71" src="sounds/boom.wav"></audio>
<audio data-key="72" src="sounds/ride.wav"></audio>
<audio data-key="74" src="sounds/snare.wav"></audio>
<audio data-key="75" src="sounds/tom.wav"></audio>
<audio data-key="76" src="sounds/tink.wav"></audio>
```
JavaScript 部分在获取了元素后可以设置 `currentTime`，调用`play()`方法等等。
通过设置 `currentTime = 0` 可以保证每次点击后都重新从头开始播放。
```
const audio = document.querySelector(`audio[data-key="${e.keyCode}"]`);
audio.currentTime = 0;
audio.play();
```

[MDN文档](https://developer.mozilla.org/en-US/docs/Web/Guide/HTML/Using_HTML5_audio_and_video)

## 取消样式
一种可能的方案是设置定时器进行 css 的 `remove`，但对于按住不放的情况这样处理不是很好。通过 [transitioned事件](https://developer.mozilla.org/en-US/docs/Web/Events/transitionend) 可以在 css transition 结束时调用 `e.target.classList.remove('playing');` 取消样式。

