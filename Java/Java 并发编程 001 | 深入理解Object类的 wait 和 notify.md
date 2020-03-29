[TOC]

## 概述

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

这三个方法都是在 Object 对象中定义的，本篇详细讲解

#### 通过 sleep/yield/join 方法的使用

这三个方法都是在 Thread 对象中定义的，这里不做讲解，具体可看 [深入理解Thread类](https://www.jianshu.com/p/79b03ff6dc8c) 这篇文章

## 各个方法解析

以下是 Object 类的源码

```java
public class Object {
		public final native void notify();
  
  	public final native void notifyAll();
  
  	public final native void wait(long timeout) throws InterruptedException;
  
  	public final void wait() throws InterruptedException {
        wait(0);
    }
}
```

wait、notify和notifyAll方法是Object类的final native方法。所以这些方法不能被子类重写，Object类是所有类的超类，因此在程序中有以下三种形式调用wait等方法。

```java
wait();//方式1：
this.wait();//方式2：
super.wait();//方式3
```

### void wait()

导致线程进入等待状态，直到它被其他线程通过notify()或者notifyAll唤醒。该方法只能在**同步方法**或**同步块**中调用。如果当前线程不是锁的持有者，该方法抛出一个`IllegalMonitorStateException`异常。

### void wait(long millis) 和 void wait(long millis, int nanos)

导致线程进入等待状态直到它被通知或者经过指定的时间。这些方法只能在**同步方法**或**同步块**中调用。如果当前线程不是锁的持有者，该方法抛出一个`IllegalMonitorStateException`异常。

### void notifyAll()

解除**所有**那些在该对象上调用wait方法的线程的阻塞状态。该方法只能在**同步方法**或**同步块**内部调用。如果当前线程不是锁的持有者，该方法抛出一个`IllegalMonitorStateException`异常。

### void notify()

**随机选择一个**在该对象上调用wait方法的线程，解除其阻塞状态。该方法只能在**同步方法**或**同步块**内部调用。如果当前线程不是锁的持有者，该方法抛出一个`IllegalMonitorStateException`异常。

Object.wait()和Object.notify()和Object.notifyall()必须写在synchronized方法内部或者synchronized块内部，这是因为：**这几个方法要求当前正在运行object.wait()方法的线程拥有object的对象锁。**即使你确实知道当前上下文线程确实拥有了对象锁，也不能将object.wait()这样的语句写在当前上下文中。

从这三个方法的文字描述可以知道以下几点信息：

　　1）wait()、notify()和notifyAll()方法是本地方法，并且为final方法，无法被重写。

　　2）调用某个对象的wait()方法能让当前线程阻塞，并且当前线程必须拥有此对象的monitor（即锁）

　　3）调用某个对象的notify()方法能够唤醒一个正在等待这个对象的monitor的线程，如果有多个线程都在等待这个对象的monitor，则只能唤醒其中一个线程；

　　4）调用notifyAll()方法能够唤醒所有正在等待这个对象的monitor的线程；

　　上面已经提到，如果调用某个对象的wait()方法，当前线程必须拥有这个对象的monitor（即锁），因此调用wait()方法必须在同步块或者同步方法中进行（synchronized块或者synchronized方法）。

　　调用某个对象的wait()方法，相当于让当前线程**交出此对象的monitor，然后进入等待状态，等待后续再次获得此对象的锁**（Thread类中的sleep方法**使当前线程暂停执行一段时间，从而让其他线程有机会继续执行，但它并不释放对象锁**）；

　　notify()方法能够唤醒一个正在等待该对象的monitor的线程，当有多个线程都在等待该对象的monitor的话，则只能唤醒其中一个线程，具体唤醒哪个线程则不得而知。

　　同样地，调用某个对象的notify()方法，当前线程也必须拥有这个对象的monitor，因此调用notify()方法必须在同步块或者同步方法中进行（synchronized块或者synchronized方法）。

　　nofityAll()方法能够唤醒所有正在等待该对象的monitor的线程，这一点与notify()方法是不同的。

　　这里要注意一点：notify()和notifyAll()方法只是唤醒等待该对象的monitor的线程，并不决定哪个线程能够获取到monitor。

　　举个简单的例子：假如有三个线程Thread1、Thread2和Thread3都在等待对象objectA的monitor，此时Thread4拥有对象objectA的monitor，当在Thread4中调用objectA.notify()方法之后，Thread1、Thread2和Thread3只有一个能被唤醒。注意，被唤醒不等于立刻就获取了objectA的monitor。假若在Thread4中调用objectA.notifyAll()方法，则Thread1、Thread2和Thread3三个线程都会被唤醒，至于哪个线程接下来能够获取到objectA的monitor就具体依赖于操作系统的调度了。

　　上面尤其要注意一点，一个线程被唤醒不代表立即获取了对象的monitor，只有等调用完notify()或者notifyAll()并退出synchronized块，释放对象锁后，其余线程才可获得锁执行。

## 为什么wait()和notify()属于Object类

wait()和notify()是Java给我们提供线程之间通信的API，既然是线程的东西，那什么是在Object类上定义，而不是在Thread类上定义呢？

其实这个问题很简单，由于每个对象都拥有monitor（即锁），所以让当前线程等待某个对象的锁，当然应该通过这个对象来操作了。而不是用当前线程来操作，因为当前线程可能会等待多个线程的锁，如果通过线程来操作，就非常复杂了。

## Java 线程wait()之后一定要notify()才能唤醒吗？

**不一定！**

线程正常结束后，会使以这个线程对象运行的wait()等待，退出等待状态！而如果在运行wait()之前，线程已经结束了，则这个wait就没有程序唤醒了

实际上，Thread源码里面的join()方法也是使用这种机制：

> As a thread terminates the {@code this.notifyAll} method is invoked.

源码里的join()方法，实际上就是运行的 wait(). 需要运行join的线程运行join方法，实际上是在此线程上调用了需要加入的线程对象的wait()方法，加入的线程运行完后，会自动调用 notifyAll 方法，被 join 的线程自然能够继续执行了。

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

**线程对象的wait()方法运行后，可以不用其notify()方法退出，会在线程结束后，自动退出。**



## Condition

Condition是在java 1.5中才出现的，它用来替代传统的Object的wait()、notify()实现线程间的协作，相比使用Object的wait()、notify()，使用Condition1的await()、signal()这种方式实现线程间协作更加安全和高效。因此通常来说比较推荐使用Condition，在阻塞队列那一篇博文中就讲述到了，阻塞队列实际上是使用了Condition来模拟线程间协作。

- Condition是个接口，基本的方法就是await()和signal()方法；
- Condition依赖于Lock接口，生成一个Condition的基本代码是lock.newCondition() 
- 调用Condition的await()和signal()方法，都必须在lock保护之内，就是说必须在lock.lock()和lock.unlock之间才可以使用

Conditon中的await()对应Object的wait()；

Condition中的signal()对应Object的notify()；

Condition中的signalAll()对应Object的notifyAll()。



## 生产者-消费者模型的实现

### 1.使用Object的wait()和notify()实现

```java
public class Test {
    private int queueSize = 10;
    private PriorityQueue<Integer> queue = new PriorityQueue<Integer>(queueSize);
      
    public static void main(String[] args)  {
        Test test = new Test();
        Producer producer = test.new Producer();
        Consumer consumer = test.new Consumer();
          
        producer.start();
        consumer.start();
    }
      
    class Consumer extends Thread{
          
        @Override
        public void run() {
            consume();
        }
          
        private void consume() {
            while(true){
                synchronized (queue) {
                    while(queue.size() == 0){
                        try {
                            System.out.println("队列空，等待数据");
                            queue.wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                            queue.notify();
                        }
                    }
                    queue.poll();          //每次移走队首元素
                    queue.notify();
                    System.out.println("从队列取走一个元素，队列剩余"+queue.size()+"个元素");
                }
            }
        }
    }
      
    class Producer extends Thread{
          
        @Override
        public void run() {
            produce();
        }
          
        private void produce() {
            while(true){
                synchronized (queue) {
                    while(queue.size() == queueSize){
                        try {
                            System.out.println("队列满，等待有空余空间");
                            queue.wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                            queue.notify();
                        }
                    }
                    queue.offer(1);        //每次插入一个元素
                    queue.notify();
                    System.out.println("向队列取中插入一个元素，队列剩余空间："+(queueSize-queue.size()));
                }
            }
        }
    }
}
```

### 2.使用Condition实现

```java
public class Test {
    private int queueSize = 10;
    private PriorityQueue<Integer> queue = new PriorityQueue<Integer>(queueSize);
    private Lock lock = new ReentrantLock();
    private Condition notFull = lock.newCondition();
    private Condition notEmpty = lock.newCondition();
     
    public static void main(String[] args)  {
        Test test = new Test();
        Producer producer = test.new Producer();
        Consumer consumer = test.new Consumer();
          
        producer.start();
        consumer.start();
    }
      
    class Consumer extends Thread{
          
        @Override
        public void run() {
            consume();
        }
          
        private void consume() {
            while(true){
                lock.lock();
                try {
                    while(queue.size() == 0){
                        try {
                            System.out.println("队列空，等待数据");
                            notEmpty.await();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                    queue.poll();                //每次移走队首元素
                    notFull.signal();
                    System.out.println("从队列取走一个元素，队列剩余"+queue.size()+"个元素");
                } finally{
                    lock.unlock();
                }
            }
        }
    }
      
    class Producer extends Thread{
          
        @Override
        public void run() {
            produce();
        }
          
        private void produce() {
            while(true){
                lock.lock();
                try {
                    while(queue.size() == queueSize){
                        try {
                            System.out.println("队列满，等待有空余空间");
                            notFull.await();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                    queue.offer(1);        //每次插入一个元素
                    notEmpty.signal();
                    System.out.println("向队列取中插入一个元素，队列剩余空间："+(queueSize-queue.size()));
                } finally{
                    lock.unlock();
                }
            }
        }
    }
}
```

## 参考资料

- https://www.cnblogs.com/dolphin0520/p/3920385.html