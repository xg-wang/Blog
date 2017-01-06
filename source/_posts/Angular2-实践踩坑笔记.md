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
