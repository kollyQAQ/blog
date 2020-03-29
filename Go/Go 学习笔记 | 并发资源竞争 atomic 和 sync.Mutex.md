[TOC]

有并发，就有资源竞争，如果两个或者多个 goroutine 在没有相互同步的情况下，访问某个共享的资源，比如同时对该资源进行读写时，就会处于相互竞争的状态，这就是并发中的资源竞争。

并发本身并不复杂，但是因为有了资源竞争的问题，就使得我们开发出好的并发程序变得复杂起来，因为会引起很多莫名其妙的问题。

```go
package main

import (
        "fmt"
        "runtime"
        "sync"
)

var (
        count int32
        wg    sync.WaitGroup
)

func main() {
        wg.Add(2)
        go incCount()
        go incCount()
        wg.Wait()
        fmt.Println(count)
}

func incCount() {
        defer wg.Done()
        for i := 0; i < 2; i++ {
                value := count
                runtime.Gosched()
                value++
                count = value
        }
}
```

这是一个资源竞争的例子，我们可以多运行几次这个程序，会发现结果可能是 2，也可以是 3，也可能是 4。因为共享资源 count 变量没有任何同步保护，所以两个 goroutine 都会对其进行读写，会导致对已经计算好的结果覆盖，以至于产生错误结果，这里我们演示一种可能，两个 goroutine 我们暂时称之为 g1 和 g2。

1. g1 读取到count为 0
2. 然后 g1 暂停了，切换到 g2 运行，g2 读取到 count 也为 0
3. g2 暂停，切换到 g1，g1 对 count+1，count 变为 1
4. g1 暂停，切换到 g2，g2 刚刚已经获取到值 0，对其 + 1，最后赋值给 count 还是 1
5. 有没有注意到，刚刚 g1 对 count+1 的结果被 g2 给覆盖了，两个 goroutine 都 + 1 还是 1

不再继续演示下去了，到这里结果已经错了，两个 goroutine 相互覆盖结果。我们这里的 **runtime.Gosched()** 是让当前 goroutine 暂停的意思，退回执行队列，让其他等待的 goroutine 运行，目的是让我们演示资源竞争的结果更明显。注意，这里还会牵涉到 CPU 问题，多核会并行，那么资源竞争的效果更明显。

**所以我们对于同一个资源的读写必须是原子化的，也就是说，同一时间只能有一个 goroutine 对共享资源进行读写操作**。

共享资源竞争的问题，非常复杂，并且难以察觉，好在Go为我们提供了一个工具帮助我们检查，这个就是 **go build -race** 命令。我们在当前项目目录下执行这个命令，生成一个可以执行文件，然后再运行这个可执行文件，就可以看到打印出的检测信息。

```go
go build -race
```

多加了一个-race标志，这样生成的可执行程序就自带了检测资源竞争的功能，下面我们运行，也是在终端运行。

```shell
./hello
```

我这里示例生成的可执行文件名是 hello，所以是这么运行的，这时候，我们看终端输出的检测结果。

```shell
➜  hello ./hello       
==================
WARNING: DATA RACE
Read at 0x0000011a5118 by goroutine 7:
  main.incCount()
      /Users/xxx/code/go/src/flysnow.org/hello/main.go:25 +0x76


Previous write at 0x0000011a5118 by goroutine 6:
  main.incCount()
      /Users/xxx/code/go/src/flysnow.org/hello/main.go:28 +0x9a


Goroutine 7 (running) created at:
  main.main()
      /Users/xxx/code/go/src/flysnow.org/hello/main.go:17 +0x77


Goroutine 6 (finished) created at:
  main.main()
      /Users/xxx/code/go/src/flysnow.org/hello/main.go:16 +0x5f
==================
4
Found 1 data race(s)
```

看，找到一个资源竞争，连在那一行代码出了问题，都标示出来了。goroutine 7 在代码 25 行读取共享资源 value := count, 而这时 goroutine 6 正在代码 28 行修改共享资源 count = value, 而这两个 goroutine 都是从 main 函数启动的，在 16、17 行，通过 go 关键字。

既然我们已经知道共享资源竞争的问题，是因为同时有两个或者多个 goroutine 对其进行了读写，那么我们只要保证，同时只有一个 goroutine 读写不就可以了，现在我们就看下传统解决资源竞争的办法–对资源加锁。

Go 语言提供了 atomic 包和 sync 包里的一些函数对共享资源同步枷锁，我们先看下 atomic 包。

```go
package main


import (
        "fmt"
        "runtime"
        "sync"
        "sync/atomic"
)


var (
        count int32
        wg    sync.WaitGroup
)


func main() {
        wg.Add(2)
        go incCount()
        go incCount()
        wg.Wait()
        fmt.Println(count)
}


func incCount() {
        defer wg.Done()
        for i := 0; i < 2; i++ {
                value := atomic.LoadInt32(&count)
                runtime.Gosched()
                value++
                atomic.StoreInt32(&count,value)
        }
}
```

留意这里 atomic.LoadInt32 和 atomic.StoreInt32 两个函数，一个读取 int32 类型变量的值，一个是修改 int32 类型变量的值，这两个都是原子性的操作，Go 已经帮助我们在底层使用加锁机制，保证了共享资源的同步和安全，所以我们可以得到正确的结果，这时候我们再使用资源竞争检测工具 go build -race 检查，也不会提示有问题了。

atomic 包里还有很多原子化的函数可以保证并发下资源同步访问修改的问题，比如函数 atomic.AddInt32 可以直接对一个 int32 类型的变量进行修改，在原值的基础上再增加多少的功能，也是原子性的，这里不再举例，大家自己可以试试。

atomic 虽然可以解决资源竞争问题，但是比较都是比较简单的，支持的数据类型也有限，所以 Go 语言还提供了一个 sync 包，这个 sync 包里提供了一种互斥型的锁，可以让我们自己灵活的控制哪些代码，同时只能有一个 goroutine 访问，被 sync 互斥锁控制的这段代码范围，被称之为临界区，临界区的代码，同一时间，只能又一个 goroutine 访问。刚刚那个例子，我们还可以这么改造。

```go
package main

import (
        "fmt"
        "runtime"
        "sync"
)

var (
        count int32
        wg    sync.WaitGroup
        mutex sync.Mutex
)

func main() {
        wg.Add(2)
        go incCount()
        go incCount()
        wg.Wait()
        fmt.Println(count)
}

func incCount() {
        defer wg.Done()
        for i := 0; i < 2; i++ {
                mutex.Lock()
                value := count
                runtime.Gosched()
                value++
                count = value
                mutex.Unlock()
        }
}
```

实例中，新声明了一个互斥锁 mutex sync.Mutex，这个互斥锁有两个方法，一个是 mutex.Lock(), 一个是 mutex.Unlock(), 这两个之间的区域就是临界区，临界区的代码是安全的。

示例中我们先调用 mutex.Lock() 对有竞争资源的代码加锁，这样当一个 goroutine 进入这个区域的时候，其他 goroutine 就进不来了，只能等待，一直到调用 mutex.Unlock() 释放这个锁为止。

这种方式比较灵活，可以让代码编写者任意定义需要保护的代码范围，也就是临界区。除了原子函数和互斥锁，Go 还为我们提供了更容易在多个 goroutine 同步的功能，就是通道 channel。