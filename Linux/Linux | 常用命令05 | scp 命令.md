-r 复制的是文件夹
-P 端口（如果是22可以省略，注意P是大写）
-i 密钥文件

#### 从远程服务器下载文件
```shell
scp -r -P 60755 appadmin@10.104.68.253:/usr/local/jweb/jweb_wb_gstatus /usr/local/jweb/
```
#### 上传文件到远程服务器
```shell
scp /home/test.sh -P 60755 appadmin@182.254.228.66:/home.test.sh
```
#### 密钥访问
```shell
scp -P 60755 -i /root/.ssh/id.rsa /root/index.new.java appadmin@10.104.85.238:/home/appadmin/index.java
```