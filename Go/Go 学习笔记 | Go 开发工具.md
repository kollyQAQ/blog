[TOC]

## Go 开发工具概览

```shell
~  go
Go is a tool for managing Go source code.

Usage:

    go <command> [arguments]

The commands are:

    bug         start a bug report
    build       compile packages and dependencies
    clean       remove object files and cached files
    doc         show documentation for package or symbol
    env         print Go environment information
    fix         update packages to use new APIs
    fmt         gofmt (reformat) package sources
    generate    generate Go files by processing source
    get         download and install packages and dependencies
    install     compile and install packages and dependencies
    list        list packages or modules
    mod         module maintenance
    run         compile and run Go program
    test        test packages
    tool        run specified go tool
    version     print Go version
    vet         report likely mistakes in packages

Use "go help <command>" for more information about a command.

Additional help topics:

    buildmode   build modes
    c           calling between Go and C
    cache       build and test caching
    environment environment variables
    filetype    file types
    go.mod      the go.mod file
    gopath      GOPATH environment variable
    gopath-get  legacy GOPATH go get
    goproxy     module proxy protocol
    importpath  import path syntax
    modules     modules, module versions, and more
    module-get  module-aware go get
    packages    package lists and patterns
    testflag    testing flags
    testfunc    testing functions

Use "go help <topic>" for more information about that topic.
```

## **go get**

go get 命令，可以从网上下载更新指定的包以及依赖的包，并对它们进行编译和安装。

```shell
go get github.com/spf13/cobra
```

以上示例，我们就可以从 github 上直接下载这个 go 库到我们 GOPATH 工作空间中，以供我们使用。下载的是整个源代码工程，并且会根据它们编译和安装，和执行 go install 类似。

go get 支持大多数版本控制系统 (VCS)，比如我们常用的 git，通过它和包依赖管理结合，我们可以在代码中直接导入网络上的包以供我们使用。

如果我们需要更新网络上的一个 go 工程，加 - u 标记即可。

## go doc

在 Go 语言工具链中自带了多种性能分析工具，供开发者分析问题。

- CPU 使用分析
- 内部使用分析
- 查看协程栈
- 查看 GC 日志
- Trace 分析工具

## 参考资料

https://www.flysnow.org/2017/03/08/go-in-action-go-tools.html

https://zhuanlan.zhihu.com/p/26695984