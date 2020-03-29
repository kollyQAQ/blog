[TOC]

### mysqldump 的使用

#### **导出整个数据库**

```
mysqldump -u 用户名 -p 数据库名 > 导出的文件名
mysqldump -u linuxde -p smgp_apps_linuxde > linuxde.sql
```

#### **导出一个表**

```
mysqldump -u 用户名 -p 数据库名 表名> 导出的文件名
mysqldump -u linuxde -p smgp_apps_linuxde users > linuxde_users.sql
```

#### **导出一个数据库结构**

```
mysqldump -u linuxde -p -d --add-drop-table smgp_apps_linuxde > linuxde_db.sql
```

`-d`没有数据，`--add-drop-tabl`e每个create语句之前增加一个`drop table`

### 导出指定列的insert语句

```shell
mysql -h127.0.0.1 -uroot -proot123 -e "SELECT CONCAT('insert into t_info(id,biz_name) values(',id,','',biz_name, '');') FROM test.t_risk_count_config where is_enable = 1" > /root/test.sql
```

```sql
SELECT CONCAT('insert into t_audit(loan_no,status,audit_actor_id,audit_actor,reject_reason,create_time) values(''',loan_no,''',',
status,',0,''',status_change_by,''',','''',status_remarks,'''',',''',status_change_time,''');') 
FROM t_status_change_record where status_change_by not in ('','risk');
```