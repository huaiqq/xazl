### mysql表的内连接
内连接实际上就是利用where子句对多表形成的笛卡尔积进行筛选
前面我们使用的那种语法都属于内连接
`select 列名 from 表1  inner  join  表2  inner  join 表3  ON 条件 ....`
### mysql的外连接
分两种：左外连接，右外连接
#### 左外连接
+ 如果左侧的表完全显示我们就说是左外连接
`select 列名 表1 left join 表2 on 条件;`
```
mysql> select * from stu;
+------+------+
| id   | name |
+------+------+
|    1 | name |
|    2 | jack |
|    3 | tom  |
|    4 | kity |
+------+------+
4 rows in set (0.00 sec)

mysql> select * from exam;
+------+-------+
| id   | grade |
+------+-------+
|    1 |    55 |
|    2 |    45 |
|    3 |    89 |
+------+-------+
3 rows in set (0.00 sec)
```
+ 显示所有人的成绩，如果没有成绩，也要显示该人的姓名和id号，成绩显示为空（这种情况使用内连接是无法实现的）
`select * from stu left join exam on stu.id=exam.id;`
```
mysql> select stu.id,name,grade from stu left join exam on stu.id=exam.id;
+------+------+-------+
| id   | name | grade |
+------+------+-------+
|    1 | name |    55 |
|    2 | jack |    45 |
|    3 | tom  |    89 |
|    4 | kity |  NULL |
+------+------+-------+
4 rows in set (0.00 sec)
```
+ 如果上述要求，如果某位同学没有分数，则置为零
`select stu.id,name,ifnull(grade,0) as grade from stu left join exam on stu.id=exam.id;`
#### 右外连接
如果右侧的表完全显示我们就说是右外连接
`select 列名 from 表1  right  join  表2  ON 条件`
```
显示所有分数，没有对应人的分数对应的name置为无名
mysql> select stu.id,ifnull(name,'wuiming'),grade from stu right join exam on stu.id=exam.id;
+------+------------------------+-------+
| id   | ifnull(name,'wuiming') | grade |
+------+------------------------+-------+
|    1 | name                   |    55 |
|    2 | jack                   |    45 |
|    3 | tom                    |    89 |
| NULL | wuiming                |    99 |
+------+------------------------+-------+
4 rows in set (0.00 sec)
```






