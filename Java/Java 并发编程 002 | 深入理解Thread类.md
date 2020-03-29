[TOC]

### 概述

### 线程的状态

Java 中线程中状态可分为五种：New（新建状态），Runnable（就绪状态），Running（运行状态），Blocked（阻塞状态），Dead（死亡状态）

- New：新建状态，当线程创建完成时为新建状态，即new Thread(...)，还没有调用start方法时，线程处于新建状态。

- Runnable：就绪状态，当调用线程的的start方法后，线程进入就绪状态，等待CPU资源。处于就绪状态的线程由Java运行时系统的线程调度程序(*thread scheduler*)来调度。

- Running：运行状态，就绪状态的线程获取到CPU执行权以后进入运行状态，开始执行run方法。

- Blocked：阻塞状态，线程没有执行完，由于某种原因（如，I/O操作等）让出CPU执行权，自身进入阻塞状态。

- Dead：死亡状态，线程执行完成或者执行过程中出现异常，线程就会进入死亡状态。

**这五种状态之间的转换关系如下图所示**

![image](https://upload-images.jianshu.io/upload_images/7017140-147709e629ea3b62.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### Java中是如何实现这几种状态的转换的

#### 通过 wait/notify/notifyAll 方法的使用

这三个方法都是在 Object 对象中定义的，这里不做讲解，具体可看 [深入理解Object类的 wait 和 notify](https://www.jianshu.com/p/493ee5c901a3) 这篇文章

#### 通过 sleep/yield/join 方法的使用

这三个方法都是在 Thread 对象中定义的，本篇详细讲解

## Thread类中常用的方法

### start方法

start()用来启动一个线程，当调用start方法后，系统才会开启一个新的线程来执行用户定义的子任务，在这个过程中，会为相应的线程分配需要的资源

### run方法

run()方法是不需要用户来调用的，当通过start方法启动一个线程之后，当线程获得了CPU执行时间，便进入run方法体去执行具体的任务。注意，继承Thread类必须重写run方法，在run方法中定义具体要执行的任务

### sleep方法

sleep相当于让线程睡眠，交出CPU，让CPU去执行其他的任务。

但是有一点要非常注意，**sleep方法不会释放锁**，也就是说如果当前线程持有对某个对象的锁，则即使调用sleep方法，其他线程也无法访问这个对象。

注意，如果调用了sleep方法，必须捕获InterruptedException异常或者将该异常向上层抛出。当线程睡眠时间满后，不一定会立即得到执行，因为此时可能CPU正在执行其他的任务。所以说调用**sleep方法相当于让线程进入阻塞状态**。

### yield方法

调用yield方法会让当前线程交出CPU权限，让CPU去执行其他的线程。它跟sleep方法类似，**同样不会释放锁**。但是yield**不能控制具体的交出CPU的时间**，另外，yield方法**只能让拥有相同优先级的线程有获取CPU执行时间的机会**。

注意，**调用yield方法并不会让线程进入阻塞状态，而是让线程重回就绪状态**，它只需要等待重新获取CPU执行时间，这一点是和sleep方法不一样的。

### join方法

假如在main线程中，调用thread.join方法，则main方法会等待thread线程执行完毕或者等待一定的时间。如果调用的是无参join方法，则等待thread执行完毕，如果调用的是指定了时间参数的join方法，则等待一定的时间。

**实际上调用join方法是调用了Object的wait方法**，这个可以通过查看源码得知

```java
/**
 * Waits at most {@code millis} milliseconds for this thread to
 * die. A timeout of {@code 0} means to wait forever.
 *
 * <p> This implementation uses a loop of {@code this.wait} calls
 * conditioned on {@code this.isAlive}. As a thread terminates the
 * {@code this.notifyAll} method is invoked. It is recommended that
 * applications not use {@code wait}, {@code notify}, or
 * {@code notifyAll} on {@code Thread} instances.
 *
 * @param  millis
 *         the time to wait in milliseconds
 *
 * @throws  IllegalArgumentException
 *          if the value of {@code millis} is negative
 *
 * @throws  InterruptedException
 *          if any thread has interrupted the current thread. The
 *          <i>interrupted status</i> of the current thread is
 *          cleared when this exception is thrown.
 */
public final synchronized void join(long millis)
throws InterruptedException {
    long base = System.currentTimeMillis();
    long now = 0;
    if (millis < 0) {
        throw new IllegalArgumentException("timeout value is negative");
    }
    if (millis == 0) {
        while (isAlive()) {
            wait(0);
        }
    } else {
        while (isAlive()) {
            long delay = millis - now;
            if (delay <= 0) {
                break;
            }
            wait(delay);
            now = System.currentTimeMillis() - base;
        }
    }
}
```

wait方法会让线程进入阻塞状态，并且会释放线程占有的锁，并交出CPU执行权限。由于wait方法会让线程释放对象锁，所以join方法同样会让线程释放对一个对象持有的锁。

**这里可能有一个疑问，wait 方法调用后为什么没有看到调用 notify 呢？**

通过上面源码的第六行注释（As a thread terminates the {@code this.notifyAll} method is invoked）可以知道，实际上是在此线程上调用了需要加入的线程对象的wait()方法，加入的线程运行完后，会自动调用 notifyAll 方法，被 join 的线程自然能够继续执行了。

### interrupt方法

interrupt，顾名思义，即中断的意思。

单独调用interrupt方法可以使得处于阻塞状态的线程抛出一个异常，也就说，它可以用来中断一个正处于阻塞状态的线程；

直接调用interrupt方法不能中断正在运行中的线程；

如果配合isInterrupted()能够中断正在运行的线程，因为调用interrupt方法相当于将中断标志位置为true，那么可以通过调用isInterrupted()判断中断标志是否被置位来中断线程的执行。比如下面这段代码：

```java
public class Test {
     
    public static void main(String[] args) throws IOException  {
        Test test = new Test();
        MyThread thread = test.new MyThread();
        thread.start();
        try {
            Thread.currentThread().sleep(2000);
        } catch (InterruptedException e) {
             
        }
        thread.interrupt();
    } 
     
    class MyThread extends Thread{
        @Override
        public void run() {
            int i = 0;
            while(!isInterrupted() && i<Integer.MAX_VALUE){
                System.out.println(i+" while循环");
                i++;
            }
        }
    }
}
```

## Thread类的一些属性

Thread类实现了Runnable接口，在Thread类中，有一些比较关键的属性，比如name是表示Thread的名字，可以通过Thread类的构造器中的参数来指定线程名字，priority表示线程的优先级（最大值为10，最小值为1，默认值为5），daemon表示线程是否是守护线程，target表示要执行的任务。

### 关于线程的优先级 priority

- 对于优先级设置的内容可以通过Thread类的几个常量来决定

- - 最高优先级：public final static int MAX_PRIORITY = 10;
  - 中等优先级：public final static int NORM_PRIORITY = 5;
  - 最低优先级：public final static int MIN_PRIORITY = 1;

- 新创建的线程的优先级就是普通优先级 **Thread.NORM_PRIORITY**

- 继承性：**Java 默认的线程优先级是父线程的优先级**，也就是如果此时在线程A中启动一个线程B，那么B和A的优先级将是一样的

- 高优先级的线程比低优先级的线程有更高的几率得到执行，**实际上这和操作系统及虚拟机版本相关**，有可能即使设置了线程的优先级也不会产生任何作用

### 关于守护线程 daemon

- Java中有两种线程：用户线程和守护线程
- 用户线程和守护线程的区别：通过调用isDaemon()方法辨别，返回false的为"用户线程"，否则为"守护线程"
- **典型的守护线程是垃圾回收线程**。只要当前JVM进程中存在任何一个还未结束的非守护线程，守护线程就会一直工作，只有当最后一个非守护线程结束时，守护线程才会随着JVM一起停止工作
- **主线程main为用户线程**

## 总结

### 线程各个状态之间的转化情况

![image](https://upload-images.jianshu.io/upload_images/7017140-f159fde83f5e7eea.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### sleep和yield有什么异同？

- **二者都不会释放锁**
- sleep方法相当于让线程进入阻塞状态，yield方法并不会让线程进入阻塞状态，而是让线程重回就绪状
- yield不能控制具体的交出CPU的时间，另外，yield方法只能让拥有相同优先级的线程有获取CPU执行时间的机会

### sleep和wait有什么异同？

- Thread.sleep()与Object.wait()二者都可以暂停当前线程，释放CPU控制权。
- **Object.wait()在释放CPU同时，释放了对象锁的控制；Thread.sleep()没有对锁释放**

## 参考资料

- https://www.cnblogs.com/dolphin0520/p/3920357.html
- https://www.cnblogs.com/w-wfy/p/6414801.html   
- https://blog.csdn.net/zhuyong7/article/details/80852884（interrupt、interrupted和isInterrupted的区别）  ，有待深入理解