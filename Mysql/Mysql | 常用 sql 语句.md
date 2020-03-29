[TOC]

#### 连接mysql

> mysql -h{host} -P{port} -u{user} -p'{password}'

```shell
mysql -h127.0.0.1 -P3306 -uroot -p'Root@123'
```

其中：host=1270.0.1或者localhost可省略 port=3306可省略 user=root可省略

```shell
mysql -p‘Root@123’
```

#### 查看mysql版本

```shell
[root@localhost ~]# mysql -V
```

或者

```sql
mysql> SELECT VERSION();
```

#### 创建数据库

```sql
CREATE DATABASE db_user;
CREATE DATABASE IF NOT EXISTS db_user;
CREATE DATABASE IF NOT EXISTS db_user CHARACTER SET 'utf8' COLLATE 'utf8_unicode_ci';
```

#### 查看数据库的创建语句

```sql
SHOW CREATE DATABASE db_user;
```

#### 创建表

> CRTATE TABLE [IF NOT EXISTS] TBNAME(col_name col_definition,...)

```sql
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
```

#### 查看表的创建语句

> SHOW CREATE TABLE TABLENAME;

```sql
SHOW CREATE TABLE t_user;
```

#### 重命名表

```sql
alter table t_bank_level RENAME TO t_credit_card_bank_category;
```

#### 设置主键自增值

```sql
ALTER TABLE test AUTO_INCREMENT = 1000;
```

#### 删除字段
```sql
alter table t_credit_card_bank_category drop column bank_level;
```

#### 增加字段

> ALTER TABLE table_name ADD [COLUMN] col_name column_definition  [ FIRST | AFTER col_name]

```sql
alter table t_bom_res_apply_modal_active add COLUMN credit_card_score int(4) DEFAULT -1  COMMENT '信用卡评分' after final_credit_risk_level;
```

#### 修改字段
```sql
alter table t_bank_level CHANGE COLUMN create_time create_time timestamp DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间'
```

#### 连表更新
```sql
update t_risk_rule_config a,t_risk_code b set b.dsc = a.rule_name where a.rule_no = b.`code` and a.rule_name <> b.dsc;
```

#### INSERT ON DUPLICATE KEY UPDATE

向数据库插入记录时，有时会有这种需求，当符合某种条件的数据存在时，去修改它，不存在时，则新增，也就是saveOrUpdate操作。这种控制可以放在业务层，也可以放在数据库层，大多数数据库都支持这种需求，如Oracle的merge语句，MySQL中的INSERT ... ON DUPLICATE KEY UPDATE语句。

```sql
INSERT INTO t_wb_lollive_hour_stat  (shopId,startTime,popupNum) VALUES (23702,'2016-11-11-11',120) ON DUPLICATE KEY UPDATE popupNum = popupNum  +120
```

对应的 Mybatis 的 xml 文件如下

```
<insert id="batchHandleLolLiveOpen" parameterType="List" >
   insert into t_wb_lollive_hour_stat  (shopId,agentId,startTime,popupNum) values
   <foreach collection="list" item="item" index= "index" separator =",">
      (#{item.shopId},#{item.agentId},#{item.startTime},#{item.popupNum})
   </foreach>
   ON DUPLICATE KEY UPDATE popupNum = popupNum  + VALUES(popupNum)
</insert>
```

### 根据日期筛选
```mysql
#昨天  
SELECT * FROM 表名 WHERE TO_DAYS( NOW( ) ) - TO_DAYS( 时间字段名) <= 1  
#今天  
SELECT * FROM 表名 WHERE TO_DAYS(时间字段名) = TO_DAYS(NOW());  
 
#7天  
SELECT * FROM 表名 where DATE_SUB(CURDATE(), INTERVAL 7 DAY) <= DATE(时间字段名)  
 
#近30天  
SELECT * FROM 表名 where DATE_SUB(CURDATE(), INTERVAL 30 DAY) <= DATE(时间字段名)  
 
#本月  
SELECT * FROM 表名 WHERE DATE_FORMAT( 时间字段名, '%Y%m' ) = DATE_FORMAT( CURDATE( ) , '%Y%m' )  
 
#上一月  
SELECT * FROM 表名 WHERE PERIOD_DIFF( DATE_FORMAT( NOW( ) , '%Y%m' ) , DATE_FORMAT( 时间字段名, '%Y%m' ) ) =1  
```

