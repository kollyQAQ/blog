[TOC]

#### 下载，解压，编译

```
$ cd /usr/local/src
$ wget http://download.redis.io/releases/redis-3.2.8.tar.gz
$ tar xzf redis-3.2.8.tar.gz
$ cd redis-3.2.8
$ make
```

如果没有安装tcl会报错

```
$ yum install tcl
$ make
```

如果没有安装gcc会报错(gcc: Command not found)

```
$ yum install gcc-c++
$ make
```

如果报错（Newer version of jemalloc required）

```
$ make MALLOC=libc
```

#### 拷贝文件

```
$ cd  /usr/local/src/redis-3.2.8/src  #二进制文件是编译完成后在src目录下
$ touch redis.conf #新建一个redis配置文件，官网有，记得修改一条：daemonize yes
$ cp redis.conf /etc/  
$ cp redis-benchmark redis-cli redis-server /usr/bin/  #方便在任何地方启动
```

#### 启动redis服务

```
$ redis-server /etc/redis.conf
```

#### 测试客户端连接

```
$ redis-cli  	
redis> set name zhangsan
OK  
 redis> get name   
"zhangsan"
```

#### 测试端口是否启动

```
$ netstat -tlnp
```

#### 设置开机启动

```
$ echo "redis-server  /etc/redis.conf" >>/etc/rc.local
```

#### 关闭redis服务

```
$ redis-cli shutdown save
```