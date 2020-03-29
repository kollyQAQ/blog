[TOC]

### 全局设置

```
#运行用户
user nginx;  

#启动进程数,通常设置成和cpu的数量相等
worker_processes  auto;

#全局错误日志及PID文件
error_log  /var/log/nginx/error.log;
pid        /var/run/nginx.pid;

#工作模式及连接数上限
events {
    use   epoll;   #epoll是多路复用IO(I/O Multiplexing)中的一种方式,可以大大提高nginx的性能
    worker_connections  1024;#单个后台worker process进程的最大并发链接数
}

```

### http服务器设置

设定http服务器，利用它的反向代理功能提供负载均衡支持

```
http {
    #日志输出内容与格式
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    #日志输出目录
    access_log  /var/log/nginx/access.log  main;

    #sendfile 指令指定 nginx 是否调用 sendfile 函数（zero copy 方式）来输出文件，对于普通应用，必须设为 on,如果用来进行下载等应用磁盘IO重负载应用，可设置为 off，以平衡磁盘与网络I/O处理速度，降低系统的uptime.
    sendfile            on;
    
    tcp_nopush          on;
    tcp_nodelay         on;
    
    #连接超时时间
    keepalive_timeout   65;
    
    types_hash_max_size 2048;

    #设定mime类型,类型由mime.type文件定义
    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;
    
    include /etc/nginx/conf.d/*.conf;
    
    upstream {
    	#见upstream配置
    }
    
    server {
    	#见server配置
    }
}
```

### upstream配置

```
upstream mysvr {
    #设定负载均衡的服务器列表
    #weigth参数表示权值，权值越高被分配到的几率越大,如果权重都相同，weight可以都不设置
    server 192.168.8.1:80 weight=5;
    server 192.168.8.2:80 weight=1;
    server 192.168.8.3:80 weight=6;
}
```



### server配置

```
server {
    
    listen         80;			#侦听80端口
    listen         443 ssl;		 #侦听443端口
    server_name    www.xxx.com;	  #定义使用www.xx.com访问
    
    #https证书
    ssl_certificate /etc/ssl/certs/www.xxx.com_bundle.crt;
    ssl_certificate_key /etc/ssl/certs/www.xxx.com.key;

    error_page 404 /html/404.html;
    error_page 500 502 503 504 /html/50x.html;
    
    #设置主页
    location = / {
        root /data/static/html;
        index index.html;
    }
    
    #静态文件，nginx自己处理
    location ^~ /(html|js|css|jpg|png|txt|json)/ {
        root /data/static/;
        #过期30天，静态文件不怎么更新，过期可以设大一点，如果频繁更新，则可以设置得小一点。
        expires 30d;
        #expires 15s;
    }
    
    #api转发到后端服务器
    location /  {
        proxy_pass    http://mysvr/;   #mysvr是上面配置的upstream
        
        #以下是一些反向代理的配置可删除.
        #后端的Web服务器可以通过X-Forwarded-For获取用户真实IP
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        client_max_body_size 10m;    #允许客户端请求的最大单文件字节数
        client_body_buffer_size 128k;    #缓冲区代理缓冲用户端请求的最大字节数，
        proxy_connect_timeout 90;    #nginx跟后端服务器连接超时时间(代理连接超时)
        proxy_send_timeout 90;    #后端服务器数据回传时间(代理发送超时)
        proxy_read_timeout 90;    #连接成功后，后端服务器响应时间(代理接收超时)
        proxy_buffer_size 4k;    #设置代理服务器（nginx）保存用户头信息的缓冲区大小
        proxy_buffers 4 32k;  #proxy_buffers缓冲区，网页平均在32k以下的话，这样设置
        proxy_busy_buffers_size 64k;  #高负荷下缓冲大小（proxy_buffers*2）
        proxy_temp_file_write_size 64k;  #设定缓存文件夹大小，大于这个值，将从upstream服务器传
        proxy_redirect off;
        proxy_intercept_errors on;
    }

}
```



### location配置语法

语法规则： `location [=|~|~*|^~] /uri/ { … }`

- `=` 开头表示精确匹配
- `^~` 开头表示uri以某个常规字符串开头，理解为匹配 url路径即可。nginx不对url做编码，因此请求为/static/20%/aa，可以被规则^~ /static/ /aa匹配到（注意是空格）。
- `~` 开头表示区分大小写的正则匹配
- `~*` 开头表示不区分大小写的正则匹配
- `!~`和`!~*`分别为区分大小写不匹配及不区分大小写不匹配 的正则
- `/` 通用匹配，任何请求都会匹配到。

多个location配置的情况下匹配顺序为（参考资料而来，还未实际验证，试试就知道了，不必拘泥，仅供参考）：

首先匹配 =，其次匹配^~, 其次是按文件中顺序的正则匹配，最后是交给 / 通用匹配。当有匹配成功时候，停止匹配，按当前匹配规则处理请求。

例子，有如下匹配规则：

```
location = / {
   #规则A
}
location = /login {
   #规则B
}
location ^~ /static/ {
   #规则C
}
location ~ \.(gif|jpg|png|js|css)$ {
   #规则D
}
location ~* \.png$ {
   #规则E
}
location !~ \.xhtml$ {
   #规则F
}
location !~* \.xhtml$ {
   #规则G
}
location / {
   #规则H
}
```
那么产生的效果如下：

