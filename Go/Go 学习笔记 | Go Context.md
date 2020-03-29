控制并发有两种经典的方式，一种是 WaitGroup，另外一种就是 Context，今天我就谈谈 Context。

## 什么是WaitGroup

WaitGroup 以前我们在并发的时候介绍过，它是一种控制并发的方式，它的这种方式是控制多个 goroutine 同时完成。

```go
func main() {
        var wg sync.WaitGroup


        wg.Add(2)
        go func() {
                time.Sleep(2*time.Second)
                fmt.Println("1号完成")
                wg.Done()
        }()
        go func() {
                time.Sleep(2*time.Second)
                fmt.Println("2号完成")
                wg.Done()
        }()
        wg.Wait()
        fmt.Println("好了，大家都干完了，放工")
}
```

一个很简单的例子，一定要例子中的 2 个 goroutine 同时做完，才算是完成，先做好的就要等着其他未完成的，所有的 goroutine 要都全部完成才可以。

这是一种控制并发的方式，这种尤其适用于，好多个 goroutine 协同做一件事情的时候，因为每个 goroutine 做的都是这件事情的一部分，只有全部的 goroutine 都完成，这件事情才算是完成，这是等待的方式。

在实际的业务种，我们可能会有这么一种场景：**需要我们主动的通知某一个 goroutine 结束。比如我们开启一个后台 goroutine 一直做事情，比如监控，现在不需要了，就需要通知这个监控 goroutine 结束，不然它会一直跑，就泄漏了。**

## chan通知

我们都知道一个 goroutine 启动后，我们是无法控制他的，大部分情况是等待它自己结束，那么如果这个 goroutine 是一个不会自己结束的后台 goroutine 呢？比如监控等，会一直运行的。

这种情况化，一直傻瓜式的办法是全局变量，其他地方通过修改这个变量完成结束通知，然后后台 goroutine 不停的检查这个变量，如果发现被通知关闭了，就自我结束。

这种方式也可以，但是首先我们要保证这个变量在多线程下的安全，基于此，有一种更好的方式：chan + select 。

```go
func main() {
        stop := make(chan bool)

        go func() {
                for {
                        select {
                        case <-stop:
                                fmt.Println("监控退出，停止了...")
                                return
                        default:
                                fmt.Println("goroutine监控中...")
                                time.Sleep(2 * time.Second)
                        }
                }
        }()

        time.Sleep(10 * time.Second)
        fmt.Println("可以了，通知监控停止")
        stop<- true
        //为了检测监控过是否停止，如果没有监控输出，就表示停止了
        time.Sleep(5 * time.Second)
}
```

这种 chan+select 的方式，是比较优雅的结束一个 goroutine 的方式，不过这种方式也有局限性，**如果有很多 goroutine 都需要控制结束怎么办呢？如果这些 goroutine 又衍生了其他更多的 goroutine 怎么办呢？如果一层层的无穷尽的 goroutine 呢？**这就非常复杂了，即使我们定义很多 chan 也很难解决这个问题，因为 goroutine 的关系链就导致了这种场景非常复杂。

## 初识Context

上面说的这种场景是存在的，比如一个网络请求 Request，每个 Request 都需要开启一个 goroutine 做一些事情，这些 goroutine 又可能会开启其他的 goroutine。所以我们需要一种可以跟踪 goroutine 的方案，才可以达到控制他们的目的，这就是 Go 语言为我们提供的 Context，称之为上下文非常贴切，它就是 goroutine 的上下文。

下面我们就使用 Go Context 重写上面的示例。

```go
func main() {
        ctx, cancel := context.WithCancel(context.Background())
        go func(ctx context.Context) {
                for {
                        select {
                        case <-ctx.Done():
                                fmt.Println("监控退出，停止了...")
                                return
                        default:
                                fmt.Println("goroutine监控中...")
                                time.Sleep(2 * time.Second)
                        }
                }
        }(ctx)

        time.Sleep(10 * time.Second)
        fmt.Println("可以了，通知监控停止")
        cancel()
        //为了检测监控过是否停止，如果没有监控输出，就表示停止了
        time.Sleep(5 * time.Second)
}
```

重写比较简单，就是把原来的 chan stop 换成 Context，使用 Context 跟踪 goroutine，以便进行控制，比如结束等。

context.Background() 返回一个空的 Context，这个空的 Context 一般用于整个 Context 树的根节点。然后我们使用 context.WithCancel(parent)函数，创建一个可取消的子 Context，然后当作参数传给 goroutine 使用，这样就可以使用这个子 Context 跟踪这个 goroutine。

在 goroutine 中，使用 select 调用 <-ctx.Done() 判断是否要结束，如果接受到值的话，就可以返回结束 goroutine 了；如果接收不到，就会继续进行监控。

那么是如何发送结束指令的呢？这就是示例中的 cancel 函数啦，它是我们调用 context.WithCancel(parent)函数生成子 Context 的时候返回的，第二个返回值就是这个取消函数，它是 CancelFunc 类型的。我们调用它就可以发出取消指令，然后我们的监控 goroutine 就会收到信号，就会返回结束。

## Context 控制多个 goroutine

使用 Context 控制一个 goroutine 的例子如上，非常简单，下面我们看看控制多个 goroutine 的例子，其实也比较简单。

```go
func main() {
        ctx, cancel := context.WithCancel(context.Background())
        go watch(ctx,"【监控1】")
        go watch(ctx,"【监控2】")
        go watch(ctx,"【监控3】")


        time.Sleep(10 * time.Second)
        fmt.Println("可以了，通知监控停止")
        cancel()
        //为了检测监控过是否停止，如果没有监控输出，就表示停止了
        time.Sleep(5 * time.Second)
}


func watch(ctx context.Context, name string) {
        for {
                select {
                case <-ctx.Done():
                        fmt.Println(name,"监控退出，停止了...")
                        return
                default:
                        fmt.Println(name,"goroutine监控中...")
                        time.Sleep(2 * time.Second)
                }
        }
}
```

示例中启动了 3 个监控 goroutine 进行不断的监控，每一个都使用了 Context 进行跟踪，当我们使用 cancel 函数通知取消时，这 3 个 goroutine 都会被结束。这就是 Context 的控制能力，它就像一个控制器一样，按下开关后，<font color="red">**所有基于这个 Context 或者衍生的子 Context 都会收到通知**</font>，这时就可以进行清理操作了，最终释放 goroutine，这就优雅的解决了 goroutine 启动后不可控的问题。

## 参考资料

https://learnku.com/articles/29877

https://www.flysnow.org/2017/05/12/go-in-action-go-context.html