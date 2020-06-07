这是一篇很短的文章，诉说着高效的包管理工具 `go mod`

我们上次说过如何让一个项目在 `Goland` 编译器跑起来，但是要自己去下包，要花不少时间找包下包，是不是很麻烦？

`java` 里有一个叫 `maven` 的包管理工具， `go` 也有一个叫 `go mod` 的管理工具，可以管理项目引用的第三方包版本、自动识别项目中用到的包、自动下载和管理包。

### 怎么用？

找到你的项目，直接执行

``` BASH
go mod init main.go
```

执行完会自动识别项目中用到的第三方包，并生成一个 `go.mod` 文件

``` BASH
$ cat go.mod
module collector_go

go 1.14

require (
	github.com/gogo/protobuf v1.3.1 // indirect
	github.com/golang/protobuf v1.4.2
	google.golang.org/protobuf v1.23.0
)
```

然后直接 `build` 、 `run` 就会自动下载包啦~！

```Go
go build -o ./collector_go main.go
```

### 有一个小前提

`golang>=1.12`
添加环境变量 `GO111MODULE` 为 `on` 或者 `auto` ，设置方法

```BASH
go env GO111MODULE=on
```

### 他解决了什么问题？

![](https://coding3min.oss-accelerate.aliyuncs.com/coding3min/2020-05-19-154359.jpg)

**原来的包管理方式**

- 在不使用额外的工具的情况下，`Go` 的依赖包需要手工下载，
- 第三方包没有版本的概念，如果第三方包的作者做了不兼容升级，会让开发者很难受
- 协作开发时，需要统一各个开发成员本地`$GOPATH/src`下的依赖包
- 引用的包引用了已经转移的包，而作者没改的话，需要自己修改引用。
- 第三方包和自己的包的源码都在`src`下，很混乱。对于混合技术栈的项目来说，目录的存放会有一些问题

**新的包管理模式解决了以上问题**

- 自动下载依赖包
- 项目不必放在`$GOPATH/src`内了
- 项目内会生成一个`go.mod`文件，列出包依赖
- 所以来的第三方包会准确的指定版本号
- 对于已经转移的包，可以用 `replace` 申明替换，不需要改代码

### tips

**Q1: 我的包下哪去了？**

A: 依赖的第三方包被下载到了 `$GOPATH/pkg/mod` 路径下。

**Q2: `GO111MODULE` 的三个参数 `auto` 、 `on` 、 `off` 有什么区别？**

A: `auto` 根据是否在 `src` 下自动判定， `on` 只用 `go.mod` ， `off` 只用 `src` 。

**Q3: 依赖包中的地址失效了怎么办？比如 golang. org/x/… 下的包都无法下载怎么办？**

A: 在 `go.mod` 文件里用 `replace` 替换包，例如

```
replace golang.org/x/text => github.com/golang/text latest
```

这样， `go` 会用 `github.com/golang/text` 替代 `golang.org/x/text`

**Q4: 在 `go mod` 模式中，项目自己引用自己中的某些模块怎么办？**

A: `go.mod` 文件里的第一行会申明 `module main` ，把这个 `main` 改为你的项目名，引用的时候就 `import "项目名/模块名"` 即可。

> 根据官方的说法，从 `Go 1.13` 开始，模块管理模式将是 Go 语言开发的**默认模式**。

参考掘金：https://juejin.im/post/5c9c8c4fe51d450bc9547ba1