访问根目录/， 比如http://localhost/ 将匹配规则A

访问 http://localhost/login 将匹配规则B，http://localhost/register 则匹配规则H

访问 http://localhost/static/a.html 将匹配规则C

访问 http://localhost/a.gif, http://localhost/b.jpg 将匹配规则D和规则E，但是规则D顺序优先，规则E不起作用， 而 http://localhost/static/c.png 则优先匹配到 规则C

访问 http://localhost/a.PNG 则匹配规则E， 而不会匹配规则D，因为规则E不区分大小写。

访问 http://localhost/a.xhtml 不会匹配规则F和规则G，http://localhost/a.XHTML不会匹配规则G，因为不区分大小写。规则F，规则G属于排除法，符合匹配规则但是不会匹配到，所以想想看实际应用中哪里会用到。

访问 http://localhost/category/id/1111 则最终匹配到规则H，因为以上规则都不匹配，这个时候应该是nginx转发请求给后端应用服务器，比如FastCGI（php），tomcat（jsp），nginx作为方向代理服务器存在。

所以实际使用中，个人觉得至少有三个匹配规则定义，如下：

```
#直接匹配网站根，通过域名访问网站首页比较频繁，使用这个会加速处理，官网如是说。
#这里是直接转发给后端应用服务器了，也可以是一个静态首页
# 第一个必选规则
location = / {
    proxy_pass http://tomcat:8080/index
}

# 第二个必选规则是处理静态文件请求，这是nginx作为http服务器的强项
# 有两种配置模式，目录匹配或后缀匹配,任选其一或搭配使用
location ^~ /static/ {
    root /webroot/static/;
}
location ~* \.(gif|jpg|jpeg|png|css|js|ico)$ {
    root /webroot/res/;
}

#第三个规则就是通用规则，用来转发动态请求到后端应用服务器
#非静态文件请求就默认是动态请求，自己根据实际把握
#毕竟目前的一些框架的流行，带.php,.jsp后缀的情况很少了

location / {
    proxy_pass http://tomcat:8080/
}
```

**nginx的其他配置信息介绍**

三、ReWrite语法

`last` – 基本上都用这个Flag。
`break` – 中止Rewirte，不在继续匹配
`redirect` – 返回临时重定向的HTTP状态302
`permanent` – 返回永久重定向的HTTP状态301

1、下面是可以用来判断的表达式：

`-f`和`!-f`用来判断是否存在文件
`-d`和`!-d`用来判断是否存在目录
`-e`和`!-e`用来判断是否存在文件或目录
`-x`和`!-x`用来判断文件是否可执行

2、下面是可以用作判断的全局变量

例：http://localhost:88/test1/test2/test.php

```
$host：localhost
$server_port：88
$request_uri：http://localhost:88/test1/test2/test.php
$document_uri：/test1/test2/test.php
$document_root：D:\nginx/html
$request_filename：D:\nginx/html/test1/test2/test.php
```

四、Redirect语法

```
server {
    listen 80;
    server_name start.igrow.cn;
    index index.html index.php;
    root html;
    if ($http_host !~ "^star\.igrow\.cn$" {
        rewrite ^(.*) http://star.igrow.cn$1 redirect;
    }
}
```

五、防盗链

```
location ~* \.(gif|jpg|swf)$ {
    valid_referers none blocked start.igrow.cn sta.igrow.cn;
    if ($invalid_referer) {
        rewrite ^/ http://$host/logo.png;
    }
}

```

六、根据文件类型设置过期时间

```
location ~* \.(js|css|jpg|jpeg|gif|png|swf)$ {
    if (-f $request_filename) {
        expires 1h;
        break;
    }
}

```

七、禁止访问某个目录

```
location ~* \.(txt|doc)${
root /data/www/wwwroot/linuxtone/test;
deny all;
}

```

附：一些可用的全局变量

```
$args
$content_length
$content_type
$document_root
$document_uri
$host
$http_user_agent
$http_cookie
$limit_rate
$request_body_file
$request_method
$remote_addr
$remote_port
$remote_user
$request_filename
$request_uri
$query
```
#### 常用示例

#### 根据请求参数过滤请求(\$args,\$arg_xxx)

```
location = /rushBuy {
        proxy_pass    http://127.0.0.1:9000/;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Host $host;
        proxy_intercept_errors on;
        if ($arg_timestamp ~* '\d*[0|2|4|6|8]$'){
            add_header Content-Type 'application/json;charset=UTF-8';
            return 200 '{"errCode": "-1","msg": "抢购失败"}';
        }
}
```

上述配置会拦截所有请求为`url：/rushBuy?userId=xxx&goodsId=xxx&timestamp=xxx`中timestamp为偶数的请求直接返回固定值，这样可以限制一半的流量进入后台服务器，在秒杀场景限流使用。

#### 根据请求uri过滤请求(\$request_uri)

```
location ^~ /client/stat {
		proxy_pass	http:127.0.0.1:9000/;
		proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Host $host;
        proxy_intercept_errors on;
        if ($request_uri ~* '/gmreport'){
            return 200;
        }
}
```