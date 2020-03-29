### 总结图
![image](https://upload-images.jianshu.io/upload_images/7017140-80374e65c6b20cbe?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### Java 内存结构
#### 第一部分：对象头

1.  **markword**：用于存储对象自身的运行时数据，如哈希码、GC分代年龄、锁状态标志、线程持有的锁等。这部分数据长度在32位机器和64位机器虚拟机中分别为4字节和8字节（64位的JVM为了节约内存可以使用选项+UseCompressedOops开启指针压缩，开启该选项后，占用字节数降为4字节）；
![image](https://upload-images.jianshu.io/upload_images/7017140-2e2d36d7271a318a?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1.  **类型指针**：即对象指向它的类元数据（保存在方法区）的指针，虚拟机通过这个指针来确定这个对象属于哪个类的实例，指针占用4个字节（64位机器占8个字节）；
2.  **数组长度**（只有数组对象才有）：如果是 Java 数组，对象头必须有一块用于记录数组长度的数据，用4个字节int来记录数组长度；

#### 第二部分：实例数据
实例数据是对象真正存储的有效信息，也是程序代码中定义的各种类型的字段内容。无论是从父类继承下来还是在子类中定义
的数据，都需要记录下来。

原生类型的内存占用情况如下：
*   boolean 1
*   byte 1
*   short 2
*   char 2
*   int 4
*   float 4
*   long 8
*   double 8

引用类型的内存占用和系统位数以及启动参数UseCompressedOops有关
*   32位系统占4字节
*   64位系统，开启 UseCompressedOops时，占用4字节，否则是8字节

#### 第三部分：对齐填充

对于hotspot迅疾的自动内存管理系统要求对象的起始地址必须为8字节的整数倍，这就要求当部位8字节的整数倍时，就需要填充数据对其填充。原因是访问未对齐的内存，处理器需要做两次内存访问，而对齐的内存访问仅需一次访问。

**如何计算或者获取某个 Java 对象的大小?**

32 位系统下，当使用 new Object() 时，JVM 将会分配 8（Mark Word+类型指针） 字节的空间，128 个 Object 对象将占用 1KB 的空间。

如果是 new Integer()，那么对象里还有一个 int 值，其占用 4 字节，这个对象也就是 8+4=12 字节，对齐后，该对象就是 16 字节。

以上只是一些简单的对象，那么对象的内部属性是怎么排布的？

```
Class A {
    int i;
    byte b;
    String str;
}
```

其中对象头部占用 「Mark Word」4 + 「类型指针」4 = 8 字节；byte 8 位长，占用 1 字节；int 32 位长，占用 4 字节；String 只有引用，占用 4 字节；

那么对象 A 一共占用了 8+1+4+4=17 字节，按照 8 字节对齐原则，对象大小也就是 24 字节。

这个计算看起来是没有问题的，对象的大小也确实是 24 字节，但是对齐（padding）的位置并不对：

在 HotSpot VM 中，对象排布时，间隙是在 4 字节基础上的（在 32 位和 64 位压缩模式下），上述例子中，int 后面的 byte，空隙只剩下 3 字节，接下来的 String 对象引用需要 4 字节来存放，因此 byte 和对象引用之间就会有 3 字节对齐，对象引用排布后，最后会有 4 字节对齐，因此结果上依然是 7 字节对齐。此时对象的结构示意图，如下图所示：

![image](https://upload-images.jianshu.io/upload_images/7017140-9100ca7c0664a3c3?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**代码验证**

```
public class Main {
    public static void main(String[] args) throws Exception {
        final List<AAAAA> aaa = new ArrayList<>(100000);
        final List<BBBBB> bbb = new ArrayList<>(100000);
        final List<CCCCC> ccc = new ArrayList<>(100000);
        final List<DDDDD> ddd = new ArrayList<>(100000);

        for (int i = 0; i < 100000; i++) {
            aaa.add(new AAAAA());
            bbb.add(new BBBBB());
            ccc.add(new CCCCC());
            ddd.add(new DDDDD());
        }

        System.in.read();
    }
}

class AAAAA {
}

class BBBBB {
    int a = 1;
}

class CCCCC {
    long a = 1L;
}

class DDDDD {
    String s = "hello";
}
```

本地的执行环境是64位的JDK8，且使用默认的启动参数，运行之后通过 jmap-dump命令生成dump文件，用MAT打开

![image](https://upload-images.jianshu.io/upload_images/7017140-1ba2591918d5d3d0?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

*   A对象只包含一个对象头，大小占12字节（markword 4 字节，类型指针 8 字节），不是8的倍数，需要4字节进行填充，一共占16字节
*   B对象包含一个对象头和int类型，12+4=16，正好是8的倍数，不需要填充，占16字节。
*   C对象包含一个对象头和long类型，12+8=20，不是8的倍数，需要4个字节进行填充，占24字节
*   D对象包含一个对象头和引用类型，12+4=16，正好是8的倍数，不需要填充，占16字节。

因为 UseCompressedOops 在 JDK8 是是默认开启的，为了验证不开启时的内存占用情况，可以添加参数 -XX:-UseCompressedOops  主动关闭指针压缩，结果如下

![image](https://upload-images.jianshu.io/upload_images/7017140-7aa942d969fac53b?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.gif](https://upload-images.jianshu.io/upload_images/7017140-af28547860746c2d.gif?imageMogr2/auto-orient/strip) 

*   A对象只包含一个对象头，大小占16字节（markword 8 字节，类型指针 8 字节），一共占16字节
*   B对象包含一个对象头和int类型，16+4=20，填充后占 24 字节。
*   C对象包含一个对象头和long类型，16+8=24，占24字节
*   D对象包含一个对象头和引用类型，16+8=24，占 24 字节。

### 参考资料
[https://www.cnblogs.com/magialmoon/p/3757767.html](https://www.cnblogs.com/magialmoon/p/3757767.html)
[https://mp.weixin.qq.com/s/BfWMp-3vPcg1eMgL4D249g](https://mp.weixin.qq.com/s/BfWMp-3vPcg1eMgL4D249g)