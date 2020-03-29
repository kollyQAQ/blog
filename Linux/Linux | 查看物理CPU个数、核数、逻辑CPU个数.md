CPU总核数 = 物理CPU个数 * 每颗物理CPU的核数

总逻辑CPU数 = 物理CPU个数 * 每颗物理CPU的核数 * 超线程数

```shell
# 查看CPU信息（型号）
[root@AAA ~]# cat /proc/cpuinfo | grep name | cut -f2 -d: | uniq -c
     8         Intel(R) Xeon(R) CPU E5-2630 0 @ 2.30GHz
     
# 查看物理CPU个数
[root@AAA ~]# cat /proc/cpuinfo| grep "physical id"| sort| uniq| wc -l
1

# 查看每个物理CPU中core的个数(即核数)
[root@AAA ~]# cat /proc/cpuinfo| grep "cpu cores"| uniq
cpu cores    : 4

# 查看逻辑CPU的个数
[root@AAA ~]# cat /proc/cpuinfo| grep "processor"| wc -l
8

# 查看汇总信息
[root@AAA ~]# lscpu
Architecture:          x86_64
CPU op-mode(s):        32-bit, 64-bit
Byte Order:            Little Endian
CPU(s):                8
On-line CPU(s) list:   0-7
Thread(s) per core:    2
Core(s) per socket:    4
Socket(s):             1
NUMA node(s):          1
Vendor ID:             GenuineIntel
CPU family:            6
Model:                 85
Model name:            Intel(R) Xeon(R) Platinum 8163 CPU @ 2.50GHz
Stepping:              4
CPU MHz:               2500.004
BogoMIPS:              5000.00
Hypervisor vendor:     KVM
Virtualization type:   full
L1d cache:             32K
L1i cache:             32K
L2 cache:              1024K
L3 cache:              33792K
NUMA node0 CPU(s):     0-7
```

这些都代表什么，那就请看CPU架构

多个物理CPU，CPU通过总线进行通信，效率比较低，如下：

![](http://ww2.sinaimg.cn/large/006tNc79gy1g4ozo7cew2j3097072749.jpg)

多核CPU，不同的核通过L2 cache进行通信，存储和外设通过总线与CPU通信，如下：

![](http://ww2.sinaimg.cn/large/006tNc79gy1g4ozobgzktj309e08amx5.jpg)

多核超线程,每个核有两个逻辑的处理单元，两个核共同分享一个核的资源，如下：

![](http://ww4.sinaimg.cn/large/006tNc79gy1g4ozofmlhyj309b05wjrb.jpg)

从上面执行的结果来看，证明我使用的cpu有1 * 4 = 4核，每个核有2个超线程，所以有8个逻辑cpu。