[TOC]

### **版本要求**

**需Mysql 5.6以上版本**

### **MySQL 脚本实现用例**

添加 `CreateTime` 设置默认时间 `CURRENT_TIMESTAMP` 

```sql
ALTER TABLE `table_name`
ADD COLUMN  `CreateTime` datetime NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间' ;
```

添加 `UpdateTime` 设置 默认时间 `CURRENT_TIMESTAMP`   设置更新时间为 `ON UPDATE CURRENT_TIMESTAMP` 

```sql
ALTER TABLE `table_name`
ADD COLUMN `UpdateTime` timestamp NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间' ;
```

#### **MySQL工具设置**

![](http://ww1.sinaimg.cn/large/006tNc79gy1g4ntses033j30ig04r74a.jpg)

![](http://ww3.sinaimg.cn/large/006tNc79gy1g4nts7evtpj30i1052jrg.jpg)