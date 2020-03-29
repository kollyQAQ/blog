方法是使用 `ssh-copy-id` 功能，原理是将本机的密钥复制到远程要连接的机器上，从而授权连接。



**(可选) 如果你的机器没有生成过 ssh 密钥则输入，如果已经存在，则忽略这步**

```
ssh-keygen
```

执行这个命令后，一路回车即可。会在 `~/.ssh` 目录下生成 `id_rsa` 和 `id_rsa.pub` 两个文件



**复制密钥到远程目的服务器**

```
ssh-copy-id -i root@50.100.11.10
```

按提示输入一次密码，`ssh-copy-id `就会自动将刚才生成的公钥 id_rsa.pub 追加到远程主机的`~/.ssh/authorized_keys ` 后面了，这样以后的 ssh 连接都不用输入密码了。



**配置 ssh config**

找到 `～/.ssh/config`，如果不存在，可以新建一个  `vim ~/.ssh/config`，输入

```
Host aliyun
  HostName 50.100.11.10
  User root
  Port 22
```



在 *iTerm2* 中输入 `ssh aliyun` ，回车，即可登录

