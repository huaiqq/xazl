### 整型
用于保存整数，常见的有tinyint，smallint，mediumint，int，bigint
![](https://upload-images.jianshu.io/upload_images/18609861-2197a2e0ceb13d2d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
+ tinyint(1个字节) < smallint(2个字节) <mediumint(3个字节) < int(4个字节) <bigint(8个字节)
+ 整型分为有符号和无符号
```
MariaDB [test]> create table people(
    - > name varchar(32) not null default '' comment '用户名', 
    - > age tinyint unsigned not null default 0 comment '年龄'
    - > )charset=utf8 engine=myisam;
Query OK, 0 rows affected (0.013 sec)
```
+ 如果不写unsigned，默认是有符号的
+ 有符号和无符号能表示的范围是不同的，有符号要把二进制最左一位拿出来表示符号正负

### 关于zerofill的说明
zerofill叫做0填充  
举例说明：
```
MariaDB [test]> create table test(
    -> num int,
    -> num2 int(4) zerofill,
    -> num3 int(4) unsigned zerofill);
Query OK, 0 rows affected (0.014 sec)
```
当int(4) zerofill 使用，如果添加的整数不够4位，则数值的左边使用0进行填充.
```
MariaDB [test]> insert into test values(1,23,345);
Query OK, 1 row affected (0.010 sec)

MariaDB [test]> select * from test;
+------+------+------+
| num  | num2 | num3 |
+------+------+------+
|    1 | 0023 | 0345 |
+------+------+------+
1 row in set (0.000 sec)
```
int(4) 不能理解成最大只能是4位的数，而应该理解成是0填充的宽度
```
MariaDB [test]> insert into test values(1,23456,34567);
Query OK, 1 row affected (0.004 sec)

MariaDB [test]> select * from test;
+------+-------+-------+
| num  | num2  | num3  |
+------+-------+-------+
|    1 |  0023 |  0345 |
|    1 | 23456 | 34567 |
+------+-------+-------+
2 rows in set (0.000 sec)
```
当一个字段被zerofill 修饰时，那么这个字段就自动成为unsigned
```
MariaDB [test]> insert into test values(1,-1,34567);
ERROR 1264 (22003): Out of range value for column 'num2' at row 1
```
+ int 如果不规定大小，默认是10
```
MariaDB [test]> create table test2(id int zerofill,age int);
Query OK, 0 rows affected (0.040 sec)

MariaDB [test]> insert into test2 values(1,18);
Query OK, 1 row affected (0.005 sec)

MariaDB [test]> select * from test2;
+------------+------+
| id         | age  |
+------------+------+
| 0000000001 |   18 |
+------------+------+
1 row in set (0.000 sec)
```

### 数值类型-bit
bit类型就是位类型
+ 基本使用案例
```
MariaDB [test]> create table test10(id int,a bit(7));
Query OK, 0 rows affected (0.005 sec)
MariaDB [test]> insert into test10 values(1,64);
Query OK, 1 row affected (0.010 sec)
MariaDB [test]> select * from test10;
+------+------+
| id   | a    |
+------+------+
|    1 | @    |
+------+------+
1 row in set (0.000 sec)
这里有一个困惑：a这个字段，bit位数为7的时候刚好，小了不行
解惑：bit(1) 这里1呢 就是一个bit位，意思是只能有1位来表示你的数据，就只能表示0和1

```
可见，bit字段在显示时，按照对应的ascii码对应的字符显示
```
MariaDB [test]> select * from test10 where a=64;
+------+------+
| id   | a    |
+------+------+
|    1 | @    |
+------+------+
1 row in set (0.000 sec)

MariaDB [test]> select * from test10 where a='@';
Empty set, 1 warning (0.001 sec)
```
+ 查询的时候仍然可以用数值
+ 位类型。默认值是1，范围是1-64,bit(1-64),可以通过bit(M) M值来控制我们填充数据的大小
+ 如果一个值只有一个0,1，可以考虑使用bit(1)，可以节约空间，其实实际分配的是一个字节，不能分配一个位
+ bit 类型, 不能手动指定unsigned，其实它本身就会是无符号的
### 数值类型--小数
+ 在mysql中使用的最多的是float , decimal
```
float(4,2)表示的范围是 -99.99~99.99
float(4,2) unsigned 表示的范围是 0-99.99
decimal(5,2)表示的范围是 -999.99~999.99
decimal(5,2) unsigned 表示的范围是 0-999.99
指定长度，指定小数点位数
```

### 数据类型--字符串
+ 最主要的有三种, 分别是 char,varchar, text 
```
MariaDB [test]> create table `user300`( 
    -> id int unsigned not null default 0, 
    -> name varchar(64) not null default '', 
    -> post_code char(6) not null default ''
    -> )charset=utf8 engine=myisam;
Query OK, 0 rows affected (0.013 sec)

MariaDB [test]> insert into user300 values(100,'你好','hello');
Query OK, 1 row affected (0.010 sec)
MariaDB [test]> select * from user300;
+-----+--------+-----------+
| id  | name   | post_code |
+-----+--------+-----------+
| 100 | 你好   | hello     |
+-----+--------+-----------+
1 row in set (0.002 sec)
```
注意：
+ char(n)这个n范围是 1-255
+ varchar(n)这个n的范围和表的字符集有关，如果是utf8，那么n最大是(65535-3)/3 = 21844;如果是gbk，n最大是(65535-3)/2 = 32766;
+ n指的都是字符数而不是字节数，与表的编码无关
+ varchar最大是有65535个字节，要有3个预留字节
+ 因为utf8一个汉字占3个字节；gbk一个汉字占两个字节
```
char(4)是定长 这里既是插入的是'aa'，也是占用分配的4个字节
varchar(4)是变长，如果插入了'aa'，实际占用空间大小是L+1
L：实际数据的长度
加的一个字节用来记录长度 
```
+ char 会将存入的最后的空格自动删除，而varchar会保留
+ 存放文本时，也可以使用text数据类型，不需要规定大小，text不能有默认值，可以将text视为varchar列，可以存放varchar最大的范围
+ 一个表定义的所有字段加起来不能超过65535，如果需求大于65535，我们可以使用text来替代varchar
----------------
### 数据类型--时间和日期
date，datetime，timestamp
```
MariaDB [test]> create table `user901`(
    -> id int,
    -> birthday date,
    -> cardtime datetime,
    -> login_time timestamp
    -> )charset=utf8 engine=myisam;
Query OK, 0 rows affected (0.010 sec)

MariaDB [test]> insert into user901 values(100,'2019-8-3','2019-8-8 8:8:4','2011-1-11 11:22:33');
Query OK, 1 row affected (0.009 sec)

MariaDB [test]> select * from user901;
+------+------------+---------------------+---------------------+
| id   | birthday   | cardtime            | login_time          |
+------+------------+---------------------+---------------------+
|  100 | 2019-08-03 | 2019-08-08 08:08:04 | 2011-01-11 11:22:33 |
+------+------------+---------------------+---------------------+
1 row in set (0.001 sec)
```
+ 对于date，只接收日期
+ datetime，timestamp 均有日期和时间
+ timestamp在insert和update时会自动根据当前时间更新
---------
### 数据类型--枚举enum，集合set
+ 对于多选我们可以使用set数据类型
+ 对于单选我们可以使用enum数据类型
```
MariaDB [test]> create table `user902`(
    -> id int unsigned not null default 1,
    -> hobby set('A','B','C','D') not null default 'A' comment '选项',
    -> sex enum('男','女') not null default '男' comment '性别'
    -> )charset=utf8 engine=myisam;
Query OK, 0 rows affected (0.009 sec)

MariaDB [test]> insert into user902 values(100,'A','男');
Query OK, 1 row affected (0.000 sec)

MariaDB [test]> insert into user902 values(200,'A,B','男');
Query OK, 1 row affected (0.000 sec)

MariaDB [test]> select * from user902;
+-----+-------+-----+
| id  | hobby | sex |
+-----+-------+-----+
| 100 | A     | 男  |
| 200 | A,B   | 男  |
+-----+-------+-----+
2 rows in set (0.002 sec)
```
+ 在enum类型中，数字可以替代我们的选项，类似下标理解，第一个选项为1，第二个选项为2，以此类推
```
MariaDB [test]> insert into user902 values(300,'C',1);
Query OK, 1 row affected (0.000 sec)

MariaDB [test]> insert into user902 values(400,'D',2);
Query OK, 1 row affected (0.000 sec)

MariaDB [test]> select * from user902;
+-----+-------+-----+
| id  | hobby | sex |
+-----+-------+-----+
| 100 | A     | 男  |
| 200 | A,B   | 男  |
| 300 | C     | 男  |
| 400 | D     | 女  |
+-----+-------+-----+
4 rows in set (0.000 sec)
```
+ set中也可以使用数字来替代，标号为1，2，4，6...偶数形式，最大64
```
A    B    C    D
1    2    4    6

7 => 1+2+4

```
+ 对set中的值进行查询
```
要借助函数
select * from user where find_in_set('A',hobby);
find_in_set('A',hobby) 返回'A'在hobby集合中处于第几位
```
### 图片电影音频数据应该如何存放？
通常不会直接存储在数据库汇中，实际开发中存放的是路径地址，然后通过地址去读取。
```
head_img varchar(64) //存储路径
```
## 创建表练习
```
MariaDB [test]> create table employee(
    -> id int unsigned not null default 0 comment '雇员id',
    -> name varchar(64) not null default '' comment '姓名',
    -> sex enum('男','女','保密') not null default '保密' comment '性别',
    -> birthday date not null comment '生日',
    -> entry_date date not null comment '入职时间',
    -> job varchar(32) not null default '' comment '职位',
    -> salary decimal(10,2) not null default 0.0 comment '薪水',
    -> resume text not null comment '个人介绍'
    -> )charset=utf8 engine=myisam;
Query OK, 0 rows affected (0.004 sec)
```
+ 插入数据
```
MariaDB [test]> insert into employee values(100,'小妖怪','男','2001-10-11','2012-11-11','巡山',2300.00,'大王.我来巡山');
Query OK, 1 row affected (0.001 sec)
MariaDB [test]> insert into employee values(200,'老妖怪','女','1999-10-11','2010-11-11','捶背',4300.00,'我给大王捶背');
Query OK, 1 row affected (0.000 sec)
```
+ 查看数据表
```
MariaDB [test]> select * from employee;
+-----+-----------+-----+------------+------------+--------+---------+-----------------------+
| id  | name      | sex | birthday   | entry_date | job    | salary  | resume                |
+-----+-----------+-----+------------+------------+--------+---------+-----------------------+
| 100 | 小妖怪    | 男  | 2001-10-11 | 2012-11-11 | 巡山   | 2300.00 | 大王叫我来巡山        |
| 200 | 老妖怪    | 女  | 1999-10-11 | 2010-11-11 | 捶背   | 4300.00 | 我给大王捶背          |
+-----+-----------+-----+------------+------------+--------+---------+-----------------------+
2 rows in set (0.002 sec)
```
+ 小技巧:如何对齐数据
```
    登录mysql的时候
    mysql -u root -p --dafault-character-set=latin1
    进入mysql之后 set names gbk;
```
