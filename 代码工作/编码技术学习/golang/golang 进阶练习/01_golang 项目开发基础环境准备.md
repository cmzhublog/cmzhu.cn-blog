## golang 基础环境准备

### golang 环境安装

参照对应地址的安装方式进行安装即可

[https://go.dev/doc/install](https://go.dev/doc/install)

### golang 环境初始化

本次将跟随[https://golang2.eddycjy.com/posts/ch2/01-simple-server/](https://golang2.eddycjy.com/posts/ch2/01-simple-server/) 使用golang 开发一个个人博客系统

开始之前需要跟随大佬初始化项目

```bash
$ go mod init github.com/cmzhublog/golang-training/src/blog
```

执行完命令之后， 就完成了初始化的第一步, 会在当前文件夹下生成一个go.mod 的文件

```bash
$ ls 
go.mod
$ cat go.mod 
module github.com/cmzhublog/golang-training/src/blog

go 1.22.2
```

### 安装相应的模块

接下来安装网站需要使用到的gin 相关的模块， 安装过程需要使用github网站， 需要使用科学方式

```bash
$ go get -u github.com/gin-gonic/gin@v1.6.3
go: downloading github.com/gin-gonic/gin v1.6.3
go: downloading github.com/gin-contrib/sse v0.1.0
go: downloading github.com/mattn/go-isatty v0.0.12
go: downloading github.com/go-playground/validator/v10 v10.2.0
go: downloading github.com/golang/protobuf v1.3.3
go: downloading github.com/ugorji/go/codec v1.1.7
go: downloading gopkg.in/yaml.v2 v2.2.8
go: downloading github.com/json-iterator/go v1.1.9
go: downloading github.com/ugorji/go v1.1.7
go: downloading github.com/go-playground/validator v9.31.0+incompatible
go: downloading github.com/json-iterator/go v1.1.12
go: downloading gopkg.in/yaml.v2 v2.4.0
go: downloading github.com/golang/protobuf v1.5.4
go: downloading github.com/mattn/go-isatty v0.0.20
go: downloading github.com/ugorji/go v1.2.12
go: downloading github.com/ugorji/go/codec v1.2.12
go: downloading github.com/modern-go/concurrent v0.0.0-20180228061459-e0a39a4cb421
go: downloading github.com/modern-go/reflect2 v0.0.0-20180701023420-4b7aa43c6742
go: downloading golang.org/x/sys v0.0.0-20200116001909-b77594299b42
go: downloading github.com/go-playground/validator/v10 v10.19.0
go: downloading github.com/leodido/go-urn v1.2.0
go: downloading github.com/go-playground/universal-translator v0.17.0
go: downloading github.com/modern-go/concurrent v0.0.0-20180306012644-bacd9c7ef1dd
go: downloading github.com/modern-go/reflect2 v1.0.2
go: downloading github.com/go-playground/universal-translator v0.18.1
go: downloading github.com/leodido/go-urn v1.4.0
go: downloading github.com/go-playground/locales v0.13.0
go: downloading golang.org/x/sys v0.19.0
go: downloading github.com/go-playground/locales v0.14.1
go: downloading github.com/gabriel-vasile/mimetype v1.4.3
go: downloading golang.org/x/crypto v0.19.0
go: downloading golang.org/x/text v0.14.0
go: downloading google.golang.org/protobuf v1.33.0
go: downloading golang.org/x/crypto v0.22.0
go: downloading golang.org/x/net v0.21.0
go: downloading golang.org/x/net v0.24.0
go: added github.com/gabriel-vasile/mimetype v1.4.3
go: added github.com/gin-contrib/sse v0.1.0
go: added github.com/gin-gonic/gin v1.6.3
go: added github.com/go-playground/locales v0.14.1
go: added github.com/go-playground/universal-translator v0.18.1
go: added github.com/go-playground/validator/v10 v10.19.0
go: added github.com/golang/protobuf v1.5.4
go: added github.com/json-iterator/go v1.1.12
go: added github.com/leodido/go-urn v1.4.0
go: added github.com/mattn/go-isatty v0.0.20
go: added github.com/modern-go/concurrent v0.0.0-20180306012644-bacd9c7ef1dd
go: added github.com/modern-go/reflect2 v1.0.2
go: added github.com/ugorji/go/codec v1.2.12
go: added golang.org/x/crypto v0.22.0
go: added golang.org/x/net v0.24.0
go: added golang.org/x/sys v0.19.0
go: added golang.org/x/text v0.14.0
go: added google.golang.org/protobuf v1.33.0
go: added gopkg.in/yaml.v2 v2.4.0
```

安装完成后， 我们查看go.mod 文件， 会发生变化， 打开文件查看如下： 下图中正是安装gin 所需要的依赖

> 为什么每一个仓库下面都有一个 indirect 的字样， 这是因为我们下载了该项目但是没有使用

```bash
$ cat go.mod
module github.com/cmzhublog/golang-training/src/blog

go 1.22.2

require (
	github.com/gabriel-vasile/mimetype v1.4.3 // indirect
	github.com/gin-contrib/sse v0.1.0 // indirect
	github.com/gin-gonic/gin v1.6.3 // indirect
	github.com/go-playground/locales v0.14.1 // indirect
	github.com/go-playground/universal-translator v0.18.1 // indirect
	github.com/go-playground/validator/v10 v10.19.0 // indirect
	github.com/golang/protobuf v1.5.4 // indirect
	github.com/json-iterator/go v1.1.12 // indirect
	github.com/leodido/go-urn v1.4.0 // indirect
	github.com/mattn/go-isatty v0.0.20 // indirect
	github.com/modern-go/concurrent v0.0.0-20180306012644-bacd9c7ef1dd // indirect
	github.com/modern-go/reflect2 v1.0.2 // indirect
	github.com/ugorji/go/codec v1.2.12 // indirect
	golang.org/x/crypto v0.22.0 // indirect
	golang.org/x/net v0.24.0 // indirect
	golang.org/x/sys v0.19.0 // indirect
	golang.org/x/text v0.14.0 // indirect
	google.golang.org/protobuf v1.33.0 // indirect
	gopkg.in/yaml.v2 v2.4.0 // indirect
)
```

### 快速启动一个gin 项目

在完成前期配置之后， 本节需要运行一个demo 出来， 使用一个main.go 文件起一个服务出来是什么样子的？具体代码如下：

```go
package main

import (
	"gin"
)

func main() {
	r := gin.Default()
	r.GET("/ping", func(c *gin.Context){
		c.Json(200, gin.H{
			"message": "pong"
		})
	})
	r.Run()
}
```

