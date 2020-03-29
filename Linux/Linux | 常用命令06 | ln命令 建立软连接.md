#### 总结

**ln  -s  soruce  target**

创建指向**soruce**的符号链接**target**

**ln  -s  log2013.log  link2013**

为log2013.log文件创建软链接link2013，如果log2013.log丢失，link2013将失效

**ln  -s  /data/jweblog  log**

为/data/jweblog目录创建软链接log

------

#### 1．命令格式

> ln [参数][源文件或目录][目标文件或目录]

#### 2．命令功能

Linux文件系统中，有所谓的链接(link)，我们可以将其视为档案的别名，而链接又可分为两种 : 硬链接(hard link)与软链接(symbolic link)，硬链接的意思是一个档案可以有多个名称，而软链接的方式则是产生一个特殊的档案，该档案的内容是指向另一个档案的位置。硬链接是存在同一个文件系统中，而软链接却可以跨越不同的文件系统。

#### 3．命令参数

**必要参数:**

-b 删除，覆盖以前建立的链接

-d 允许超级用户制作目录的硬链接

-f 强制执行

-i 交互模式，文件存在则提示用户是否覆盖

-n 把符号链接视为一般目录

**-s 软链接(符号链接)**

-v 显示详细的处理过程

#### **4．使用实例**

**实例1：给文件创建软链接**

**命令：**

ln -s log2013.log link2013

**实例2：给文件创建硬链接**

**命令：**

ln log2013.log ln2013

**输出：**

```shell
[root@localhost test]# ll
-rw-r--r-- 1 root bin      61 11-13 06:03 log2013.log

[root@localhost test]# ln -s log2013.log link2013
[root@localhost test]# ll
lrwxrwxrwx 1 root root     11 12-07 16:01 link2013 -> log2013.log
-rw-r--r-- 1 root bin      61 11-13 06:03 log2013.log
```