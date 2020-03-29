[TOC]

### 一些规律
* X选项：-Xms128m  -xmx2g
* XX选项：如果是布尔类型的选项，它的格式为-XX:+flag或者-XX:-flag，分别表示开启和关闭该选项
* XX选项：针对非布尔类型的选项，它的格式为-XX:flag=value
* 例子：java -Xms128m -Xmx2g  -XX:+UseParallelGC -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/data/logs  MyApp

### 基本参数
-Xms<value>   初始的最小堆体积
-Xmx<value>   最大堆体积
-Xss<value>    每个线程的堆栈大小

#### 新生代参数
-XX:NewRatio=<value>     老年代和新生代的比例（默认情况下，这个数值是 2，意味着老年代是新生代的 2 倍大；换句话说，新生代是堆大小的 1/3）
-XX:SurvivorRatio=<value>  Eden 和 Survivor 的大小的比例（默认是8，Survivor 区域是 Eden 的 1/8 大小，也就是新生代的 1/10）

#### 进阶参数
-XX:MaxTenuringThreshold=value  对象被晋升到老年代的阈值
OOM相关的选项
-XX:+HeapDumpOnOutOfMemoryError：表示在内存出现OOM的时候，把Heap转存(Dump)到文件以便后续分析，文件名通常是java_pid<pid>.hprof，其中pid为该程序的进程号。
-XX:HeapDumpPath=<path>: 用来指定heap转存文件的存储路径，需要指定的路径下有足够的空间来保存转存文件。
-XX:OnOutOfMemoryError：用来指定一个可行性程序或者脚本的路径，当发生OOM的时候，去执行这个脚本

```
$ java -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/tmp/heapdump.hprof -XX:OnOutOfMemoryError ="sh ~/cleanup.sh" MyApp
```



####查看当前 JVM 启动参数的值
 -XX:+PrintCommandLineFlags：打印出用户手动设置或者JVM自动设置的XX选项
-XX:+PrintFlagsInitial：打印出所有XX选项的默认值
-XX:+PrintFlagsFinal：打印出XX选项在运行程序时生效的值
指定JIT编译器的模式
我们知道Java是一种解释型语言，但是随着JIT技术的进步，它能在运行时将Java的字节码编译成本地代码。
以下是几个相关的选项：
-Xint：表示禁用JIT，所有字节码都被解释执行，这个模式的速度最慢的
-Xcomp：表示所有字节码都首先被编译成本地代码，然后再执行
-Xmixed：默认模式，让JIT根据程序运行的情况，有选择地将某些代码编译成本地代码（-Xcomp和-Xmixed到底谁的速度快，针对不同的程序可能有不同的结果，基本还是推荐用默认模式）