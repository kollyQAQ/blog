#### 去除文件名中的非目录部分，仅显示与目录有关的内容

```
[root@localhost ~]# dirname gotoredis.sh
.
[root@localhost ~]# dirname data/src/install-php7.sh
data/src
[root@localhost ~]# dirname /usr/src/zookeeper-3.4.10.tar.gz
/usr/src
```

