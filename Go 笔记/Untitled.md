[TOC]

# go mod 使用

`go modules` 是 golang 1.11 新加的特性。现在 1.12 已经发布了，是时候用起来了。Modules 官方定义为：

> 模块是相关 Go 包的集合。modules 是源代码交换和版本控制的单元。 go 命令直接支持使用 modules，包括记录和解析对其他模块的依赖性。modules 替换旧的基于 GOPATH 的方法来指定在给定构建中使用哪些源文件。

## 如何使用 Modules ？

1. 把 golang 升级到 1.11（现在 1.12 已经发布了，建议使用 1.12）
2. 设置 `GO111MODULE`

### GO111MODULE

`GO111MODULE`有三个值：`off`, `on`和 `auto（默认值）`。

- `GO111MODULE=off`，go 命令行将不会支持 module 功能，寻找依赖包的方式将会沿用旧版本那种通过 vendor 目录或者 GOPATH 模式来查找。

- `GO111MODULE=on`，go 命令行会使用 modules，而一点也不会去 GOPATH 目录下查找。

- ```
  GO111MODULE=auto
  ```

  ，默认值，go 命令行将会根据当前目录来决定是否启用 module 功能。这种情况下可以分为两种情形：

  - 当前目录在 GOPATH/src 之外且该目录包含 go.mod 文件
  - 当前文件在包含 go.mod 文件的目录下面。

> 当 modules 功能启用时，依赖包的存放位置变更为 `$GOPATH/pkg`，允许同一个 package 多个版本并存，且多个项目可以共享缓存的 module。

### go mod

golang 提供了 `go mod`命令来管理包。

go mod 有以下命令：

| 命令     | 说明                                                         |
| -------- | ------------------------------------------------------------ |
| download | download modules to local cache (下载依赖包)                 |
| edit     | edit go.mod from tools or scripts（编辑 go.mod               |
| graph    | print module requirement graph (打印模块依赖图)              |
| init     | initialize new module in current directory（在当前目录初始化 mod） |
| tidy     | add missing and remove unused modules (拉取缺少的模块，移除不用的模块) |
| vendor   | make vendored copy of dependencies (将依赖复制到 vendor 下)  |
| verify   | verify dependencies have expected content (验证依赖是否正确） |
| why      | explain why packages or modules are needed (解释为什么需要依赖) |

## 如何在项目中使用

1. 在 `GOPATH 目录之外`新建一个目录，并使用 `go mod init` 初始化生成 `go.mod` 文件

   > go.mod 文件一旦创建后，它的内容将会被 go toolchain 全面掌控。go toolchain 会在各类命令执行时，比如 go get、go build、go mod 等修改和维护 go.mod 文件

2. 添加依赖

   go module 安装 package 的原則是先拉最新的 release tag，若无 tag 则拉最新的 commit，详见 [Modules 官方介绍](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fgolang%2Fgo%2Fwiki%2FModules)。 go 会自动生成一个 go.sum 文件来记录 dependency tree

## 使用 replace 替换无法直接获取的 package

由于某些已知的原因，并不是所有的 package 都能成功下载，比如：`golang.org`下的包。

modules 可以通过在 go.mod 文件中使用 replace 指令替换成 github 上对应的库，比如：

```
replace (
	golang.org/x/crypto v0.0.0-20190313024323-a1f597ede03a => github.com/golang/crypto v0.0.0-20190313024323-a1f597ede03a
)
```

或者

```
replace golang.org/x/crypto v0.0.0-20190313024323-a1f597ede03a => github.com/golang/crypto v0.0.0-20190313024323-a1f597ede03a
```

