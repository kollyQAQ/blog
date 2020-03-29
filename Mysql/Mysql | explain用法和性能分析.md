[TOC]

### explain的用法

```sql
explain select * from t_risk_rule_config where rule_no like 'T%';
```

结果如下

![](http://ww3.sinaimg.cn/large/006tNc79gy1g4nuvifjm4j30op01cdfr.jpg)

### EXPLAIN结果各列的解释

| 列名          | 含义                                                         |
| ------------- | ------------------------------------------------------------ |
| select_type   | select查询的类型，主要是区别普通查询和联合查询、子查询之类的复杂查询，它有六种情况 |
| table         | 显示这一行的数据是关于哪张表的                               |
| type          | 这是重要的列，显示连接使用了何种类型。从坏到好的连接类型为ALL < index < range < index_merge < fulltext < ref < eq_ref < const < system |
| possible_keys | 显示可能应用在这张表中的索引。如果为空，没有可能的索引。可以为相关的域从WHERE语句中选择一个合适的语句 |
| key           | 实际使用的索引。如果为NULL，则没有使用索引。很少的情况下，MYSQL会选择优化不足的索引。这种情况下，可以在SELECT语句中使用USE INDEX（indexname）来强制使用一个索引或者用IGNORE INDEX（indexname）来强制MYSQL忽略索引 |
| key_len       | 使用的索引的长度。在不损失精确性的情况下，长度越短越好       |
| ref           | 显示索引的哪一列被使用了，如果可能的话，是一个常数           |
| rows          | MYSQL认为必须检查的用来返回请求数据的行数                    |
| Extra         | 关于MYSQL如何解析查询的额外信息                              |

#### **select_type**字段各个值的解释

select查询的类型，主要是区别普通查询和联合查询、子查询之类的复杂查询，它有六种情况

| 可能值       | 解释                                                         |
| ------------ | ------------------------------------------------------------ |
| SIMPLE       | 简单的select查询，查询中不包含子查询或者UNION                |
| PRIMARY      | 查询中包含任何复杂的子部分，最外层查询则被标记为它           |
| SUBQUERY     | 在SELECT或WHERE列表中包含了子查询                            |
| DERIVED      | 在FROM列表中包含的子查询被标记为DERIVED（衍生），MySQL会递归执行这些子查询，把结果放在临时表里 |
| UNION        | 若第二个SELECT出现在UNION之后，则被标记为UNION；若UNION包含在FROM子句的子查询中，外层的SELECT将被标记为：DERIVED |
| UNION RESULT | 从UNION表获取结果的SELECT                                    |

#### **type**字段各个值的解释

显示的是访问类型，是较为重要的一个指标，结果值坏到好的连接类型为`ALL < index < range < index_merge < fulltext < ref < eq_ref < const < system`，除了all之外，其他的type都可以使用到索引，除了index_merge之外，其他的type只可以用到一个索引

一般来说，得保证查询至少达到`range`级别，最好能达到`ref`。

| 可能值      | 解释                                                         |
| ----------- | ------------------------------------------------------------ |
| system      | 表中只有一行数据或者是空表，且只能用于myisam和memory表。如果是Innodb引擎表，type列在这个情况通常都是all或者index |
| const       | 使用唯一索引或者主键，返回记录一定是1行记录的等值where条件时，通常type是const。其他数据库也叫做唯一索引扫描 |
| eq_ref      | 出现在要连接多个表的查询计划中，驱动表只返回一行数据，且这行数据是第二个表的主键或者唯一索引，且必须为not null，唯一索引和主键是多列时，只有所有的列都用作比较时才会出现eq_ref |
| ref         | 不像eq_ref那样要求连接顺序，也没有主键和唯一索引的要求，只要使用相等条件检索时就可能出现，常见与辅助索引的等值查找。或者多列主键、唯一索引中，使用第一个列之外的列作为等值查找也会出现，总之，返回数据不唯一的等值查找就可能出现 |
| fulltext    | 全文索引检索，要注意，全文索引的优先级很高，若全文索引和普通索引同时存在时，mysql不管代价，优先选择使用全文索引 |
| index_merge | 表示查询使用了两个以上的索引，最后取交集或者并集，常见and ，or的条件使用了不同的索引，官方排序这个在ref_or_null之后，但是实际上由于要读取多个索引，性能可能大部分时间都不如range |
| range       | 索引范围扫描，常见于使用>,<,is null,between ,in ,like等运算符的查询中。 |
| index       | 索引全表扫描，把索引从头到尾扫一遍，常见于使用索引列就可以处理不需要读取数据文件的查询、可以使用索引排序或者分组的查询 |
| all         | 这个就是全表扫描数据文件，然后再在server层进行过滤返回符合要求的记录 |

#### Extra 相关信息

**Using** **filesort**

在Mysql中使用explain来查看sql执行信息时，经常会看到Using filesort。那么Using filesort在MySQL中代表什么意思呢？

有人会说是外部排序，其实是不对或者不准确的。事实上Using filesort是一个非常差的命名。真实的情况是，如果一个排序操作不能通过索引来完成，那这次排序操作就叫做filesort，这跟file没有任何关系。filesort应该叫做sort，而它的实现，就是大家熟悉的快排。