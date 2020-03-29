## **netstat命令**

> netstat -an | grep 3306 

3306替换成需要grep的端口号

 

## **lsof命令**

通过list open file命令可以查看到当前打开文件，在linux中所有事物都是以文件形式存在，包括网络连接及硬件设备。

> lsof -i:80

**-i**参数表示网络链接，**:80**指明端口号，该命令会同时列出PID，方便kill

查看所有进程监听的端口

> lsof -i -P | grep -i "listen" 

