[TOC]

#### 第一步：安装（电脑先安装好Homebrew）
```
[root@localhost ~]# brew install nginx
```
安装完以后，可以在终端输出的信息里看到一些配置路径和使用说明
```shell
==> nginx
Docroot is: /usr/local/var/www

The default port has been set in /usr/local/etc/nginx/nginx.conf to 8080 so that
nginx can run without sudo.

nginx will load all files in /usr/local/etc/nginx/servers/.

To have launchd start nginx now and restart at login:
  brew services start nginx
Or, if you don't want/need a background service you can just run:
  nginx
```
#### 第二步: 启动
```
[root@localhost ~] # brew services start nginx
```
#### 第三步：验证
```
[root@localhost ~] curl -IL http://127.0.0.1:8080
```
输出下面的信息证明启动成功啦
```
HTTP/1.1 200 OK
Server: nginx/1.15.9
Date: Mon, 25 Mar 2019 02:21:50 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Tue, 26 Feb 2019 15:29:26 GMT
Connection: keep-alive
ETag: "5c755b56-264"
Accept-Ranges: bytes
```
当然也可以在浏览器访问`http://127.0.0.1:8080`,正常启动的话会出现一个`Welcome to nginx!`的欢迎界面，证明安装并且启动成功啦~

#### 其他命令
* 停止 nginx 服务器：nginx -s stop
* 重新载入配置文件：nginx -s reload