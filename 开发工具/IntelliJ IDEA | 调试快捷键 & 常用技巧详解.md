[TOC]

### Debug 常用快捷键

| 快捷键      | 介绍                                                         |
| ----------- | ------------------------------------------------------------ |
| F7          | 进入函数                                                     |
| Shift +  F7 | 进入函数，如果断点所在行上有多个方法调用，会弹出进入哪个方法 |
| F8          | 进入下一步，如果当前行断点是一个方法，则不进入当前方法体内   |
| F9          | 恢复程序运行，但是如果该断点下面代码还有断点则停在下一个断点上 |
| F10         | 执行到光标处                                                 |
| Alt + F8    | 选中对象，弹出可输入计算表达式调试框，查看该输入内容的调试结果 |
| Shift + F8  | 跳出当前函数                                                 |

### 调试技巧

#### 条件断点

我们调试自己的项目或者看源码的时候，经常遇到 for 循环的情况，但是我们只想debug循环中的一种情况，如果我们在 for 循环中打了断点，需要 F9 N 次之后才能进入想要的 debug 的地方，效率实在太低。这个时候就推荐使用条件断点，鼠标放在断点上右键就可以弹出窗口填写条件了，如下图

![ceshi](https://upload-images.jianshu.io/upload_images/7017140-cc19d35ac7688263?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 强制返回值

调试过程中，经常遇到要测试多个分支的情况，但是实际上要通过模拟数据让代码进入各个分支的情况并不是那么容易，所有强制返回值就可以解决这种问题，通过自己设置函数的返回值来控制流程，下面介绍使用的例子

![image](https://upload-images.jianshu.io/upload_images/7017140-3b1640dea7a6f42b?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image](https://upload-images.jianshu.io/upload_images/7017140-6abacca0d24fc604?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image](https://upload-images.jianshu.io/upload_images/7017140-21e33d6150d055d4?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image](https://upload-images.jianshu.io/upload_images/7017140-140a26d57c8f73e9?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 模拟异常

在做业务开发中，我们有时需要模拟某个方法抛出异常，看看自己的代码是不是可靠。但是你每次去写死一个异常,然后再删掉，这种方式太过于低效。其实我们可以模拟让一个方法抛出异常

![image](https://upload-images.jianshu.io/upload_images/7017140-34a5785f13f71af3?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image](https://upload-images.jianshu.io/upload_images/7017140-fe7a9e4bf5c492ee?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 表达式计算(Evaluate Expression)

比如我们看源码时遇到这个一个场景,这里有一个 `byte[]`,但是我们就想看一下这个的值到底是啥

![image](https://upload-images.jianshu.io/upload_images/7017140-988f704a4e35d8c6?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

那么我们可以这么操作一波，按 `ALT+F8`弹出表达式弹窗

![image](https://upload-images.jianshu.io/upload_images/7017140-20fe94b22f523574?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

还有个功能的使用场景非常的广，通过这个功能，可以在看源码时,给某个变量赋我们要想的值，从而改变代码的分支走向等等，下面看一下使用方法

![image](https://upload-images.jianshu.io/upload_images/7017140-8410ce3d82b7e47c?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这个时候按 `ALT+F8`弹出表达式弹窗，可以自定义设置变量的值

![image](https://upload-images.jianshu.io/upload_images/7017140-34fc951f9ffc2542?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)