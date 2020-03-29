[TOC]

## 第一步：修改 host

打开 host 文件

```shell
vim /ect/hosts
```

在后面一行添加

```
0.0.0.0 account.jetbrains.com
```

## 第二步：下载激活包

### 下载 `jetbrains-agent.jar`

2019.2 版本下载地址 [https://github.com/Gleans/crack/releases](https://github.com/Gleans/crack/releases)

### 拷贝jar文件

显示包内容

![](https://kolly-imgstore.oss-cn-shenzhen.aliyuncs.com/img/IDEA 激活 | 显示包内容.png)

将 `jetbrains-agent.jar`存放到 bin 目录下

![](https://kolly-imgstore.oss-cn-shenzhen.aliyuncs.com/img/IDEA激活|拷贝 jar.png)

## 第三步：修改配置文件

打开 bin目录下的 `idea.vmoptions` 文件

在最后一行增加如下代码

```
-javaagent:jetbrains-agent.jar
```

Mac环境下请给予权限，windows忽略

```shell
cd /Applications/IntelliJ\ IDEA.app/Contents/bin/  

chmod 777 jetbrains-agent.jar
```

## 第四步：启动 IDEA

![](https://kolly-imgstore.oss-cn-shenzhen.aliyuncs.com/img/IDEA 激活 | 破解 1.png)

选择 `License Server` 然后点击 `Discover Server`
或者自行输入

```
http://jetbrains-license-server
```

![](https://kolly-imgstore.oss-cn-shenzhen.aliyuncs.com/img/IDEA 激活|破解 2.png)