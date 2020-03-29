[TOC]

## 概览

先看一下线程池的类图

![image](https://upload-images.jianshu.io/upload_images/7017140-56bd4c28001d43d8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## Executor 类

**Executor**是一个顶层接口，在它里面只声明了一个方法execute(Runnable)，返回值为void，参数为Runnable类型，从字面意思可以理解，就是用来执行传进去的任务的

## ExecutorService 类

**ExecutorService**接口继承了Executor接口，并声明了一些方法：submit、invokeAll、invokeAny以及shutDown等

## AbstractExecutorService 类

抽象类**AbstractExecutorService**实现了ExecutorService接口，基本实现了ExecutorService中声明的所有方法；

## ThreadPoolExecutor 类

**ThreadPoolExecutor**继承了类AbstractExecutorService。在ThreadPoolExecutor类中有几个非常重要的方法如下。

### 构造方法

```java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler) {
    if (corePoolSize < 0 ||
        maximumPoolSize <= 0 ||
        maximumPoolSize < corePoolSize ||
        keepAliveTime < 0)
        throw new IllegalArgumentException();
    if (workQueue == null || threadFactory == null || handler == null)
        throw new NullPointerException();
    this.acc = System.getSecurityManager() == null ?
            null :
            AccessController.getContext();
    this.corePoolSize = corePoolSize;
    this.maximumPoolSize = maximumPoolSize;
    this.workQueue = workQueue;
    this.keepAliveTime = unit.toNanos(keepAliveTime);
    this.threadFactory = threadFactory;
    this.handler = handler;
}
```

- corePoolSize：核心池的大小

- maximumPoolSize：线程池最大线程数

- keepAliveTime：表示线程没有任务执行时最多保持多久时间会终止

- unit：参数keepAliveTime的时间单位

- workQueue：一个阻塞队列，用来存储等待执行的任务，一般来说，这里的阻塞队列有以下几种选择：

- - ArrayBlockingQueue：基于数组的先进先出队列，此队列创建时必须指定大小
  - LinkedBlockingQueue：基于链表的先进先出队列，如果创建时没有指定此队列大小，则默认为Integer.MAX_VALUE
  - SynchronousQueue：没有容量，是无缓冲等待队列，是一个不存储元素的阻塞队列，它不会保存提交的任务，而是将直接新建一个线程来执行新来的任务

- threadFactory：线程工厂，主要用来创建线程

- handler：表示当拒绝处理任务时的策略，有以下四种取值：

- - ThreadPoolExecutor.**AbortPolicy**:丢弃任务并抛出RejectedExecutionException异常**(默认策略)**
  - ThreadPoolExecutor.**DiscardPolicy**：也是丢弃任务，但是不抛出异常
  - ThreadPoolExecutor.**DiscardOldestPolicy**：丢弃队列最前面的任务，然后重新尝试执行任务
  - ThreadPoolExecutor.**CallerRunsPolicy**：由调用线程处理该任务

### execute()

execute()方法实际上是Executor中声明的方法，在ThreadPoolExecutor进行了具体的实现，这个方法是ThreadPoolExecutor的核心方法，通过这个方法可以向线程池提交一个任务，交由线程池去执行。

![image](https://upload-images.jianshu.io/upload_images/7017140-3d15e753fc2bd6b0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### submit()

submit()方法是在ExecutorService中声明的方法，在AbstractExecutorService就已经有了具体的实现，在ThreadPoolExecutor中并没有对其进行重写，这个方法也是用来向线程池提交任务的，但是它和execute()方法不同，它能够返回任务执行的结果，去看submit()方法的实现，会发现它实际上还是调用的execute()方法，只不过它利用了Future来获取任务执行结果

### shutdown()和shutdownNow()

shutdown()和shutdownNow()是用来关闭线程池的。

## Executors类

在 Java doc 中，并不提倡我们直接使用 ThreadPoolExecutor，而是使用 Executors 类中提供的几个静态方法来创建线程池

```java
Executors.newCachedThreadPool();       //创建一个缓冲池，缓冲池容量大小为Integer.MAX_VALUE
Executors.newSingleThreadExecutor();   //创建容量为1的缓冲池
Executors.newFixedThreadPool(int);     //创建固定容量大小的缓冲池
```

下面是这三个静态方法的具体实现

```java
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
}

public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}

public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}
```

从它们的具体实现来看，它们实际上也是调用了ThreadPoolExecutor，只不过参数都已配置好了。

**newFixedThreadPool**创建的线程池corePoolSize和maximumPoolSize值是相等的，它使用的LinkedBlockingQueue；

**newSingleThreadExecutor**将corePoolSize和maximumPoolSize都设置为1，也使用的LinkedBlockingQueue；

**newCachedThreadPool**将corePoolSize设置为0，将maximumPoolSize设置为Integer.MAX_VALUE，使用的SynchronousQueue，也就是说来了任务就创建线程运行，当线程空闲超过60秒，就销毁线程。

实际中，如果Executors提供的三个静态方法能满足要求，就尽量使用它提供的三个方法，因为自己去手动配置ThreadPoolExecutor的参数有点麻烦，要根据实际任务的类型和数量来进行配置。

另外，如果ThreadPoolExecutor达不到要求，可以自己继承ThreadPoolExecutor类进行重写。

## 线程池的状态转换过程

当创建线程池后，初始时，线程池处于`RUNNING`状态；

如果调用了shutdown()方法，则线程池处于`SHUTDOWN`状态，此时不会立即终止线程池，而是要等所有任务缓存队列中的任务都执行完后才终止，但再也不会接受新的任务

如果调用了shutdownNow()方法，则线程池处于`STOP`状态，此时立即终止线程池，并尝试打断正在执行的任务，并且清空任务缓存队列，返回尚未执行的任务

当线程池处于SHUTDOWN或STOP状态，并且所有工作线程已经销毁，任务缓存队列已经清空或执行结束后，线程池被设置为`TERMINATED`状态。

![image](https://upload-images.jianshu.io/upload_images/7017140-c4776d1847062add.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 线程池的监控

通过线程池提供的参数进行监控。线程池里有一些属性在监控线程池的时候可以使用

- getTaskCount()：线程池已经执行的和未执行的任务总数；
- getCompletedTaskCount()：线程池已完成的任务数量，该值小于等于taskCount；
- getLargestPoolSize()：线程池曾经创建过的最大线程数量。通过这个数据可以知道线程池是否满过，也就是达到了maximumPoolSize；
- getPoolSize()：线程池当前的线程数量；
- getActiveCount()：当前线程池中正在执行任务的线程数量。

通过这些方法，可以对线程池进行监控，在ThreadPoolExecutor类中提供了几个空方法，如beforeExecute方法，afterExecute方法和terminated方法，**可以扩展这些方法在执行前或执行后增加一些新的操作，例如统计线程池的执行任务的时间等，可以继承自ThreadPoolExecutor来进行扩展。**

## 如何合理配置线程池的大小

线程池究竟设置多大要看你的线程池执行的什么任务了，CPU密集型、IO密集型、混合型，任务类型不同，设置的方式也不一样

任务一般分为：CPU密集型、IO密集型、混合型，对于不同类型的任务需要分配不同大小的线程池

### **1、CPU密集型**

尽量使用较小的线程池，一般Cpu核心数+1

因为CPU密集型任务CPU的使用率很高，若开过多的线程，只能增加线程上下文的切换次数，带来额外的开销

### **2、IO密集型**

方法一：可以使用较大的线程池，一般CPU核心数 * 2

IO密集型CPU使用率不高，可以让CPU等待IO的时候处理别的任务，充分利用cpu时间


方法二：线程等待时间所占比例越高，需要越多线程。线程CPU时间所占比例越高，需要越少线程。

下面举个例子：

比如平均每个线程CPU运行时间为0.5s，而线程等待时间（非CPU运行时间，比如IO）为1.5s，CPU核心数为8，那么根据上面这个公式估算得到：((0.5+1.5)/0.5)*8=32。这个公式进一步转化为：

最佳线程数目 = （线程等待时间与线程CPU时间之比 + 1）* CPU数目

### **3、混合型**

可以将任务分为CPU密集型和IO密集型，然后分别使用不同的线程池去处理，按情况而定