[TOC]

## 数据库管理

#### 连接 mysql

```shell
mysql -h$ip -P$port -u$user -p$pwd
```

其中：host=1270.0.1 **或者** localhost 可省略; port=3306 可省略; user=root 可省略

#### 显示用户正在运行的线程

```mysql
show processlist;
```

需要注意的是，除了 root 用户能看到所有正在运行的线程外，其他用户都只能看到自己正在运行的线程，看不到其它用户正在运行的线程。除非单独个这个用户赋予了 PROCESS 权限。

#### 查看MySQL服务器配置信息 

```mysql
SHOW VARIABLES；
SHOW VARIABLES like 'character%';
```

#### 查看MySQL服务器运行的各种状态值 

mysql 系统参数分为session和global 之分， session只当前连接生效，global 全局连接生效

```mysql
show global status;
```

#### 慢查询

```mysql
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

```mysql
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

**设置的最大连接数是500，而响应的连接数是498   max_used_connections / max_connections * 100% = 99.6% （理想值 ≈ 85%）** 

#### server 接受的数据包大小限制

```mysql
mysql> show variables like 'max_allowed_packet';  
+----------------------+---------+  
| Variable_name        | Value   |  
+----------------------+---------+  
| max_allowed_packet   | 1048576 |  
+----------------------+---------+  
```

有时候大的插入和更新会受 max_allowed_packet 参数限制，导致写入或者更新失败。以上说明目前的配置是：1M。

修改方法：在 mysql 命令行中运行` set global max_allowed_packet = 20 * 1024 * 1024` 然后退出命令行，重启mysql服务，再进入。（这里设置的是20M大小）  

#### 导出 sql 结果到文件

```shell
mysql -h127.0.0.1 -uroot -P3306 -e "use test; select * from tb_test;" > /tmp/rs.txt
```



## DDL

#### 创建数据库

```mysql
CREATE DATABASE db_user;
CREATE DATABASE IF NOT EXISTS db_user;
CREATE DATABASE IF NOT EXISTS db_user CHARACTER SET 'utf8' COLLATE 'utf8_unicode_ci';

SHOW CREATE DATABASE db_user; -- 查看数据库的创建语句
```

#### 创建表

```mysql
CREATE TABLE t_user (
  id INT(10) UNSIGNED NOT NULL AUTO_INCREMENT,
  name VARCHAR(32) NOT NULL,
  birthday VARCHAR(10) NOT NULL,
  cardNo VARCHAR(64) NOT NULL,
  createTime DATETIME DEFAULT CURRENT_TIMESTAMP,
  updateTime DATETIME DEFAULT NULL ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (id),
  INDEX(cardNo),
  UNIQUE KEY COMB_UNIQUE (name,birthday),
) ENGINE=INNODB DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;

SHOW CREATE TABLE t_user; -- 查看表的创建语句
```

#### 重命名表

```mysql
ALTER TABLE users RENAME TO t_user;
```

#### 设置主键自增值

```mysql
ALTER TABLE t_user AUTO_INCREMENT = 1000;
```

#### 删除字段

```mysql
ALTER TABLE t_user DROP COLUMN name;
```

#### 增加字段

```mysql
ALTER TABLE t_user ADD COLUMN age int(4) DEFAULT -1  COMMENT '年龄' after sex;
```



#### 修改字段

```mysql
ALTER TABLE t_user CHANGE COLUMN create_time create_time timestamp DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间'

ALTER TABLE t_user MODIFY COLUMN `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间';
```



## DML

#### 联表更新

```mysql
update t_risk_rule_config a,t_risk_code b set b.dsc = a.rule_name where a.rule_no = b.`code` and a.rule_name <> b.dsc;
```

#### INSERT ON DUPLICATE KEY UPDATE

向数据库插入记录时，有时会有这种需求，当符合某种条件的数据存在时，去修改它，不存在时，则新增，也就是saveOrUpdate操作。这种控制可以放在业务层，也可以放在数据库层，大多数数据库都支持这种需求，如Oracle的merge语句，MySQL中的INSERT ... ON DUPLICATE KEY UPDATE语句。

```mysql
INSERT INTO t_user(idcard,name,age) VALUES ('420984199304112321','allen',25) ON DUPLICATE KEY UPDATE age = 25
```



## Mysql 函数

#### 日期函数

- DATE_SUB() 从日期减去指定的时间间隔

```mysql
#update_time 是 t_user 表中的 datetime 类型的字段名

#昨天  
SELECT * FROM t_user WHERE TO_DAYS(NOW()) - TO_DAYS(update_time) <= 1  

#今天  
SELECT * FROM t_user WHERE TO_DAYS(update_time) = TO_DAYS(NOW());  
 
#近1小时  
SELECT * FROM t_user where update_time > DATE_SUB(CURDATE(), INTERVAL 1 HOUT)
 
#近30天  
SELECT * FROM t_user where update_time > DATE_SUB(CURDATE(), INTERVAL 30 DAY)
 
#本月  
SELECT * FROM t_user WHERE DATE_FORMAT(update_time,'%Y%m') = DATE_FORMAT(CURDATE(), '%Y%m')  
 
#上一月  
SELECT * FROM t_user WHERE PERIOD_DIFF(DATE_FORMAT(NOW(),'%Y%m'), DATE_FORMAT(update_time,'%Y%m')) =1  

# 日期格式化
select DATE_FORMAT(timeId,'%Y-%m-%d') as 'timeId', sum(timeCost) as 'timeCost' from t_wb_business_data where shopId = ${map.shopId} AND timeId between ${map.startDate} and ${map.endDate} group by timeId ORDER BY timeId
```

#### 字符串函数

**FIND_IN_SET**(str1, str2)  返回 str2 中 str1 所在的位置索引

```mysql
mysql > SELECT FIND_IN_SET('黄金','黄金,白银,白金') AS pos 
​	-> 1
```

**REPLACE**(str, from_str, to_str)  在字符串 str 中所有出现的字符串 from_str 均被 to_str替换，然后返回这个字符串

```mysql
UPDATE tb1 SET f1 = REPLACE(f1, 'abc', 'def'); 
```

**concat** 字符串拼接 

```mysql
UPDATE t_user SET name=concat('x',name)
```

#### IFNULL 为空判断 

```mysql
SELECT IFNULL(SUM(score),0) FROM t_user;
```

#### when、case

```mysql
select 
    id,
    (case type when '1' then 'text' when '2' then 'template' end) as type_text
from taxon_attr_value;
```

#### 四舍五入

**TRUNCATE()**：直接截取，不四舍五入，返回类型是数字型

**FORMAT()**： 会四舍五入，返回类型是字符串因为满3位会加一个逗号

**FLOOR(X)**：返回不大于X的最大整数值。注意返回值被变换为一个BIGINT！  

**CEILING(X)**：返回不小于X的最小整数值。注意返回值被变换为一个BIGINT！ 

**ROUND(X)**：返回参数X的四舍五入的一个整数。注意返回值被变换为一个BIGINT! 