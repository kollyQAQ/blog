[TOC]

## 安装rabbitMQ

```shell
run -d --hostname my-rabbit --name some-rabbit -p 15672:15672 -p 5672:5672 -p 5671:5671 -e RABBITMQ_DEFAULT_USER=root -e RABBITMQ_DEFAULT_PASS=root rabbitmq:3-management
```

```shell
# 后台登录 账号：root 密码：root
http://localhost:15672/
```

