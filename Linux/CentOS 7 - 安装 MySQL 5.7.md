[TOC]

# CentOS 7 - 安装 MySQL 5.7

1. 添加 Mysql5.7 仓库

```
sudo rpm -ivh https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
```

2. 开始安装 Mysql5.7

```
sudo yum -y install mysql-community-server
```

3. 启动 Mysql

```shell
# 启动
sudo systemctl start mysqld

# 设置系统启动时自动启动
sudo systemctl enable mysqld

# 查看启动状态
sudo systemctl status mysqld
```

4. Mysql 的安全设置

CentOS 上的 root 默认密码可以在文件 /var/log/mysqld.log 找到，通过下面命令可以打印出来

```
cat /var/log/mysqld.log | grep -i 'temporary password'
```

执行下面命令进行安全设置，这个命令会进行设置 root 密码设置，移除匿名用户，禁止 root 用户远程连接等

```
mysql_secure_installation
```

5. 设置数据库编码为 utf8

```shell
# 打开配置文件
sudo vim /etc/my.cnf

# 在 [mysqld]，[client]，[mysql] 节点下添加编码设置
[client]
default-character-set=utf8

[mysql]
default-character-set=utf8

[mysqld]
collation-server = utf8_unicode_ci
init-connect='SET NAMES utf8'
character-set-server = utf8

# 重启 Mysql 即可
sudo systemctl restart mysqld
```

6. 本地登录

```shell
mysql -uroot -p'123456'
```

7. 允许远程连接

   1. 查看系统用户

      use mysql;

      select user,host from user;

   2. 支持 root 用户允许远程连接 mysql 数据库

      grant all privileges on *.* to 'root'@'%' identified by '123456' with grant option;

      flush privileges;

8. 查看阿里云安全组规则

