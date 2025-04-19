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
	"github.com/gin-gonic/gin"
)

func main() {
	r := gin.Default()
	r.GET("/ping", func(c *gin.Context){
		c.JSON(200, gin.H {"message": "pong"})
	})
	r.Run()
}
```

运行结果

```bash
go run main.go
[GIN-debug] [WARNING] Creating an Engine instance with the Logger and Recovery middleware already attached.

[GIN-debug] [WARNING] Running in "debug" mode. Switch to "release" mode in production.
 - using env:   export GIN_MODE=release
 - using code:  gin.SetMode(gin.ReleaseMode)

[GIN-debug] GET    /ping                     --> main.main.func1 (3 handlers)
[GIN-debug] Environment variable PORT is undefined. Using port :8080 by default
[GIN-debug] Listening and serving HTTP on :8080
[GIN] 2024/04/29 - 10:08:23 | 200 |    1.002083ms |       127.0.0.1 | GET      "/ping"
[GIN] 2024/04/29 - 10:08:24 | 404 |        4.75µs |       127.0.0.1 | GET      "/favicon.ico"
```

启动服务后， 输出了很多信息 

- 默认engine 实例： 默认使用了官方提供的Logger 和Recovery 中间创建了Engine 实例。
- 运行模式： 当前为调试模式， 建议若在生产环境切换成发布模式
- 路由注册： 注册了由get 方法 的`ping` 路由， 并输出其调用方法的方法名
- 运行信息： 本次启动时监听了 `8080` 端口。由于没有设置端口号， 默认为80 端口

### Gin架构分析

![image](./01_golang%20%E9%A1%B9%E7%9B%AE%E5%BC%80%E5%8F%91%E5%9F%BA%E7%A1%80%E7%8E%AF%E5%A2%83%E5%87%86%E5%A4%87.assets/gin-analysis.jpg)

#### gin.Default

```go
func Default() *Engine {
  debugPrintWARNINGDefault()
  engine := New()
  engine.Use(Logger(), Recovery())
  return engine
}
```

我们会通过调用 `gin.Default` 来创建默认的Engine 实例，会在初始化阶段引入Logger 和Recovery 中间件， 能够保障应用程序的基本运作， 

- Logger： 输出请求日志， 标准化日志格式
- Recovery： 异常捕获， 也就是针对每次请求进行recovery 处理， 防止出现panic 导致服务崩溃， 并标准化异常日志格式。

调用debugPrintWARNINGDefault 方法时， 会判断go 版本是否达到gin 的要求,进行调试时， 有以下日志输出， 用于提醒开发人员框架内部已经默认检查和集成了缺省值

```log
[WARNING] Creating an Engine instance with the Logger and Recovery middleware already attached.
```



#### gin.New

最重要的莫过于 `New` 方法，因为该方法会进行 Engine 实例的初始化动作并返回，它在 gin 中承担了主轴的作用，那在初始化时会设置什么参数，是否会影响到我们的日常开发呢，我们继续一起深入探究，代码如下

```go
func New() *Engine {
	debugPrintWARNINGNew()
	engine := &Engine{
		RouterGroup: RouterGroup{
			Handlers: nil,
			basePath: "/",
			root:     true,
		},
		FuncMap:                template.FuncMap{},
		RedirectTrailingSlash:  true,
		RedirectFixedPath:      false,
		HandleMethodNotAllowed: false,
		ForwardedByClientIP:    true,
		AppEngine:              defaultAppEngine,
		UseRawPath:             false,
		UnescapePathValues:     true,
		MaxMultipartMemory:     defaultMultipartMemory,
		trees:                  make(methodTrees, 0, 9),
		delims:                 render.Delims{Left: "{{", Right: "}}"},
		secureJsonPrefix:       "while(1);",
	}
	engine.RouterGroup.engine = engine
	engine.pool.New = func() interface{} {
		return engine.allocateContext()
	}
	return engine
}
```

- RouterGroup：路由组，所有的路由规则都由 `*RouterGroup` 所属的方法进行管理，在 gin 中和 Engine 实例形成一个重要的关联组件。
- RedirectTrailingSlash：是否自动重定向，如果启用了，在无法匹配当前路由的情况下，则自动重定向到带有或不带斜杠的处理程序去。例如：当外部请求了 `/tour/` 路由，但当前并没有注册该路由规则，只有 `/tour` 的路由规则时，将会在内部进行判定，若是 HTTP GET 请求，将会通过 HTTP Code 301 重定向到 `/tour` 的处理程序去，但若是其他类型的 HTTP 请求，那么将会是以 HTTP Code 307 重定向。，通过指定的 HTTP 状态码重定向到 `/tour` 路由的处理程序去。
- RedirectFixedPath：是否尝试修复当前请求路径，也就是在开启的情况下，gin 会尽可能的帮你找到一个相似的路由规则并在内部重定向过去，主要是对当前的请求路径进行格式清除（删除多余的斜杠）和不区分大小写的路由查找等。
- HandleMethodNotAllowed：判断当前路由是否允许调用其他方法，如果当前请求无法路由，则返回 Method Not Allowed（HTTP Code 405）的响应结果。如果无法路由，也不支持重定向其他方法，则交由 NotFound Hander 进行处理。
- ForwardedByClientIP：如果开启，则尽可能的返回真实的客户端 IP，先从 X-Forwarded-For 取值，如果没有再从 X-Real-Ip。
- UseRawPath：如果开启，则会使用 `url.RawPath` 来获取请求参数，不开启则还是按 url.Path 去获取。
- UnescapePathValues：是否对路径值进行转义处理。
- MaxMultipartMemory：相对应 `http.Request ParseMultipartForm` 方法，用于控制最大的文件上传大小。
- trees：多个压缩字典树（Radix Tree），每个树都对应着一种 HTTP Method。你可以理解为，每当你添加一个新路由规则时，就会往 HTTP Method 对应的那个树里新增一个 node 节点，以此形成关联关系。
- delims：用于 HTML 模板的左右定界符。

总的来讲，Engine 实例就像引擎一样，与整个应用的运行、路由、对象、模板等管理和调度都有关联，另外通过上述的解析，你可以发现其实 gin 在初始化默认已经替我们做了很多事情，可以说是既定了一些默认运行基础。

#### r.GET

注册路由时， 使用r.GET 方法来将定义的路由注册进去

```go
func (group *RouterGroup) handle(httpMethod, relativePath string, handlers HandlersChain) IRoutes {
	absolutePath := group.calculateAbsolutePath(relativePath)
	handlers = group.combineHandlers(handlers)
	group.engine.addRoute(httpMethod, absolutePath, handlers)
	return group.returnObj()
}
```

- 计算路由的绝对路径 `group.basePath`
- 合并现有的Handler 并创建一个函数链， HandlersChain
- 将当前新注册的路由规则， 追加到对应的树中

创建函数链 HandlersChain 的详细步骤:

```go
func (group *RouterGroup) combineHandlers(handlers HandlersChain) HandlersChain {
	finalSize := len(group.Handlers) + len(handlers)
	mergedHandlers := make(HandlersChain, finalSize)
	copy(mergedHandlers, group.Handlers)
	copy(mergedHandlers[len(group.Handlers):], handlers)
	return mergedHandlers
}

