[TOC]

## 镜像常用命令

### 搜索镜像

```shell
docker search nginx
```

### 下载镜像

```shell
docker pull nginx:1.17.0
```

> 由于docker search命令只能查找出是否有该镜像，不能找到该镜像支持的版本，所以我们需要通过docker hub来搜索支持的版本

### 列出镜像

```shell
docker images
```

### 删除镜像

```shell
# 指定名称删除镜像
docker rmi java:8

# 指定名称删除镜像（强制）
docker rmi -f java:8
```



## 容器常用命令

### 新建并启动容器

```shell
docker run -p 80:80 --name nginx -d nginx:1.17.0
```

- **-d**选项：表示后台运行	
- **-i:** 以交互模式运行容器，通常与 -t 同时使用
- **-t:** 为容器重新分配一个伪输入终端，通常与 -i 同时使用
- **-p**选项：指定端口映射，格式为：hostPort:containerPort
- **--name**选项：指定运行后容器的名字为nginx,之后可以通过名字来操作容器
- **--link=[]:** 添加链接到另一个容器

### 列出容器信息

```shell
# 列出运行中的容器
docker ps

# 列出全部容器
docker ps -a
```

### 删除容器

```shell
# 删除一个容器
docker rm $ContainerName(或者$ContainerId)

# 强制删除所有容器
docker rm -f `docker ps -a -q`
```

### 启停容器

```shell
# 启动|停止|重启 某一容器的所有进程
docker start|stop|restart $ContainerName(或者$ContainerId)

# 暂停|恢复 某一容器的所有进程
docker pause|unpause $ContainerName(或者$ContainerId)

# 强制停止容器
docker kill $ContainerName(或者$ContainerId)
```

### 查看容器日志

```shell
docker logs $ContainerName(或者$ContainerId)
```

### 进入容器

- Docker exec

```shell
docker exec -it $ContainerName(或者$ContainerId) /bin/sh
```

- 使用nsenter （需安装）

```shell
# 先查询出容器的pid
docker inspect --format "{{.State.Pid}}" $ContainerName(或者$ContainerId)

# 根据容器的pid进入容器
nsenter --target "$pid" --mount --uts --ipc --net --pid
```

为了使用方便可以写一个脚本自动完成

```shell
$ vim /bin/docker_enter

#!/bin/bash
sudo nsenter --target `docker inspect --format {% raw %}{{.State.Pid}}{% endraw %} $1` --mount --uts --ipc --net --pid bash
```

这样每次要进入某个 container 只需要执行`docker_enter <container_name_or_ID>`就可以了

### 查看容器 IP 地址

```shell
docker inspect --format '{{ .NetworkSettings.IPAddress }}' $ContainerName(或者$ContainerId)
```

### 同步宿主机时间到容器

```shell
docker cp /etc/localtime $ContainerName(或者$ContainerId):/etc/
```

## 容器的备份、恢复和迁移

### 备份

### 恢复

### 迁移