---
title: Angular2 实践踩坑笔记
date: 2016-12-21 21:14:35
tags: [Angular]
---

# Template event emission
在传子component事件触发函数时`(onSelect)="onSelectStart($event)"`必须写成`$event`，
然后在ts文件中可以把参数的类型定义为正确的类型。

即使是其他类型的值也要写成完整的`$event`，不然无法读取。

# Routing wildcard
在 `appRoutes` 中我们一般要使用 "wildcard" 设置一下空白页的引导，对此官方文档有很好的[例子](https://angular.io/docs/ts/latest/guide/router.html#!#base-href)。

但这里只提到了但 Routing Module 的情况，实际中遇到需要配置多个 Routing Module时需要import多个Routes。
例如：
```
const appRoutes: Routes = [
  { path: '', redirectTo: 'pages', pathMatch: 'full' },
  { path: '**', redirectTo: 'pages/overview' }
];
// ...
const pagesRoutes: Routes = [
  {
    path: 'pages',
    component: PagesComponent,
    children: [
      { path: '', redirectTo: 'overview', pathMatch: 'full' },
      { path: 'overview', loadChildren: 'app/pages/overview/overview.module#OverviewModule' },
      { path: 'setting', loadChildren: 'app/pages/setting/setting.module#SettingModule' }
    ]
  }
];
```

开发中遇到一个问题就是无论routerLink导向哪里实际都会被引导到空白页。
其原因在于`app.module.ts`中的`import`里面必须将`AppRoutingModule`放在其他含有router的Module之后，这样其他有效的router就可以被先匹配到。

```
@NgModule({
  //...
  imports: [
    BrowserModule,
    FormsModule,
    HttpModule,
    PagesModule,
    AppRoutingModule
  ],
  //...
```

# [Typescript] number from object
在获取Chartjs的label时，因为label是String类型，我想拿到数字对应的number，于是使用了
`let label: number = ......label`
之后发生了label实际上还是String的问题，debug了好久才发现，就连VSCode的tslint都没有发现这个问题。

之后在`ts-node`中试验了一下，

```
> let a: any = {b: "2"}
undefined
> a
{ b: '2' }
> let num: number = a.b
undefined
> num
'2'
```

可见number类型的变量使用any的元素赋值时不会触发错误。

有时间还是应该系统学习一下Typescript，大型工程中还是很有用的。

# `[hidden]` vs `*ngIf`
对于是否显示某个 DOM 元素或者 Component，可以用默认的 Directive `[hidden]` 或者是 `*ngIf`。
但两者有一些差别在使用中需要注意：

- hidden 的实现原理是 `display: none;`
- ngIf 的原理是对 DOM 元素销毁与构造

所以对于初始化时需要 DOM 元素存在的一些部件就需要仔细考虑，
例如 Chartjs 在构造时需要读取父元素的 Size，如果使用 hidden 来控制显示就会出现没有绘制的情况 - 初始化时父元素的尺寸是 0。
