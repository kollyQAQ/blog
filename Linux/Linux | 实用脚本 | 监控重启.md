[TOC]

#### **crontab**

```shell
*/5 * * * * /home/appadmin/gmasterMonitor.sh >> /home/appadmin/restart.log

* * * * * sleep 10; /home/appadmin/gstatMonitor.sh >> /home/appadmin/restart.log
* * * * * sleep 30; /home/appadmin/gstatMonitor.sh >> /home/appadmin/restart.log
* * * * * sleep 50; /home/appadmin/gstatMonitor.sh >> /home/appadmin/restart.log
```

#### gmasterMonitor.sh

```shell
#!/bin/bash
process=`ps -ef | grep "gmaster" | awk '{print $8}'`;
function main(){
for pro in $process
do
if [ $pro == "/usr/bin/java" ]; then
echo `date "+%Y-%m-%d %H:%M:%S"` "gmaster is running..";
exit;
fi
done
restart;
}

function restart(){
echo `date "+%Y-%m-%d %H:%M:%S"` "gamster is not running restart gmaster...";
sh /usr/local/jweb/service.sh restart jweb_wb_gmaster; # 重启服务
}
```

