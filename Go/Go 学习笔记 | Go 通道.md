[TOC]

## 什么是通道

通道，他有点像在两个 routine 之间架设的管道，一个 goroutine 可以往这个管道里塞数据，另外一个可以从这个管道里取数据，有点类似于我们说的队列。

声明一个通道很简单，我们使用chan关键字即可，除此之外，还要指定通道中发送和接收数据的类型，这样我们才能知道，要发送什么类型的数据给通道，也知道从这个通道里可以接收到什么类型的数据。

```go
ch := make(chan int)
```

**通道是用于在 goroutine 之间通信的**，它具有发送和接收两个操作，而且这两个操作的运算符都是**<-**

```go
ch <- 2 //发送数值2给这个通道
x := <-ch //从通道里读取值，并把读取的值赋值给x变量
<-ch //从通道里读取值，然后忽略
```

**可以使用内置的 close 函数关闭。**

```go
close(ch)
```

<font color="red"> 如果一个通道被关闭了，我们就不能往这个通道里发送数据了</font>，如果发送的话，会引起 painc 异常。<font color="red"> 但是，我们还可以接收通道里的数据</font>，如果通道里没有数据的话，接收的数据是**对应类型的零值**。

```go
func TestClosedChan(t *testing.T) {
   ch := make(chan int, 1)
   ch <- 1
   close(ch)
   //ch <- 1 // 报错，不能向已关闭的 channel 写数据
   t.Log(<-ch) // output：1，可以向关闭的 channel 读数据
   t.Log(<-ch) // output：0，读取一个已关闭的 channel 时，总是能读取到对应类型的零值
   // 为了和读取非空未关闭 channel 的行为区别，可以使用两个接收值
   x, ok := <-ch // 如果 channel 已关闭，ok 返回值为 false
   t.Log(ok, x)  // output：false 0
}
```

刚刚我们使用make函数初始化的时候，只有一个参数，其实make还可以有第二个参数，用于指定通道的大小。默认没有第二个参数的时候，通道的大小为0，这种通道也被成为**无缓冲通道**。

```go
ch:=make(chan int)
ch:=make(chan int,0) // 和上面的等价
ch:=make(chan int,2) // 创建了一个大小为2的通道，这种称为有缓冲通道
```

## 无缓冲的通道

无缓冲的通道指的是通道的大小为 0，也就是说，**这种类型的通道在接收前没有能力保存任何值**，它要求发送 goroutine 和接收 goroutine 同时准备好，才可以完成发送和接收操作。

从上面无缓冲的通道定义来看，发送 goroutine 和接收 gouroutine 必须是同步的，同时准备后，如果没有同时准备好的话，先执行的操作就会<font color="red">**阻塞等待**</font>，直到另一个相对应的操作准备好为止。**这种无缓冲的通道我们也称之为同步通道。**

<img src="https://kolly-imgstore.oss-cn-shenzhen.aliyuncs.com/img/go-无缓冲通道.png" style="zoom: 33%;" />

```go
func main() {
        ch := make(chan int)


        go func() {
                var sum int = 0
                for i := 0; i < 10; i++ {
                        sum += i
                }
                ch <- sum
        }()
        
        fmt.Println(<-ch)  // 会一直阻塞等待，直到 ch 里面有内容可以读取

}
```

在前面的例子中，我们为了演示 goroutine，防止程序提前终止，都是使用 sync.WaitGroup 进行等待，现在的这个例子就不用了，我们使用同步通道来等待。

在计算 sum 和的 goroutine 没有执行完，把值赋给 ch 通道之前，fmt.Println(<-ch) 会一直等待，所以 main 主 goroutine 就不会终止，只有当计算和的 goroutine 完成后，并且发送到 ch 通道的操作准备好后，同时 <-ch 就会接收计算好的值，然后打印出来。

## 有缓冲的通道

有缓冲通道，其实是一个队列，这个队列的最大容量就是我们使用 make 函数创建通道时，通过第二个参数指定的。

```go
ch := make(chan int, 3)
```

这里创建容量为 3 的，有缓冲的通道。对于有缓冲的通道，向其发送操作就是向队列的尾部插入元素，接收操作则是从队列的头部删除元素，并返回这个刚刚删除的元素。

**当队列满的时候，发送操作会阻塞；当队列空的时候，接受操作会阻塞**。有缓冲的通道，不要求发送和接收操作时同步的，相反可以解耦发送和接收操作。

<img src="https://kolly-imgstore.oss-cn-shenzhen.aliyuncs.com/img/go-缓冲通道.png" style="zoom:33%;" />

想知道通道的容量以及里面有几个元素数据怎么办？其实和 map 一样，使用 cap 和 len 函数就可以了。

