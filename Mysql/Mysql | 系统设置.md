#### 查看MySQL服务器配置信息 

```mysql
mysql> show variables;
```



#### 查看MySQL服务器运行的各种状态值 

```mysql
mysql> show global status;  
```



mysql 系统参数分为session和global 之分， session只当前连接生效，global 全局连接生效



#### 慢查询

```
mysql> show variables like '%slow%';  
+------------------+-------+  
| Variable_name    | Value |  
+------------------+-------+  
| log_slow_queries | OFF   |  
| slow_launch_time | 2     |  
+------------------+-------+  

mysql> show global status like '%slow%';  
+---------------------+-------+  
| Variable_name       | Value |  
+---------------------+-------+  
| Slow_launch_threads | 0     |  
| Slow_queries        | 279   |  
+---------------------+-------+ 
```
**配置中关闭了记录慢查询（最好是打开，方便优化），超过2秒即为慢查询，一共有279条慢查询**



#### 连接数

```
mysql> show variables like 'max_connections';  
+-----------------+-------+  
| Variable_name   | Value |  
+-----------------+-------+  
| max_connections | 500   |  
+-----------------+-------+  
  
mysql> show global status like 'max_used_connections';  
+----------------------+-------+  
| Variable_name        | Value |  
+----------------------+-------+  
| Max_used_connections | 498   |  
+----------------------+-------+  
```

**设置的最大连接数是500，而响应的连接数是498   max_used_connections / max_connections * 100% = 99.6% （理想值 ≈ 85%）** 



#### server接受的数据包大小限制

```
mysql> show variables like 'max_allowed_packet';  
+----------------------+---------+  
| Variable_name        | Value   |  
+----------------------+---------+  
| max_allowed_packet   | 1048576 |  
+----------------------+---------+  
```

有时候大的插入和更新会受max_allowed_packet 参数限制，导致写入或者更新失败。以上说明目前的配置是：1M。

> 修改方法 :在mysql 命令行中运行  set global max_allowed_packet = 2\*1024\*1024\*10然后退出命令行，重启mysql服务，再进入。（这里设置的是20M大小）  
> show variables like '%max_allowed_packet%'; 查看下max_allowed_packet是否编辑成功
>
> 注意：该值设置过小将导致单个记录超过限制后写入数据库失败，且后续记录写入也将失败