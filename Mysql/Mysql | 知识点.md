[TOC]

#### and or not先后顺序问题

SQL执行这类条件判断时是有先后顺序的，具体顺序如下： （）、not、and、or 

最优先执行的是()内的判断条件，然后到not，再到and，最后才判断or



#### 对GROUBY得出的结果集进行筛选用HAVING

```sql
-- 查询登录次数超过 10 次的用户
SELECT user_id,COUNT(*) AS sum FROM t_user_login_log GROUP BY user_id Having sum > 10;
```

