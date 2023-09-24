### 查询

#### where



#### group by

单独使用 group by 关键字时，查询结果只会显示每个分组的第一条记录；

- 可以与**GROUP_CONCAT()** 函数一起使用， GROUP_CONCAT() 会把每个分组的字符值显示出来

```sql
select sex, group_concat(name) from student_info group by sex;

+------+----------------------------+
| sex  | GROUP_CONCAT(name)         |
+------+----------------------------+
| 女   | Henry,Jim,John,Thomas,Tom  |
| 男   | Dany,Green,Jane,Lily,Susan |
+------+----------------------------+
```

- 使用**COUNT() **统计记录的条数，**SUM()** 计算字段值的总和，**AVG()**计算字段值的平均值，还有**MAX()、MIN()** 等 **聚合函数**

```sql
select sex, count(sex) from student_info group by sex;

+------+------------+
| sex  | COUNT(sex) |
+------+------------+
| 女   |          5 |
| 男   |          5 |
+------+------------+
```

- WITH POLLUP：用来在所有记录的最后加上所有记录的总和；

```sql
select sex, group_concat(name) from student_info group by sex with pollup;

+------+------------------------------------------------------+
| sex  | GROUP_CONCAT(name)                                   |
+------+------------------------------------------------------+
| 女   | Henry,Jim,John,Thomas,Tom                            |
| 男   | Dany,Green,Jane,Lily,Susan                           |
| NULL | Henry,Jim,John,Thomas,Tom,Dany,Green,Jane,Lily,Susan |
+------+------------------------------------------------------+
```



#### having

##### where 区别和having区别

- where 用于过滤 **数据行**，having 用于过滤**分组**；前者在数据分组前过滤，后者数据分组后过滤；
- where不可使用 **聚合函数**，而having可以
- where 针对数据库文件进行过滤，后者针对查询结果（前面已查询出的字段）进行过滤；
- where不能使用字段别名，后者可以



#### 子查询

##### 实例1

```sql
# 查询学习java的同学
select name from tb_student_info
where course_id in (
	select id from tb_course where course_name = 'java'
);

# 先单独执行内循环
select tb_course where course_name = 'java';
+----+
| id |
+----+
|  1 |
+----+

# 再执行外层循环
select name from tb_student_info where course_id in (1);
+-------+
| name  |
+-------+
| Dany  |
| Henry |
+-------+
```



#### 总结

- 能用单表尽量用单表，即使需要用 group by、order by、 limit 等；
- 不能用单表时候尽量用连接。但join 次数不宜过多，连接是接近指数级增长的关联过程；
- 能不用子查询、笛卡尔积尽量不用，效率难以保证；
- 自定义变量在复杂 SQL中会很有用；
- 某些带聚合功能的查询需求应用窗口函数是个最优选择；
