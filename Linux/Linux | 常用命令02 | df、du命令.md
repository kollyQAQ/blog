> 当磁盘大小超过标准时会有报警提示，这时如果掌握df和du命令是非常明智的选择。
>
> df可以查看一级文件夹大小、使用比例、档案系统及其挂入点，但对文件却无能为力。
>
> du可以查看文件及文件夹的大小。
>
> 两者配合使用，非常有效。比如用df查看哪个一级目录过大，然后用df查看文件夹或文件的大小，如此便可迅速确定症结。




做后台开发经常遇到磁盘占满的情况，毕竟不断有日志在生成，不断有文件在备份，如果长时间不管的话，总有一天会占用满。

题主有一天在发布java项目的时候，就发现提示

```
No space left on device
```

因为题主的发布脚本中有一段逻辑是把之前的jar包先备份到/data/backup目录，执行之一步的时候发现磁盘空间不足，于是报错，版本发布失败。

遇到这种问题，解决思路很简单嘛，磁盘满了删除没用的东西咯，没错，但是怎样能最快找到那些占用磁盘很大又已经没用的文件或者文件夹呢，这就是今天的课题了。**利用linux的df和du命令查看文件和目录的内存占用**

### 第一步 使用df命令

```
[appadmin@VM_95_213_centos ~]$ df -h
Filesystem           	  Size      Used Available Use% Mounted on
/dev/vda1              8254240   3710232   4124716  48% /
/dev/vdb1             30962748  21870140   7519728  75% /usr/local
/dev/vdb2             20641788   1672712  17920480   9% /home
/dev/vdb4            138303008 131277884         0 100% /data
```

**df**命令可以显示目前所有文件系统的可用空间及使用情形

* 参数`-h`表示使用「Human-readable」的输出，也就是在档案系统大小使用 GB、MB 等易读的格式。
* 上面的命令输出的第一个字段`Filesystem`及最后一个字段`Mounted on`分别是档案系统及其挂入点。我们可以看到`/dev/vda1` 这个分割区被挂在根目录下。
* 接下来的四个字段 `Size`、`Used`、`Available`、及` Use% `分别是该分割区的容量、已使用的大小、剩下的大小、及使用的百分比。

可以看到`/data`这个目录磁盘已经到了100%，所以在想要往/data/backup目录写文件时系统发出`No space left on device`的警告。

那么接下来的目的很简单，清理/data目录，找出占用磁盘空间大而且没用的文件或者目录，delete！

###  第二步 使用du命令

1. 进入`data`目录

   ```
   [appadmin@VM_95_213_centos jweblog]$ cd /data
   ```

2. 执行du命令

   ```
   [appadmin@VM_95_213_centos data]$ du --max-depth=1 -h
   15G	./jweb_static
   108G	./jweblog
   8.2M	./news
   1.3G	./japplog
   16K	./lost+found
   7.7M	./backup
   595M	./varlog
   1.1G	./pyweb_log
   125G	.
   ```

   **du**命令可以查询文件或文件夹的磁盘使用空间

   参数`-h`表示使用「Human-readable」的输出，也就是在档案系统大小使用 GB、MB 等易读的格式

   参数`--max-depth`指定深入目录的层数,这是个极为有用的参数,如果当前目录下文件和文件夹很多，使用不带参数du的命令，会循环列出所有文件和文件夹所使用的空间。这对查看究竟是那个地方过大是不利的

   可以看到的是`jweblog`这个目录有108G个G，沃德天，赶紧进去看看~

3. 进入`jweblog`目录

   ```
   [appadmin@VM_95_213_centos data]$ cd jweblog/
   ```

4. 继续执行du命令

   ```
   [appadmin@VM_95_213_centos jweblog]$ du --max-depth=1 -h
   32K	./jweb_mbox_acs
   4.0K	./jweb_cz_gmaster
   732K	./jweb_coomix_scibo
   15G	./jweb_game_farm
   23G	./jweb_open_manager
   404M	./jweb_mbox_app
   8.2M	./jweb_ak_backend
   14M	./jweb_wifishare_manager
   23M	./jweb_yzj_open
   4.3G	./jweb_akgame_third
   4.2G	./jweb_mpos_wxuser_auth_ak
   654M	./jweb_mbox_wifi_svr
   608K	./jweb_qqy_wx_auth
   970M	./jweb_wb_gmwx
   8.3G	./jweb_mpos_wxuser_auth
   4.2G	./jweb_bc_bottle
   160K	./jweb_mbox_wifi_wx_svr
   132K	./jweb_wb_gmaster
   1.1G	./jweb_mpos_kmkuser_auth
   4.0K	./default
   35G	./jweb_promotion_manager
   12G	./jweb_mpos_wxuser_auth_cn
   108G	.
   ```

到此应该就可以破案了，这里是各个工程的日志，看看哪些日志没用的删掉就可以啦~