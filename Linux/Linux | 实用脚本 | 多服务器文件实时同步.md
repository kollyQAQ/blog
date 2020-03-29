[TOC]
### 需求背景
公司有八台nginx服务器用来做静态文件服务器和http请求转发，所以当需要修改nginx配置文件或者部署静态文件的时候，需要分别修改八台机器上的文件，费时费力。

### 最终目的
对其中某一个机器的某些文件夹做监控，只要其中的内容有发生修改，其余机器马上同步这些修改到自己的机器上

### 实现工具
rsync+inotify

### 实现步骤


1. 主服务器A安装rsync

   ```
   $ yum install rsync
   ```

2. 写配置文件，需要自己建文件夹和文件

   ```
   $ cd /etc/rsync.d
   $ touch rsyncd.secret
   ```

   rsyncd.secret文件内容如下：

   ```
   admin123   //后面其他服务器要同步此服务器文件需要用到的密码
   ```

3. 安装inotify工具

   ```
   yum install inotify-tools
   ```

4. 编写实现同步的脚本,这个脚本的作用就是通过inotify监控本服务器某个文件目录的变化，进而触发其他host进行rsync同步操作

   ```
   $ cd /etc/inotify
   $ touch inotify.sh
   ```

   inotify.sh文件内容如下：

   ```
   #!/bin/bash
       host="10.104.xx.xxx 10.104.xx.xxx 10.104.xx.xxx"
       src=/data/web/html
       dst=htmlFile
       user=root
       pass_file=/etc/rsync.d/rsyncd.secrets

       /usr/bin/inotifywait -mrq --timefmt '%d/%m/%y %H:%M' --format '%T %w%f' -e  modify,delete,create,attrib $src | while read files
            do
            for hostip in $host
                do
                /usr/bin/rsync -vzrtopg --delete --progress --password-file=$pass_file $src $user@$hostip::$dst    
                done
            echo "${files} was rsynced" >>/tmp/rsync.log 2>&1
            done
   ```

5. 同步服务器B、C、D安装rsync

   ```
   $ yum install rsync
   ```

6. 写配置文件，需要自己建文件夹和文件

   ```
   $ cd /etc/rsync.d
   $ touch rsyncd.conf rsyncd.secret rsyncd.motd
   ```

   rsyncd.conf是配置文件，内容如下

   ```
   uid = root
   gid = root
   use chroot = yes

   max connections = 100

   pid file = /var/run/rsyncd.pid
   log file = /var/log/rsyncd.log
   motd file = /etc/rsync.d/rsyncd.motd

   [htmlFile]
   path = /data 
   comment = Rsync html file
   uid = root
   gid = root
   auth users = root
   secrets file = /etc/rsync.d/rsyncd.secrets
   read only = yes
   list = yes
   ignore errors
   read only = no
   write only = no
   hosts allow = xx.xxx.xx.xx (主服务器A的host)
   ```

   rsyncd.secret是密码文件，内容如下

   ```
   root:admin123
   ```

   rsyncd.motd是服务器描述文件，内容如下

   ```
   ++++++++++++++++++++++++++++++++++++
   This is Server B, Welcome！
   +++++++++++++++++++++++++++++++++++++
   ```

7. 将rsyncd.secrets这个密码文件的文件属性设为root拥有, 且权限要设为600, 否则无法备份成功

   ```
   $ chmod 600 rsyncd.secrets
   ```

8. 启动rsync服务器(daemon参数方式，是让rsync以服务器模式运行)

   ```
   /usr/bin/rsync --daemon  --config=/etc/rsync.d/rsyncd.conf 
   ```

9. 设置开机启动rsync服务

   ```
   echo  “/usr/bin/rsync --daemon  --config=/etc/rsync.d/rsyncd.conf >/dev/null &”>>/etc/rc.local
   ```