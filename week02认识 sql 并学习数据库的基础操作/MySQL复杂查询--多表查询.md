#### 多表查询
多表查询是指基于两个和两个以上的表或是视图的查询.在实际应用中,查询单个表可能不能满足你的需求,（如显示sales部门位置和其员工的姓名),这种情况下需要使用到(dept表和emp表)
+ 如果直接对两个表联查，将会得到结果=>笛卡尔集
`select * from emp,dept;`
+ 笛卡尔集
当我们对多表进行联查时，我们就会得到的 表1记录数 * 表2的记录数
![](https://upload-images.jianshu.io/upload_images/18609861-9f884b65685c3649.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
+ 在多表查询中，由于笛卡尔集的原理，重要的是要找到多表关联的字段
`select * from emp,dept where emp.deptno=dept.deptno;`
若两表中关联的字段名称不相同，也可以不加表名限制
`select ename,sal,dname,dept.deptno from emp,dept where emp.deptno=dept.deptno;`
显示emp所有字段加dept表中的dname
`select emp.*,dept.dname from emp,dept where emp.deptno=dept.deptno;`
+ 多表查询的条件是至少不能少于表的个数-1
```
select 
dept.dname,emp.dname,emp.sal 
from emp,dept 
where emp.deptno=dept.deptno 
and emp.deptno=10;
```
+ 关键在于where语句对笛卡尔集的规避，对表的关系要搞清楚

### 自连接
   自连接是指在同一张表的连接查询

`select * from emp where empno=(select mgr from emp where ename='ford');`
1. 将同一张表看做两张表
![](https://upload-images.jianshu.io/upload_images/18609861-72fe4a10ca301b11.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
select 
worker.ename,boss.ename 
from emp as worker,emp as boss 
where 
worker.mgr = boss.empno;
```
### 子查询
子查询是指嵌入在其它sql查询语句中的select语句,也叫嵌套查询
#### 单行子查询
单行子查询是**指只返回一行数据的子查询语句**
`select * from emp where deptno=(select deptno from emp where ename='smith');`
#### 多行子查询
多行子查询指返回多行数据的子查询   使用关键字 in
```
如何查询和10号部门的工作相同的雇员的信息，但是不含10号部门的雇员。
select * 
from emp 
where job 
in 
(select distinct job from emp where deptno=10)
and deptno <>10;
```
#### 多列子查询
如果我们的一个子查询，返回的 结果是多列，就叫做列子查询
```
查询和宋江数学，英语，语文完全相同的学生
select * from student 
where 
(math,english,chinese) 
= (select math,english,chinese from student where name='宋江');
```
#### from子句中使用子查询
+ **将一个查询结果看做一个临时表**
案例：显示高于自己部门平均工资的员工的信息
```
select ename, sal, temp. myavg,emp.deptno  from 
emp, (select avg(sal) as myavg , deptno from emp group by deptno) as temp
where 
emp.deptno = temp.deptno 
AND
emp.sal > temp. myavg;
```
#### 合并查询
合并多个select语句的结果，可以使用集合操作符 union,union all
+ union 合并查询
`select 语句1 union select 语句2;`
两个sql语句结果合并返回，而且会去掉重复的记录

+ union all 和union唯一的区别就是union all 不去重















