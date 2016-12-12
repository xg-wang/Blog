---
title: JavaScript Array sort()
date: 2016-12-11 18:46:39
tags:
---

今天在 Leetcode 刷题踩了一个 JS 的小坑，记录一下。

```JavaScript
var a = [823564440,115438165,784484492,74243042,114807987,137522503,441282327,16531729,823378840,143542612]

a.sort()
a.sort((a,b) => a-b)
```

第一感觉这两个应该会得到相同的结果，然而实际上是不一样的：

```
a.sort()
[114807987, 115438165, 137522503, 143542612, 16531729, 441282327, 74243042, 784484492, 823378840, 823564440]
a.sort((a, b) => a-b)
[16531729, 74243042, 114807987, 115438165, 137522503, 143542612, 441282327, 784484492, 823378840, 823564440]
```

很明显默认的`sort()`是按字符串比较，根据[MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/sort)，

> The sort() method sorts the elements of an array in place and returns the array. The sort is not necessarily stable. The default sort order is according to **string Unicode code points**.

这背后的逻辑是：JS 数组支持所有类型的元素，统一转换成 String 类型进行比较会比较方便。
因此在对 numerical array 进行 sort 时一定要定义自己的 comparing function。
