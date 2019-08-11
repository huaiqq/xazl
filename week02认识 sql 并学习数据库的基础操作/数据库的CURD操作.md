###基本概念
crud操作，表示是增删改查. c[create] / r[read] / u[update] /d[delete]
### insert语句
+ 将数据添加到表中
+ 基本语法
```
insert into tablename [(column1 [,column2,,...])]
values (value1 [,value2...]);
```
+ 添加数据时，可以一次添加多条数据
+ 可以选择字段添加部分字段数据
```
字段  id   name   price
insert into goods values(10,'可口可乐',3.0);
insert into goods (id,name,price) values(20,'小小酥',2.5);
insert into goods (id,name) values(30,'红牛');
```
+ 插入空值的前提是该字段允许为空,还要注意字段是否有默认值
### update语句
```
update tablename
    set col_name1=expr1 [,col_name2=expr2...]
    [where expr];
```
```
将老妖怪的薪水在原有基础上增加1000元
update worker 
    set salary=(select salary from worker where user_name='老妖怪')+1000 
    where user_name='老妖怪';
-------------------
update worker 
    set salary = salary + 1000 
    where user_name='老妖怪';
```
### delete语句
```
delete from tablename
    where expr;
```
+ delete语句不能删除某一列，可以使用update将该字段设置为null
+ delete会返回删除的记录数
##### 小技巧：快速的复制一张表
```
create table worker2 like worker; //会创建一张空表，结构与worker相同
insert into worker2 select * from worker;
```
### select语句
```
select [distinct] *|{column1,column2,column3..}
    from tablename;

如果我们希望过滤重复的数据，则加上 distinct
* 号表示将所有的字段都检索出来，一般来说我们开发中不会使用select * 语句.这种语句会返回所有字段，效率较低
如果我们只希望检索某几列，则写清楚字段名就可以
我们的使用原则是，需要什么字段，就取回什么字段

select distinct * from student;
```
+ select语句可以对列进行运算
```
select *|{column1 | expression,column2..}
    from tablename;

别名写法
select name,chinese+english+math from student;
select name,chinese+english+math as totalgrade from student;

MariaDB [test]> select name,(chinese+english+math)totalgrade from student;
+-----------+------------+
| name      | totalgrade |
+-----------+------------+
| 韩顺平    |     257.00 |
| 宋江      |     242.00 |
| 关羽      |     276.00 |
| 赵云      |     233.00 |
| 欧阳锋    |     185.00 |
| 黄蓉      |     170.00 |
+-----------+------------+
6 rows in set (0.000 sec)
```
##### where子句常用运算符
+ 比较运算符
>,<, <,=, >,=, =, <> ,!=
betwween ... and ...
in(set)
like
not like
is null
+ 逻辑运算符
and，or，not
-----------------------
##### order by子句
用于排序
```
select name,math from student order by math desc;
select name,math from student order by math asc; //默认升序
select name,math from student order by math desc,name asc; 

```