func (group *RouterGroup) Use(middleware ...HandlerFunc) IRoutes {
	group.Handlers = append(group.Handlers, middleware...)
	return group.returnObj()
}
```

###  r.Run

支撑使用HTTPServer 的Run方法：

```go
func (engine *Engine) Run(addr ...string) (err error) {
	defer func() { debugPrintError(err) }()

	address := resolveAddress(addr)
	debugPrint("Listening and serving HTTP on %s\n", address)
	err = http.ListenAndServe(address, engine)
	return
}
```

该方法会通过解析地址，再调用 `http.ListenAndServe` 将 Engine 实例作为 `Handler` 注册进去，然后启动服务，开始对外提供 HTTP 服务。

这里值得关注的是，为什么 Engine 实例能够传进去呢，明明形参要求的是 `Handler` 接口类型。这块如果你有了解过 Go 语言的相关基础知识的话，应该会知道，实际上在 Go 语言中如果某个结构体实现了 interface 定义声明的那些方法，那么就可以认为这个结构体实现了 interface。

那么在 gin 中，Engine 这一个结构体确确实实是实现了 `ServeHTTP` 方法的，也就是符合 `http.Handler` 接口标准，代码如下：

```go
func (engine *Engine) ServeHTTP(w http.ResponseWriter, req *http.Request) {
	c := engine.pool.Get().(*Context)
	c.writermem.reset(w)
	c.Request = req
	c.reset()

	engine.handleHTTPRequest(c)
	engine.pool.Put(c)
}
```

- 从 `sync.Pool` 对象池中获取一个上下文对象。
- 重新初始化取出来的上下文对象。
- 处理外部的 HTTP 请求。
- 处理完毕，将取出的上下文对象返回给对象池。

##### 最终练习代码：

```go
package main

import (
	"github.com/gin-gonic/gin"
)

func main() {
	r := gin.Default()
	r.GET("/ping", func(c *gin.Context){
		c.JSON(200, gin.H {"message": "pong"})
	})
	r.GET("/ping2", func(c *gin.Context){
		c.JSON(200, gin.H {"message": "pong2"})
	})
	r.Run("0.0.0.0:8000")
}
```

