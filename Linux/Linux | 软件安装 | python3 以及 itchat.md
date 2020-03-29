[TOC]

#### **下载 & 解压**

- cd /usr/local 
- wget https://www.python.org/ftp/python/3.6.3/Python-3.6.3.tgz 下载安装包 
- tar zxvf Python-3.6.3.tgz 解压安装包 

#### **编译 & 安装**

- cd Python-3.6.3 转到该安装包目录下 
- ./configure --prefix=/usr/local/python3.6 这一步极其重要，对安装进行配置，并指定安装路径，安装路径不指定的话不利于后面的系统管理 
- make 编译 
- make install 安装（可以合并为 make & make install） 

#### **建立软连接**

- ln -s /usr/local/python3.6/bin/python3.6 /usr/bin/python3 
- ln -s /usr/local/python3.6/bin/pip3 /usr/bin/pip3

#### **将 python3 加入 PATH 环境变量**

```
vim ~/.bash_profile 
```

在 export PATH 前增加一行 PATH=$PATH:/usr/local/python3.6/bin 

:wq 保存 

#### **pip3 安装**

https://www.cnblogs.com/wenchengxiaopenyou/p/5709218.html

#### **测试是否安装成功**

```shell
[root@localhost bin]# python3 -V 
Python 3.6.3 

[root@localhost bin]# pip3 -V 
pip 9.0.1 from /usr/local/python3.6/lib/python3.6/site-packages (python 3.6) 
```

#### **安装 itchat 相关依赖**

```
pip3 install itchat  
pip3 install apscheduler 
pip3 install beautifulsoup4 
pip3 install lxml 
pip3 install mysql-connector-python 
pip3 install sqlalchemy 
```

