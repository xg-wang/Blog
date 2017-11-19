---
title: Go搭建静态页面server笔记
date: 2017-11-18 21:34:25
tags: [golang, web]
---

go是一门简洁强大的语言，简单体验之后觉得对于网络和命令行的支持也非常棒，本文介绍一下go实现静态服务器的大致流程。
<!-- more -->

# 基础实现

最近接手了[gobyexample](https://github.com/xg-wang/gobyexample)的翻译工作，将项目重构后需要本地的测试环境。
由于想要页面的url显示为“`https://gobyexample.xgwang.me/hello-world`”这种结尾不带“/”的形式，子页面没有带上html，并且有图片资源因此需要一个static server。

根据[golang wiki](https://github.com/golang/go/wiki/HttpStaticFiles)，实现这个简单server只需要...一行代码:

```go
package main

import "net/http"

func main() {
	panic(http.ListenAndServe(":8080", http.FileServer(http.Dir("/usr/share/doc"))))
}
```

加入log后稍微改写一下，放在我们项目的[tools目录](https://github.com/xg-wang/gobyexample/blob/master/tools/serve.go)下：

```go
package main

import (
	"log"
	"net/http"
)

func main() {
	// Simple static webserver:
	port := ":8080"
	log.Printf("Serving at: http://localhost%s\n", port)
	err := http.ListenAndServe(port, http.FileServer(http.Dir("public")))
	if err != nil {
		log.Fatal("ListenAndServe fail:", err)
	}
}
```

再来一个可执行的`tools/serve`文件

```bash
#!/bin/bash
exec go run tools/serve.go
```

ok现在只需要`tools/serve`就可以启动这个服务器了。

# 404

一切看起来很正常，但如果我们访问一下不存在的某个页面，404.html并不会被serve，这是因为go提供的`FileServer`并不知道我们自定义的404页面。
所以我们需要将`http.FileServer`改为一个自定义的`Handler`。

写go的时候体验特别好的一点就是go官方团队提供了很opinionated的convention，比如go-get，go-fmt等。
在我们输入`http.FileServer`时会自动在`imports`中添加相应的库，跳转到源码后看到了这个函数的实现：

```go
type fileHandler struct {
	root FileSystem
}

// FileServer returns a handler that serves HTTP requests
// with the contents of the file system rooted at root.
//
// To use the operating system's file system implementation,
// use http.Dir:
//
//     http.Handle("/", http.FileServer(http.Dir("/tmp")))
//
// As a special case, the returned file server redirects any request
// ending in "/index.html" to the same path, without the final
// "index.html".
func FileServer(root FileSystem) Handler {
	return &fileHandler{root}
}

func (f *fileHandler) ServeHTTP(w ResponseWriter, r *Request) {
	upath := r.URL.Path
	if !strings.HasPrefix(upath, "/") {
		upath = "/" + upath
		r.URL.Path = upath
	}
	serveFile(w, r, f.root, path.Clean(upath), true)
}
```

于是我们知道了这里的函数需要返回的Handler有一个`ServeHTTP`方法。但是这里的`serveFile`并不能直接由`http.serveFile`调用：go规定一个package内小写字母开头的均为私有，不能被外部package访问。

但是没有关系，我们可以在`fileHandler`上再包装一层代理，在执行完我们判断文件存在的逻辑后执行原先所有`fileHandler.ServeHTTP`的内容，修改后的代码如下：

```go
type fileHandler struct {
	root http.FileSystem
	h    http.Handler
}

func fileServer(root http.FileSystem, h http.Handler) http.Handler {
	return &fileHandler{root, h}
}

func (f *fileHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	path := r.URL.Path
	if _, err := os.Stat("public/" + path); os.IsNotExist(err) {
		http.ServeFile(w, r, "public/404.html")
		return
	}
	f.h.ServeHTTP(w, r)
}

func main() {
	// Simple static webserver:
	port := ":8080"
	log.Printf("Serving at: http://localhost%s\n", port)
	fs := http.Dir("public")
	http.Handle("/", fileServer(&fs, http.FileServer(&fs)))
	err := http.ListenAndServe(port, nil)
	if err != nil {
		log.Fatal("ListenAndServe fail:", err)
	}
}
```

在传入`FileSystem`的时候传入指针也避免创建，很有C的感觉。

# 小细节

基本功能都已经实现，但作为一个命令行工具，希望再进行一些完善。

首先我们需要支持传参，go对于命令行参数的支持非常棒，只要引入builtin的flag包之后，我们加入

```go
port := flag.String("port", ":8080", "localhost port to serve")
path := flag.String("path", "public", "public files path")
flag.Parse()
```

就可以得到`*string`类型的命令行参数，并且天生支持默认值和描述，测试一下`go run tools/serve.go -h`，可以得到：

```
Usage of /var/folders/sd/cwk5fwtd4ms5vflhq5_0_5rr0000gn/T/go-build178666598/command-line-arguments/_obj/exe/serve:
  -path string
        public files path (default "public")
  -port string
        localhost port to serve (default ":8080")
```

准备serve文件之前，再输出一下带有格式的信息加粗一下我们传入的参数：

```go
log.Printf("Serving \x1b[1m%s\x1b[0m at: http://localhost\x1b[1m%s\x1b[0m\n", *path, *port)
```

这里`\x1b[0m`代表“All attributes off(color at startup)”，`\x1b[1m`代表“Bold on(enable foreground intensity)”。

# 总结

go作为静态语言拥有可以与动态语言媲美的灵活性，有完整易用的工具链和丰富的标准库，是[2017年增长最快的语言](https://blog.golang.org/8years)，简单的同时非常强大。
希望有更多的人可以一起学习go，我正在完善[Go By Example](https://gobyexample.xgwang.me/)的翻译，欢迎阅读以及贡献PR！