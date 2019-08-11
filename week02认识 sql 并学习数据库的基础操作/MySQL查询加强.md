#### 查看表结构

`desc 表名;`
`show create table 表名;`

![](https://upload-images.jianshu.io/upload_images/18609861-272dbacb45f69cb5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#### 使用算术表达式加强
```
计算每个雇员的年薪
select (ifnull(sal,0.0)+ifnull(comm,0.0)) * 12 as 年薪 from emp;
```
#### where子句的加强
select * from emp where sal between 2000 and 3000;
select * from emp where sal>=2000 and sal<=3000;

显示首字符为SS的员工姓名和工资
select ename,sal from emp where 'S'=left(ename,1);
`select ename,sal from emp where ename like 'S%';`

显示第三个字符为大写o的所有员工的姓名和工资
`select ename,sal from emp where ename like '__O%';`

显示empno为123,234,800的雇员情况
`select * from emp where empo in (123,234,800);`
显示没有上级的雇员情况
`select * from emp where mgr is null;`
#### 逻辑操作符的加强
`select * from emp where (sal>500 or job='MANAGER') and ename like 'J%';`
#### 使用order by子句
按照工资从高到低的顺序显示
`select * from emp order by sal;`
按照部门号升序而雇员的工资降序排序
`select * from emp order by deptno,sal desc;`
#### mysql的分页查询
实际查询时，不可能把所有的记录都返回，而是一页一页返回，这是我们就会使用分页查询(**limit**)
+ 分页查询有两个重要的参数,\$pageSize 表示一页显示几条记录
分页查询有两个重要的参数 $pageNow 表示显示第几页 
 
`select 列名 from 表名 limit ($pageNow-1)*$pageSize,$pageSize;`  
+ 索引从0开始，所以下图是查询第一条记录
`select chinese from student limit 0,1;`
#### 聚合函数的加强-max,min,avg,sum,count
`select avg(sal),deptno from emp where deptno=20;`
显示工资最高的员工和其工作岗位
`select * from emp where sal=(select max(sal) from emp)`
显示工资高于平均工资的员工信息
`select * from emp where sal>(select avg(sal) from emp);`
#### group by 和having子句的加强
显示**每个部门**的平均工资和最高工资
`select avg(sal),max(sal) from emp group by deptno;`
显示**每个部门的每种岗位**的平均工资和最低工资
`select avg(sal),max(sal) from emp group by deptno,job;`
显示平均工资低于2000的部门号和它的平均工资
`select avg(sal) as myavg,deptno from emp group by deptno having myavg<2000;`
#### 数据分组关键字顺序
同时出现书写顺序 group by ,having ,order by
统计各个部门的平均工资，并且是大于2000，按照平均工资从高到低排序
`select avg(sal) as myavg from emp group by deptno having myavg>2000 order by myavg desc;`



