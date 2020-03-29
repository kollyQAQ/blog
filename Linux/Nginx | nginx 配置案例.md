#### 这是一个生产环境的案例，用到了 location 的正则匹配和 rewrite 语法，实现根据请求 url 的区别分发到不同的机器

```
server {
    listen         8088;
    server_name    localhost;

    charset utf-8,gbk;
    default_type application/octet-stream;

    location = /favicon.ico {
        log_not_found off;
    }

    types {
       text/plain log;
       text/html html htm shtml;
    }

    location ~ (1|3|5|7|9)\.(html|log)$ {
        rewrite ^/(.*)$ http://rusheye2.yunjiaplus.com/$1 permanent;
    }

    location / {
        charset utf-8,gbk;
        root /data/www/caselog;
        add_header Access-Control-Allow-Origin *;
        add_header 'Access-Control-Allow-Credentials' 'true';
        add_header 'Access-Control-Allow-Methods' 'GET,POST';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $https;
        add_header Access-Control-Allow-Origin *;
      }
}
```