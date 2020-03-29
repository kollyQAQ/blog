服务器报错：Java heap space

```
2018-07-10 10:15:05.147|ERROR|c.d.d.j.e.handler.impl.DefaultJobExceptionHandler - Job 'persistHistoryDataJob' exception occur in job processing
java.lang.OutOfMemoryError: Java heap space

2018-07-10 10:25:05.661|ERROR|c.d.d.j.e.handler.impl.DefaultJobExceptionHandler - Job 'persistHistoryDataJob' exception occur in job processing
java.lang.OutOfMemoryError: GC overhead limit exceeded
```

**查看java进程**

```
$ sudo jps -l
32577 com.xxxx.xxx.event.Main
25386 com.xxxx.xxx.counting.Main
```



**生成dump文件**

```
jmap -dump:format=b,file=20170307.dump 16048
```



**用jvisualvm分析dump文件**

jvisualvm是JDK自带的Java性能分析工具，在JDK的bin目录下，文件名就叫jvisualvm.exe，mac 直接在终端输入jvisualvm即可打开

jvisualvm可以监控本地、远程的java进程，实时查看进程的cpu、堆、线程等参数，对java进程生成dump文件，并对dump文件进行分析