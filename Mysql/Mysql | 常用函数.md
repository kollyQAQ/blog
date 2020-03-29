[TOC]

#### 字符串拼接 concat

```sql
UPDATE t_user SET name=concat('x',name)
```



#### 为空判断 IFNULL

```sql
SELECT IFNULL(SUM(popupNum),0) FROM t_wb_lollive_day_stat WHERE agentId = 'lgj' AND startTime LIKE CONCAT('2016-12-','%')
```



#### 字符串替换 replace

> REPLACE(str, from_str, to_str) 
>
> 在字符串 str 中所有出现的字符串 from_str 均被 to_str替换，然后返回这个字符串

```sql
UPDATE tb1 SET f1=REPLACE(f1, 'abc', 'def'); 
```



#### 字符串包含 find_in_set

> find_in_set(str1, str2)
>
> 返回str2中str1所在的位置索引

```
mysql > SELECT FIND_IN_SET('黄金','黄金,白银,白金')  AS pos 

​	-> 1
```



#### 日期格式化 DATE_FORMAT

```sql
select DATE_FORMAT(timeId,'%Y-%m-%d') as 'timeId', sum(timeCost) as 'timeCost' from t_wb_business_data where shopId = #{map.shopId} AND timeId **between** #{map.startDate} and #{map.endDate} group by timeId ORDER BY timeId
```



#### 四舍五入

**TRUNCATE()**：直接截取，不四舍五入，返回类型是数字型

**FORMAT()**： 会四舍五入，返回类型是字符串因为满3位会加一个逗号



**FLOOR(X)**

返回不大于X的最大整数值。注意返回值被变换为一个BIGINT！  

mysql> select FLOOR(1.23);        

​	-> 1

mysql> select FLOOR(-1.23);        

​	-> -2

**CEILING(X)**

返回不小于X的最小整数值。注意返回值被变换为一个BIGINT！ 

mysql> select CEILING(1.23);        

​	-> 2

mysql> select CEILING(-1.23);        

​	-> -1

**ROUND(X)**

返回参数X的四舍五入的一个整数。注意返回值被变换为一个BIGINT! 

mysql> select ROUND(-1.23);        

​	-> -1

mysql> select ROUND(-1.58);        

​	-> -2

mysql> select ROUND(1.58);        

​	-> 2 

**ROUND(X,D)**

返回参数X的四舍五入的有D为小数的一个数字。如果D为0，结果将没有小数点或小数部分。

mysql> select ROUND(1.298, 1);        

​	-> 1.3

mysql> select ROUND(1.298, 0);        

​	-> 1