```go
cap(ch) // cap函数返回通道的最大容量
len(ch) // len函数返回现在通道里有几个元素
```

下面是 Go 语言圣经里比较有意义的一个例子，例子是想获取服务端的一个数据，不过这个数据在三个镜像站点上都存在，这三个镜像分散在不同的地理位置，而我们的目的又是想最快的获取到数据。

```go
func mirroredQuery() string {
    responses := make(chan string, 3)
    go func() { responses <- request("asia.gopl.io") }()
    go func() { responses <- request("europe.gopl.io") }()
    go func() { responses <- request("americas.gopl.io") }()
    return <- responses // return the quickest response
}
func request(hostname string) (response string) { /* ... */ }
```

这里我们定义了一个容量为 3 的通道 responses，然后同时发起 3 个并发 goroutine 向这三个镜像获取数据，获取到的数据发送到通道 responses 中，最后我们使用 `return <- responses` 返回获取到的第一个数据，也就是最快返回的那个镜像的数据。

## 单向通道

有时候，我们有一些特殊场景，比如限制一个通道只可以接收，但是不能发送；有时候限制一个通道只能发送，但是不能接收，这种通道我们称为单向通道。

定义单向通道也很简单，只需要在定义的时候，带上 `<-` 即可。

```go
var send chan<- int //只能发送
var receive <-chan int //只能接收
```

注意 `<-` 操作符的位置，在后面是只能发送，对应发送操作；在前面是只能接收，对应接收操作。

单向通道应用于函数或者方法的参数比较多，比如 Context 接口的定义

```
type Context interface {
        Deadline() (deadline time.Time, ok bool)
        Done() <-chan struct{} // 返回一个只读的chan
        Err() error
        Value(key interface{}) interface{}
}
```

## 一个例子

新手在刚学习 channel 刚开始的时候可能会写出下面这样的代码

```go
func TestChan(t *testing.T) {
   ch := make(chan int)
   ch <- 1
   x := <-ch
   t.Log(x)
}
```

结果执行的时候却报错：fatal error: all goroutines are asleep - deadlock!

一脸懵逼 ~

**为什么呢？**

首先我们这里通过 make(chan int)，开辟的通道是一种无缓冲通道，也叫做同步通道，所以当对这个缓冲通道写的时候，会一直阻塞等到某个协程对这个缓冲通道读（这个与典型的生产者消费者有点不一样，当队列中 “内容” 已经满了，生产者再生往里放东西才会阻塞，而这里我 c<-1 理解为生产，他却是需要等到某个协程读了再能继续运行）。

main 函数的执行在 go 语言中本身就是一个协程的执行，所以在执行到 c < -1 的时候，执行 main 函数的协程将被阻塞，换句话说 main 函数被阻塞了，此时不能在继续往下执行了，所以 x := <-ch 无法执行到了，就无法读到 ch 中的内容了，所以整个程序阻塞，发生了死锁。

如何解决这个问题呢？大家想一下，我们阻塞的原因是 main 函数不能继续向下执行了，所以 x := <-ch 执行不到，对吧，那我们在 main 函数中另开一个协程，让他对 ch 进行写（对 ch 进行写的过程不在 main 函数对应的协程中做）

```go
func TestChan(t *testing.T) {
   ch := make(chan int)
   go func() {
      ch <- 1
   }()
   x := <-ch
   t.Log(x)
}
```

还有一种写法就是把 ch 定义为缓冲通道，这样 main 函数所在协程对 ch 进行写操作的时候就不会阻塞了，也就是我们上面说的生产者消费者模式

```go
func TestBufferedChan(t *testing.T) {
   ch := make(chan int, 1)
   ch <- 1
   x := <-ch
   t.Log(x)
}
```

通过这个例子，是不是对缓冲通道和非缓冲通道的理解加深了一步呢。

## 总结

golang 中大部分类型都是值类型(只有 slice / channel / map 是引用类型)，读 / 写类型是值类型的 channel 时，**如果元素 size 比较大时，应该使用指针代替，避免频繁的内存拷贝开销**。

channel 可以看成一个 FIFO 队列，对 FIFO 队列的读写都是原子的操作，不需要加锁。对 channel 的操作行为结果总结如下：

| 操作     | nil channel | closed channel   | not-closed non-nil channel |
| -------- | ----------- | ---------------- | -------------------------- |
| close    | panic       | panic            | 成功 close                 |
| 写 ch <- | 一直阻塞    | panic            | 阻塞或成功写入数据         |
| 读 <- ch | 一直阻塞    | 读取对应类型零值 | 阻塞或成功读取数据         |

## 参考资料

