[TOC]

## Java 基础
### 数据类型

八种基本类型及其占用字节数：**boolean 1、byte 1、short 2、char 2、int 4、float 4、long 8、double 8**

引用类型的内存占用和系统位数以及启动参数 UseCompressedOops 有关，**32 位系统 4 个字节，64 位系统 8 个字节**（开启 UseCompressedOops 时，占用 4 字节）

1K = 1024字节，1M = 100 万字节，1G = 10亿字节

UTF8字符集下：1 个英文字符1 个字节，1 个中文 3-4 字节，**1k =** 1024 字节 = **1024 个英文字符** = 341 中文

1 个整数 4 个字节，**1 亿个整数** = 4 亿个字节 **= 400 M 存储空间**

### Java 对象内存结构

![](https://kolly-imgstore.oss-cn-shenzhen.aliyuncs.com/img/Java对象内存结构.png)

### JDK 代理和 cglib 代理

1、原理

- jdk静态代理实现比较简单，一般是直接代理对象直接包装了被代理对象。

- jdk动态代理是接口代理，被代理类A需要实现业务接口，业务代理类B需要实现InvocationHandler接口。

- jdk动态代理会根据被代理对象生成一个继承了Proxy类，并实现了该业务接口的jdk代理类，该类的字节码会被传进去的ClassLoader加载，创建了jdk代理对象实例，

- jdk代理对象实例在创建时，业务代理对象实例会被赋值给Proxy类，jdk代理对象实例也就有了业务代理对象实例，同时jdk代理对象实例通过反射根据被代理类的业务方法创建了相应的Method对象m（可能有多个）。当jdk代理对象实例调用业务方法，如proxy.addUser();这个会先把对应的m对象作为参数传给invoke()方法（就是invoke方法的第二个参数），调用了jdk代理对象实例的invoke()回调方法，在invoke方法里面再通过反射来调用被代理对象的因为方法，即result = method.invoke(target, args);。

- cglib动态代理是继承代理，通过ASM字节码框架修改字节码生成新的子类，重写并增强方法的功能。

2、优缺点

- jdk静态代理类只能为一个被代理类服务，如果需要代理的类比较多，那么会产生过多的代理类。jdk静态代理在编译时产生class文件，运行时无需产生，可直接使用，效率好。

- jdk动态代理必须实现接口，通过反射来动态代理方法，消耗系统性能。但是无需产生过多的代理类，避免了重复代码的产生，系统更加灵活。

- cglib动态代理无需实现接口，通过生成子类字节码来实现，比反射快一点，没有性能问题。但是由于cglib会继承被代理类，需要重写被代理方法，所以被代理类不能是final类，被代理方法不能是final。

因此，cglib的应用更加广泛一点。

## Java 集合

### ArrayList

- 随机访问、尾部插入或删除元素 —— O(1)
- 中间插入或者中间删除元素——O(n)，因为需要搬运元素
- 数组拷贝使用 System.arraycopy
- elementData 定义为 transient 的原因是自己根据 size 序列化真实的元素，而不是根据数组的长度序列化元素，减少了空间占用

### CopyOnWriteArrayList

- ArrayList的线程安全版本，内部也是通过数组实现，每次对数组的修改都完全拷贝一份新的数组来修改，修改完了再替换掉老数组，这样保证了只阻塞写操作，不阻塞读操作，实现读写分离
- private transient volatile Object[] array 真正存储元素的地方，使用transient修饰表示不自动序列化，使用volatile修饰表示一个线程对这个字段的修改另外一个线程立即可见
- CopyOnWriteArrayList采用读写分离的思想，读操作不加锁，写操作加锁，且写操作占用较大内存空间，所以适用于读多写少的场合
- CopyOnWriteArrayList只保证最终一致性，不保证实时一致性
- 为什么CopyOnWriteArrayList没有size属性？因为每次修改都是拷贝一份正好可以存储目标个数元素的数组，所以不需要size属性了，数组的长度就是集合的大小

### HashMap

- 查询和修改的速度都很快，能达到O(1)的平均时间复杂度。它是非线程安全的，且不保证元素存储的顺序
- HashMap的实现采用了（数组 + 链表 + 红黑树）的复杂结构，数组的一个元素又称作桶
- 数组的查询效率为O(1)，链表的查询效率是O(n)，红黑树的查询效率是O(log n)，k为桶中的元素个数，所以当元素数量非常多的时候，转化为红黑树能极大地提高效率
- HashMap的默认初始容量为16（1<<4），默认装载因子为0.75f，容量总是2的n次方；HashMap扩容时每次容量变为原来的两倍；
- 为了使数据在桶中均匀分布减少碰撞，采用取模的算法，也就是 hash % length，根据取模的结果决定放到哪个桶中；计算机中直接求余效率不如位移运算，如果 length 是 2 的 n 次方，那么 hash & (length-1)就等同于hash % length，但是效率更高
- HashMap 扩容**在多线程情况下**出现死循环，导致CPU100%现象，主要原因是多个线程执行扩容操作时可能导致链表形成环，在 get 的时候出现死循环，1.8 版本已经通过将新节点放在链表尾部解决

### TreeMap的实现原理

TreeMap 是线程不安全的，它的iterator 方法返回的**迭代器是fail-fast**的

TreeMap 是一个**有序的key-value集合**，它是通过红黑树实现的 

TreeMap的基本操作 containsKey、get、put 和 remove 的时间复杂度是 log(n) 

### ConcurrentHashMap

在JDK1.7的时候，ConcurrentHashMap 对整个桶数组进行了分割分段（Segment），每一把锁只锁容器其中一部分数据，多线程访问容器里不同数据段的数据，就不会存在锁竞争，提高并发访问率。 到了 JDK1.8 的时候已经摒弃了Segment的概念，而是直接用 Node 数组+链表+红黑树的数据结构（同 HashMap）来实现，并发控制使用 synchronized 和 CAS 来操作，整个看起来就像是优化过且线程安全的 HashMap，虽然在 JDK1.8 中还能看到 Segment 的数据结构，但是已经简化了属性，只是为了兼容旧版本。

为什么使用 synchronized 而不是 ReentrantLock？因为 synchronized 已经得到了极大地优化，在特定情况下并不比 ReentrantLock 差

**添加元素流程**（添加元素操作中使用的锁主要有（自旋锁 + CAS + synchronized + 分段锁））

- 如果桶数组未初始化，则初始化（使用 CAS 锁控制只有一个线程初始化桶数组）

- 如果待插入的元素所在的桶为空，则尝试把此元素直接插入到桶的第一个位置；

- 如果正在扩容，则当前线程一起加入到扩容的过程中；

- 如果待插入的元素所在的桶不为空且不在迁移元素，则用 synchronized 锁住这个桶（分段锁）；

- 如果当前桶中元素以链表方式存储，则在链表中寻找该元素或者插入元素；

- 如果当前桶中元素以红黑树方式存储，则在红黑树中寻找该元素或者插入元素；

- 如果元素存在，则返回旧值；

- 如果元素不存在，整个 Map 的元素个数加 1，并检查是否需要扩容；

  - 扩容的时候，**新桶数组大小是旧桶数组的两倍**，迁移元素时会锁住当前桶，也是分段锁的思想
  - 整个扩容过程都是通过 CAS 控制 sizeCtl 这个字段来进行的，这很关键

**获取元素**

- 查询操作是不会加锁的，所以 ConcurrentHashMap 不是强一致性的

**ConcurrentHashMap 中有哪些值得学习的技术呢？**

（1）CAS + 自旋，乐观锁的思想，减少线程上下文切换的时间；

（2）分段锁的思想，减少同一把锁争用带来的低效问题；

（3）CounterCell，分段存储元素个数，减少多线程同时更新一个字段带来的低效；

（4）@sun.misc.Contended（CounterCell 上的注解），避免伪共享；（p.s. 伪共享我们后面也会讲的 ^^）

（5）多线程协同进行扩容；

### 并发集合类有哪些

同步容器：Vector、Stack、HashTable、Collections.synchronizedXxxx

并发容器：ConcurrentHashMap、CopyOnWriteArrayList

阻塞队列：ArrayBlockingQueue、LinkedBlockingQueue



### ArrayList 中 elementData设置成了transient，为什么

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable{
        
    private static final long serialVersionUID = 8683452581122892189L;
    
    /**
		 * 存储元素的数组
		 */
		transient Object[] elementData;
		
		private void writeObject(java.io.ObjectOutputStream s)
        throws java.io.IOException{
        // 具体序列化逻辑，只序列化真实的元素
    }
    
    private void readObject(java.io.ObjectOutputStream s)
        throws java.io.IOException{
        // 具体序列化逻辑，只序列化真实的元素
    }
}

```

elementData 是 ArrayList 真正存放元素的地方，使用 transient 是为了不序列化这个字段，那 ArrayList 是怎么把元素序列化的呢？

一般地，只要实现了 Serializable 接口即可自动序列化，writeObject() 和 readObject() 是为了**自己控制序列化的方式**，这两个方法必须声明为 private，在 java.io.ObjectStreamClass#getPrivateMethod() 方法中通过反射获取到 writeObject() 这个方法。

在 ArrayList 的 writeObject() 方法中先调用了 s.defaultWriteObject() 方法，这个方法是写入非 static 非 transient的 属性，在 ArrayLis t中也就是 size 属性。同样地，在 readObject() 方法中先调用了 s.defaultReadObject() 方法解析出了 size 属性。

**ArrayList 中 elementData 定义为 transient 的原因是自己根据 size 序列化真实的元素，而不是根据数组的长度序列化元素，减少了空间占用。**



### HashMap为什么当链表长度大于阈值（默认为8）时，将链表转化为红黑树

**TreeNodes 占用空间是普通 Nodes 的两倍**，这样就解析了为什么不是一开始就将其转换为 TreeNodes，而是需要一定节点数才转为 TreeNodes，说白了就是 trade-off，空间和时间的权衡

理想情况下随机 hashCode 算法下所有 bin 中节点的分布频率会遵循**泊松分布**，我们可以看到，一个 bin 中链表长度达到 8 个元素的概率为 0.00000006，几乎是不可能事件。所以，之所以选择 8，不是拍拍屁股决定的，而是根据概率统计决定的

### 参考资料

- https://www.cnblogs.com/tong-yuan/p/10638855.html



## Java 并发编程

### 线程

![](https://kolly-imgstore.oss-cn-shenzhen.aliyuncs.com/img/线程各个状态之间的转换.png)

- 调用某个对象的 wait 或者 notify 方法，，当前线程必须拥有这个对象的 monitor（即锁） 。因此**调用 wait 或 notify 方法必须在同步块或者同步方法中进行**（synchronized 块或者 synchronized 方法）
- 调用某个对象的 wait 方法，相当于让当前线程交出此对象的 monitor，然后进入等待状态，等待后续再次获得此对象的锁。但是 Thread 类中的 sleep 方法使当前线程暂停执行一段时间，从而让其他线程有机会继续执行，但它并不释放对象锁（**wait 方法会释放锁，sleep 方法不会释放锁**）

- 线程正常结束后，会使以这个线程对象运行的 wait () 等待，退出等待状态。所以，**对象调用 wait 之后不一定要 notify () 才能唤醒**

- **A 线程调用 B 线程对象的 join 方法，实际上就是运行 B 线程对象的 wait 方法**，B 线程运行完后，会自动调用 this.notifyAll 方法，A 线程自然能够继续执行了

  ```java
  // 主线程
  public class Father extends Thread {
      public void run() {
          Son s = new Son();
          s.start();
          s.join(); //调用了s.wait()，s线程完毕后会自动this.notifyAll，然后主线程这里往下执行
          ...
      }
  }
  // 子线程
  public class Son extends Thread {
      public void run() {
          ...
      }
  }
  ```

- sleep 和 yield 的区别：调用 yield 方法会让当前线程交出 CPU 权限，让 CPU 去执行其他的线程。它跟 sleep 方法类似，**同样不会释放锁**。但是 yield **不能控制具体的交出 CPU 的时间**。同时，**调用 yield 方法并不会让线程进入阻塞状态，而是让线程重回就绪状态**

#### 为什么 wait () 和 notify () 属于 Object 类

wait () 和 notify () 是 Java 给我们提供线程之间通信的 API，既然是线程的东西，那什么是在 Object 类上定义，而不是在 Thread 类上定义呢？

其实这个问题很简单，由于每个对象都拥有 monitor（即锁），所以让当前线程等待某个对象的锁，当然应该通过这个对象来操作了，而不是用当前线程来操作，因为当前线程可能会等待多个对象的锁，如果通过线程来操作，就非常复杂了，比如 Thread.wait(Object object)。

### 线程池

* FixedThreadPool - 线程池大小固定，任务队列无界

* SingleThreadExecutor - 线程池大小固定为1，任务队列无界

* CachedThreadPool - 线程池无限大（MAX INT），等待队列长度为0

  ![](http://upload-images.jianshu.io/upload_images/2184951-18c425ebd3877453.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

  对于需要保证所有提交的任务都要被执行的情况，使用FixedThreadPool

  如果限定只能使用一个线程进行任务处理，使用SingleThreadExecutor

  如果希望提交的任务尽快分配线程执行，使用CachedThreadPool

  如果业务上允许任务执行失败，或者任务执行过程可能出现执行时间过长进而影响其他业务的应用场景，可以通过使用限定线程数量的线程池以及限定长度的队列进行容错处理。

  ```
  private ThreadPoolExecutor executor = null;

  @PostConstruct
  public void initThreadPool() {
  		/**
  		 * 核心线程池大小100
  		 * 最大线程池大小1000
  		 * 待执行队列长度2000
  		 * 空闲线程结束的超时时间3s
  		 * 异常处理：微信发送模版消息
  		 */
  		BlockingQueue<Runnable> queue = new ArrayBlockingQueue<>(2000);
  		executor = new ThreadPoolExecutor(100, 1000, 3, TimeUnit.SECONDS, queue,
  				new RejectedExecutionHandler() {
  					@Override
  					public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
  						teammateMessageBiz.sendWarnTemplateMessage("#FF0000", "统计系统线程池不足");
  					}
  				});
  }

  @RequestMapping(value = {"/trackEvent"})
  public void trackEvent(@RequestParam final String shopId, final @RequestParam String hostName){
  		//调用线程池线程
    		executor.execute(new Runnable() {
  			@Override
  			public void run() {
  				//记录统计到redis
  				recordTrackStatToRedis(shopId, hostName);
  			}
  		});
  }
  ```

#### 如何合理配置线程池的大小

CPU 密集型：尽量使用较小的线程池，一般 Cpu 核心数 + 1

IO 密集型：可以使用较大的线程池，一般 CPU 核心数 * 2；线程等待时间所占比例越高，需要越多线程。线程 CPU 时间所占比例越高，需要越少线程

最佳线程数目 = （线程等待时间与线程 CPU 时间之比 + 1）* CPU 数目

###  Java 同步器（原理： AQS 的底层原理）

1. CountDownLatch 可以实现计数等待，主要用于某个线程等待其他几个线程
2. CyclicBarrier 实现循环栅栏，主要用于多个线程同时等待其他线程
3. Semaphore 信号量，主要强调只有某些个数量的线程能拿到资源执行

### ThreadLocal

![](https://kolly-imgstore.oss-cn-shenzhen.aliyuncs.com/img/ThreadLocal%20%E5%9B%BE%E8%A7%A3.png)

#### 内存泄漏具体是怎么产生的

**线程池创建的 ThreadLocal** 要在 finally 中手动 remove，不然会有内存泄漏的问题。

主要分两种场景：主线程仍然对 ThreadLocal 有引用和主线程不存在对 ThreadLocal 的引用。第一种场景因为主线程仍然在运行，所以还是有对 ThreadLocal 的引用，那么 ThreadLocal 变量的引用和 value 是不会被回收的。第二种场景虽然主线程不存在对 ThreadLocal 的引用，且该引用是弱引用，所以会在 gc 的时候被回收，但是对用的 value 不是弱引用，不会被内存回收，仍然会造成内存泄漏

### Java 的锁有哪些？

#### 1. volatile（非锁）

Java 中的关键字，**保证内存可见性**，当多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程能够立即看得到修改的值。（这里牵涉到 Java 内存模型的知识，Java 内存模型规定了所有的变量都存储在主内存中，每条线程还有自己的工作内存，线程的工作内存中保存了该线程中是用到的变量的主内存副本拷贝，线程对变量的所有操作都必须在工作内存中进行，而不能直接读写主内存，**volatile 的操作都在 Main Memory 中**，而Main Memory 是被所有线程所共享的）（另外，通过synchronized和Lock也能够保证可见性，synchronized和Lock能保证同一时刻只有一个线程获取锁然后执行同步代码，并且在释放锁之前会将对变量的修改刷新到主存当中。因此可以保证可见性）

**禁止指令重排**，比如双重检查实现单例模式中，单例对象使用 volatile 是为了保证，`Object instance = new Object()` 这个操作的三个指令不会发生重排，导致 instance 不为 null 但实际上并没有分配内存。

volatile 只保证可见性，不保证原子性，比如 volatile修改的变量 i，针对i++操作，不保证每次结果都正确，因为i++操作是两步操作，相当于 i = i +1，先读取，再加1，这种情况volatile是无法保证的。

#### 2. synchronized

Java 中的关键字，JVM 级别实现，内部实现为监视器锁，主要是通过对象监视器在**对象头中的字段**来实现的

synchronized 从旧版本到现在已经做了很多优化了，大部分情况下并不比 ReentrantLock 差，而且相比 ReentrantLock 还可以避免忘记释放锁

##### 在运行时会有三种存在方式：偏向锁，轻量级锁，重量级锁。

偏向锁，是指一段同步代码一直被一个线程访问，那么这个线程会自动获取锁，降低获取锁的代价。

轻量级锁，是指当锁是偏向锁时，被另一个线程所访问，偏向锁会升级为轻量级锁，这个线程会通过**自旋**的方式尝试获取锁，不会阻塞，提高性能。

重量级锁，是指当锁是轻量级锁时，当自旋的线程自旋了一定的次数后，还没有获取到锁，就会进入阻塞状态，该锁升级为重量级锁，重量级锁会使其他线程阻塞，性能降低。

##### 自旋锁

自旋锁，是指尝试获取锁的线程不会阻塞，而是循环的方式不断尝试，这样的好处是减少线程的上下文切换带来的开销，提高性能，缺点是循环会消耗CPU。

#### 3. ReentrantLock

可重入锁，是指一个线程获取锁之后再尝试获取锁时会自动获取锁，可重入锁的优点是避免死锁，synchronized也是可重入锁

相比 synchronized 优点：可以手动取消锁等待、可以设置为公平锁、可以限时、支持以非阻塞方式获取锁

#### 4. CAS

CAS，Compare And Swap，它是一种乐观锁，认为对于同一个数据的并发操作不一定会发生修改，在更新数据的时候，尝试去更新数据，如果失败就不断尝试。

在 JAVA 中，`sun.misc.Unsafe` 类提供了硬件级别的原子操作来实现这个 CAS。 `java.util.concurrent` 包下的大量类都使用了这个 `Unsafe.java` 类的 CAS 操作，`java.util.concurrent.atomic` 包下的类大多是使用 CAS 操作来实现的。

ABA 问题，加版本号解决

#### 5. AQS

AQS：AbstractQueuedSynchronizer，即**队列同步器**。它是构建锁或者其他同步组件的基础框架（如 ReentrantLock公平锁实现、CountDownLatch、Semaphore 等），JUC 并发包的作者（**Doug Lea**）期望它能够成为实现大部分同步需求的基础。它是 JUC 并发包中的核心基础组件。AQS 中有两个重要的成员：

- 成员变量 state。用于表示锁现在的状态，用 volatile 修饰，保证内存一致性。**同时所用对 state 的操作都是使用 CAS 进行的**。state 为 0 表示没有任何线程持有这个锁，线程持有该锁后将 state 加 1，释放时减 1。多次持有释放则多次加减。

- 还有一个双向链表，链表除了头结点外，每一个节点都记录了线程的信息，代表一个等待线程。这是一个 FIFO 的链表。

这个抽象类被设计为作为一些可用原子 int 值来表示状态的同步器的基类。

#### 6. 分段锁

分段锁，**是一种锁的设计思路**，它细化了锁的粒度，主要运用在ConcurrentHashMap中，实现高效的并发操作，当操作不需要更新整个数组时，就只锁数组中的一项就可以了。

#### 参考资料

- https://www.cnblogs.com/tong-yuan/p/10674283.html



## JVM 知识

### 类加载机制
- 类是在运行期间第一次使用时动态加载的，而不是一次性加载所有类。因为如果一次性加载，那么会占用很多的内存
- 类加载过程包含了加载、验证、准备、解析和初始化这 5 个阶段
- 加载过程完成以下三件事
通过类的完全限定名称获取定义该类的二进制字节流。
将该字节流表示的静态存储结构转换为方法区的运行时存储结构。
在内存中生成一个代表该类的 Class 对象，作为方法区中该类各种数据的访问入口。
- 验证
- 准备
类变量是被 static 修饰的变量，准备阶段为类变量分配内存并设置初始值，使用的是方法区的内存。


#### Java 为什么要设计双亲委派模型？

一个类加载器首先将类加载请求转发到父类加载器，只有当父类加载器无法完成时才尝试自己加载

**BootstrapClassLoader**（**启动类加载器**，C++实现） -> **ExtensionClassLoader (扩展类加载器)**  -> **AppClassLoader (应用程序类加载器)**

双亲委派模型保证了 Java 程序的稳定运行，可以避免类的重复加载（JVM 区分不同类的方式不仅仅根据类名，相同的类文件被不同的类加载器加载产生的是两个不同的类），也保证了 Java 的核心 API 不被篡改。如果没有使用双亲委派模型，而是每个类加载器加载自己的话就会出现一些问题，比如我们编写一个称为 `java.lang.Object` 类的话，那么程序运行的时候，系统就会出现多个不同的 `Object` 类。

为了避免双亲委托机制（就像自己实现一个 Object 类），我们可以自己定义一个类加载器，然后重载 `loadClass()` 即可。如果我们要自定义自己的类加载器，很明显需要继承 `ClassLoader`。

### 运行时数据区

![](https://kolly-imgstore.oss-cn-shenzhen.aliyuncs.com/img/JVM 运行时数据区.png)

Java 虚拟机所管理的内存中最大的一块，Java 堆是所有线程共享的一块内存区域，在虚拟机启动时创建。**此内存区域的唯一目的就是存放对象实例，几乎所有的对象实例以及数组都在这里分配内存。**

Java 堆是垃圾收集器管理的主要区域，因此也被称作 **GC 堆（Garbage Collected Heap）**. 从垃圾回收的角度，由于现在收集器基本都采用分代垃圾收集算法，所以 Java 堆还可以细分为：新生代和老年代：再细致一点有：Eden 空间、From Survivor、To Survivor 空间等。**进一步划分的目的是更好地回收内存，或者更快地分配内存。**

![img](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-3%E5%A0%86%E7%BB%93%E6%9E%84.png)

上图所示的 eden 区、s0 区、s1 区都属于新生代，tentired 区属于老年代。大部分情况，对象都会首先在 Eden 区域分配，在一次新生代垃圾回收后，如果对象还存活，则会进入 s0 或者 s1，并且对象的年龄还会加 1 (Eden 区 ->Survivor 区后对象的初始年龄变为 1)，当它的年龄增加到一定程度（默认为 15 岁），就会被晋升到老年代中。对象晋升到老年代的年龄阈值，可以通过参数 `-XX:MaxTenuringThreshold` 来设置。大对象会直接分配在老年代中。

### 垃圾收集算法

#### 复制算法

效率高，复制操作会浪费部分空间

用于新生代 GC（Minor GC），每次收集都会有大量对象死去，所以可以选择复制算法，只需要付出少量对象的复制成本就可以完成每次垃圾收集

#### 标记 - 清除算法

效率低，产生大量空间碎片，CMS 收集器收集器使用算法。

#### 标记 - 整理算法

用于老年代 GC（Full GC），老年代的对象存活几率是比较高的，而且没有额外的空间对它进行分配担保，所以我们必须选择 “标记 - 清除” 或 “标记 - 整理” 算法进行垃圾收集。

### 垃圾收集器

1. **Serial 收集器**：历史最悠久，单线程工作，回收垃圾时，必须暂停所有其它线程 ——stop the world，采用复制算法；
2. **ParNew 收集器**：本质为 Serial 收集器的多线程版本，采用复制算法；
3. **Parallel scavenge**：具备自使用调节功能，以提供最合适的暂停时间和吞吐量，采用复制算法；
4. **Serial old 收集器**：是 Serial 收集器的老年代版本，同样为单线程，但采用的是 “标记 - 整理” 算法；
5. **Parallel old 收集器**：Parallel scavenge 收集器的老年代版本，多线程，采用的是 “标记 - 整理” 算法；
6. **CMS 收集器**：即 Concurrent Mark Sweep 收集器，以获取最短停顿时间为目标，采用 “标记 - 清除” 算法；
7. **G1 收集器**： 即 Garbage-First 收集器，是目前最新的收集器，采用与其它收集器完全不同的设计思想，历时十年才实现商用，采用了混合算法，兼有 “复制” 和 “标记 - 整理” 算法的特点；

JVM 中，青年代和老年代特点迥异，青年代中对象 “朝生夕死” 的特点，回收频率较高，适合采用复制算法；而老年代则更适合 “标记 - 整理” 算法。鉴于此，JVM 采用**分代回收**的策略：青年代采用复制算法的回收器，老年代采用 “标记 - 整理” 算法的回收器。

### JVM 常用参数

#### 内存分配

-Xss（默认 1M）、-Xms、-Xmx、-Xmn

-XX:NewRatio（默认值 2，新老 1:2）、-XX:SurvivorRatio（默认 8，1:1:8）

-XX:MaxTenuringThreshold（年轻代最大 gc 年龄、默认 15）

#### 设置垃圾收集器参数

**-XX:+UseSerialGC**，虚拟机运行在 Client 模式下的默认值，Serial + Serial Old。

**-XX:+UseParNewGC**，ParNew + Serial Old，在 JDK1.8 被废弃，在 JDK1.7 还可以使用。

**-XX:+UseParallelGC**，虚拟机运行在 Server 模式下的默认值，Parallel Scavenge + Serial Old

**-XX:+UseParallelOldGC**，Parallel Scavenge + Parallel Old。

**-XX:+UseConcMarkSweepGC**，ParNew + CMS + Serial Old。

**-XX:+UseG1GC**，G1 + G1。

#### 其他

 **-XX:+UseCompressedOops**， 压缩指针（8 字节 -> 4字节），起到节约内存占用

CompressedOops，可以让跑在 64 位平台下的 JVM，不需要因为更宽的寻址，而付出 Heap 容量损失的代价。
不过，它的实现方式是在机器码中植入压缩与解压指令，可能会给 JVM 增加额外的开销。

### JVM 调优

- jps、jmap、jstack、jhat

- load 资源过大，而 heap 不大，造成反复 Full GC（三方征信系统，接口返回报文过大）

- jmap 导出 dump 文件用 mat 工具分析，发现有个超大 ArrayList 对象（统计表数据过多，全部加载入内存）

  jstat -gcutil 分析发现 Full GC 触发频繁并且耗时较长

- CPU100%，jstack 发现线程死锁，修改 HashMap 为 ConcurrentHashMap

- JDK 自带的工具如 jvisualvm 查看线程运行状态，如果有线程在等待锁，可能发生死锁，这时可以定位到代码具体位置

### 参考资料

- [深入理解 JVM 01 | 运行时数据区域](https://www.jianshu.com/p/af3ed28616c6)
- [深入理解 JVM 02 | 垃圾收集](https://www.jianshu.com/p/d4c43dbe9367)
- [https://cyc2018.github.io/CS-Notes/#/notes/Java%20%E8%99%9A%E6%8B%9F%E6%9C%BA](https://cyc2018.github.io/CS-Notes/#/notes/Java 虚拟机)
- https://xiaozhuanlan.com/topic/1847690325

## Java IO
### NIO

Java NIO 包含了很多东西，但核心的东西不外乎 Buffer、Channel 和 Selector

http://www.tianxiaobo.com/categories/foundation-of-java/NIO/



## 设计模式

### 装饰模式
JDK：InputStream

mybatis： 缓存实现



### 单例

懒汉、饿汉、双重锁、枚举、静态内部类

静态内部类：代码简洁、不用加锁。静态的成员式内部类，该内部类的实例与外部类的实例没有绑定关系，而且只有被调用到时才会装载(装载过程是由 JVM 保证线程安全) ，从而实现了延迟加载。

- Dubbo SPI 机制 `ExtensionLoader` 的 `getExtension` 方法

- Spring IOC 容器 `BeanFactory` 的 `getBean` 方法