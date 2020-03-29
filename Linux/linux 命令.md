[TOC]

# linux 查看服务器信息

1. uname -a

```shell
Linux iZwz93qneew6w67gb6lmsbZ 3.10.0-957.21.3.el7.x86_64 #1 SMP Tue Jun 18 16:35:19 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
```

2. 查询系统发型版本

```shell
# 方法一
lsb_release -a

# 方法二
cat /proc/version

# 方法三
cat /etc/*-release
```

# 常用命令

1. 查看端口开放

```shell
ss -tlnp

netstat -apn|grep 3306
```