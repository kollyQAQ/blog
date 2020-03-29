[TOC]

### 核心提炼

- `Thread` 类有维护了一个属性变量 `threadLocals` （ThreadLocal.ThreadLocalMap threadLocals = null），也就是说每个线程有都一个自己的 `ThreadLocalMap` ，所以每个线程往这个 `ThreadLocal` 中读写隔离的，并且是互相不会影响的。

- `ThreadLocalMap` 类是 `ThreadLocal` 的静态内部类

- `ThreadLocalMap` 维护了一个 `Entry` 数组，`Entry` 的 key 是 `ThreadLocal` 对象，value 是存入的对象，所以一个 `ThreadLocal` 只能存储一个Object对象，如果需要存储多个Object对象那么就需要多个 `ThreadLocal`

- `Entry` 的 key 引用 `ThreadLocal` 是弱引用

- `Thread`、`ThreadLocalMap`、`ThreadLocal`总览图如下

  ![image](https://upload-images.jianshu.io/upload_images/7017140-5488416588943e1f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

  ![image](https://upload-images.jianshu.io/upload_images/7017140-fdeb80688ac14be0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### ThreadLocal 是用来干嘛的

**ThreadLocal 主要是用在多线程的场景中**

- **保存线程上下文信息，在任意需要的地方可以获取** (比如下面案例中的保存用户信息)
- **线程安全，避免某些情况需要考虑线程安全必须同步带来的性能损失**

### 使用场景案例

在spring MVC开发中，我们常用 ThreadLocal 保存当前登陆用户信息，这样线程在任意地方都可以取到用户信息，比如我们会有以下类似下面的 `UserContext` 类，然后给配置一个拦截器，拦截器里面在请求执行前调用 `UserContext` 的 `setUserInfo` 方法将用户信息存入 `ThreadLocal` 对象 `userInfoLocal` 中，然后在请求的具体执行的任意地方调用 `UserContext` 的 `getUserInfo` 方法取出用户信息，最后在拦截器里面在请求结束返回前调用 `UserContext` 的 `clear` 方法清除数据

```java
public class UserContext {
    private static final ThreadLocal<UserInfo> userInfoLocal = new ThreadLocal<UserInfo>();

    public static UserInfo getUserInfo() {
        return userInfoLocal.get();
    }

    public static void setUserInfo(UserInfo userInfo) {
        userInfoLocal.set(userInfo);
    }

    public static void clear() {
        userInfoLocal.remove();
    }

}
```

#### ThreadLocal 使用代码示例

```java
public class ThreadLocalTest {
    private static ThreadLocal<Integer> threadLocal = new ThreadLocal<>();
    public static void main(String[] args) {
        new Thread(() -> {
            try {
                for (int i = 0; i < 5; i++) {
                    threadLocal.set(i);
                    System.out.println(Thread.currentThread().getName() + "====" + threadLocal.get());
                    try {
                        Thread.sleep(200);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            } finally {
                threadLocal.remove();
            }
        }, "thread-1").start();

        new Thread(() -> {
            try {
                for (int i = 0; i < 5; i++) {
                    System.out.println(Thread.currentThread().getName() + "====" + threadLocal.get());
                    try {
                        Thread.sleep(200);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            } finally {
                threadLocal.remove();
            }
        }, "thread-2").start();
    }
}

```

#### 运行结果

```
thread-1====0
thread-2====null
thread-2====null
thread-1====1
thread-2====null
thread-1====2
thread-2====null
thread-1====3
thread-2====null
thread-1====4
```

从运行结果可以看出，`thread-1`线程中对`threadLocal`对象的赋值对`thread-2`线程中`threadLocal`对象的值任何影响

### 源码细节

#### Thread 类

```java
public class Thread implements Runnable {
  /* ThreadLocal values pertaining to this thread. This map is maintained
     * by the ThreadLocal class. */
		ThreadLocal.ThreadLocalMap threadLocals = null;
}
```

`Thread `类有属性变量 `threadLocals` （类型是ThreadLocal.ThreadLocalMap），也就是说每个线程有一个自己的 `ThreadLocalMap` ，所以每个线程往这个`ThreadLocal`中读写隔离的，并且是互相不会影响的

**一个ThreadLocal只能存储一个Object对象，如果需要存储多个Object对象那么就需要多个ThreadLocal**

#### ThreadLocal 类

##### 类签名

```java
public class ThreadLocal<T> {

}
```

##### 关键方法 | set

```java
public void set(T value) {
        Thread t = Thread.currentThread();
  			// ** 取出当前线程维护的 ThreadLocalMap 对象 **
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }

// 创建一个 ThreadLocalMap 赋值为当前 Thread 对象的属性，并添加第一个 Entry
void createMap(Thread t, T firstValue) {
        t.threadLocals = new ThreadLocalMap(this, firstValue);
    }
```

##### 关键方法 | get

```java
public T get() {
        Thread t = Thread.currentThread();
  			// ** 取出当前线程维护的 ThreadLocalMap 对象 **
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
    }

// 取出线程维护的 ThreadLocalMap 对象
ThreadLocalMap getMap(Thread t) {
        return t.threadLocals;
    }
```

##### 其他方法

```java
public void remove() {
         ThreadLocalMap m = getMap(Thread.currentThread());
         if (m != null)
             m.remove(this);
     }
```

#### ThreadLocalMap 类

`ThreadLocalMap`是`ThreadLocal`的静态内部类

##### 类签名

```java
public class ThreadLocal<T> {
  	// ThreadLocalMap 是 ThreadLocal 的静态内部类
	static class ThreadLocalMap {
        
		// Entry 是 ThreadLocalMap 的静态内部类
      	static class Entry extends WeakReference<ThreadLocal<?>> {
            /** The value associated with this ThreadLocal. */
            Object value;

          	// Entry 的 key 是 ThreadLocal 对象，value 是关联的数据对象
            Entry(ThreadLocal<?> k, Object v) {
                super(k);
                value = v;
            }
        }
      
        // ThreadLocalMap 维护了一个 Entry 数组，
      	private Entry[] table;
    }    
}
```

##### 构造方法

```java
ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
            table = new Entry[INITIAL_CAPACITY];
            int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
            table[i] = new Entry(firstKey, firstValue);
            size = 1;
            setThreshold(INITIAL_CAPACITY);
        }
```

##### 关键方法 | set

```java
private void set(ThreadLocal<?> key, Object value) {
            Entry[] tab = table;
            int len = tab.length;
            int i = key.threadLocalHashCode & (len-1);

  					// 如果 ThreadLocal 在 Entry 数组中已经存在，则替换其 value
            for (Entry e = tab[i];
                 e != null;
                 e = tab[i = nextIndex(i, len)]) {
                ThreadLocal<?> k = e.get();

                if (k == key) {
                    e.value = value;
                    return;
                }

                if (k == null) {
                    replaceStaleEntry(key, value, i);
                    return;
                }
            }

  					// 不存在，则新建一个 Entry 插入到 Entry数组中
            tab[i] = new Entry(key, value);
            int sz = ++size;
            if (!cleanSomeSlots(i, sz) && sz >= threshold)
                rehash();
        }
```

##### 关键方法 | getEntry

```java
private Entry getEntry(ThreadLocal<?> key) {
            int i = key.threadLocalHashCode & (table.length - 1);
            Entry e = table[i];
            if (e != null && e.get() == key)
                return e;
            else
                return getEntryAfterMiss(key, i, e);
        }
```

##### 其他方法

```java
/**
         * Remove the entry for key.
         */
        private void remove(ThreadLocal<?> key) {
            Entry[] tab = table;
            int len = tab.length;
            int i = key.threadLocalHashCode & (len-1);
            for (Entry e = tab[i];
                 e != null;
                 e = tab[i = nextIndex(i, len)]) {
                if (e.get() == key) {
                    e.clear();
                    expungeStaleEntry(i);
                    return;
                }
            }
        }
```

#### ThreadLocalMap 里 Entry 为何声明为 WeakReference？

```java
// Entry 是 ThreadLocalMap 的静态内部类
static class Entry extends WeakReference<ThreadLocal<?>> {
    /** The value associated with this ThreadLocal. */
    Object value;

    Entry(ThreadLocal<?> k, Object v) {
        super(k);
        value = v;
    }
}
```

##### WeakReference是什么

1. 强引用（StrongReference）：被强引用关联的对象不会被回收
2. 软引用（SoftReference）：被软引用关联的对象只有在内存不够的情况下才会被回收
3. 弱引用（WeakReference）：被弱引用关联的对象一定会被回收，也就是说它只能存活到下一次垃圾回收发生之前

>  WeakReference 是 Java 语言规范中为了区别直接的对象引用（程序中通过构造函数声明出来的对象引用）而定义的另外一种引用关系。WeakReference 标志性的特点是：reference 实例不会影响到被引用对象的 GC 回收行为（即只要对象被除 WeakReference 对象之外所有的对象解除引用后，该对象便可以被 GC 回收），只不过在被对象回收之后，reference 实例想获得被应用的对象时程序会返回 null。

##### 为什么ThreadLocalMap的key用弱引用，为什么不用强引用呢?

**看回这个例子**

```java
public class UserContext {
    private static final ThreadLocal<UserInfo> userInfoLocal = new ThreadLocal<UserInfo>();

    public static UserInfo getUserInfo() {
        return userInfoLocal.get();
    }

    public static void setUserInfo(UserInfo userInfo) {
        userInfoLocal.set(userInfo);
    }

    public static void clear() {
        userInfoLocal.remove();
    }

}
```

此时 Entry 的情况是

```java
key instance of WeakReference<ThreadLocal<UserInfo>>
value instance of UserInfo
```

WeakReference 对引用的对象 userInfoLocal 是弱引用，不会影响到 userInfoLocal 的 GC 行为。  如果是强引用的话，在线程运行过程中，我们不再使用 userInfoLocal 了，将 userInfoLocal 置为 null，但 userInfoLocal 在线程的 ThreadLocalMap 里还有引用，导致其无法被GC回收（当然，可以等到线程运行结束后，整个Map都会被回收，但很多线程要运行很久，如果等到线程结束，便会一直占着内存空间）。  而 Entry 声明为 WeakReference，userInfoLocal 置为 null 后，线程的 threadLocalMap 就不算强引用了，userInfoLocal 就可以被GC回收了。map的后续操作中，也会逐渐把对应的"stale entry"清理出去，避免内存泄漏。

所以，我们在使用完 ThreadLocal 变量时，尽量用threadLocal.remove()来清除，避免 threadLocal=null 的操作。  前者 remove() 会同时清除掉线程 threadLocalMap 里的 entry，算是彻底清除；而后者虽然释放掉了 threadLocal，但线程 threadLocalMap 里还有其"stale entry"，后续还需要处理。

这里的弱引用可以首先由 gc 来判断 ThreadLocal 实例（userInfoLocal）是否真的可以回收，由 gc 回收的结果，间接告诉我们，key 为 null 了，这时候 value 也可以被清理了，并且最终通过高频操作get/set/remove封装好的方法进行清理。如果用强引用那么我们一直不知道这个entry是否可以被回收，除非强制每个coder在逻辑执行完的最后进行一次全局清理。

##### 为什么value不用弱引用呢？

value 不像 key 那样，还有一个外部的强引用(userInfoLocal)，如果将 value 设置为弱引用，可能在业务执行过程中发生了gc，value 就被清理了，业务后边取值会出错的。

##### set/get/remove方法，这些方法会对key为null的entry进行释放

### 参考资料

- https://cloud.tencent.com/developer/article/1125219