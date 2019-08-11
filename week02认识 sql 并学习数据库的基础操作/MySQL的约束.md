约束是用于维护数据的完整性, 所谓数据的完整性指的是，根据业务逻辑的需要，我们将表的某些字段设置成一种约束，从而保证数据的合理性和完整性，比如身份证编号就是唯一，我们就可以使用unique 这个约束.
### mysql的约束分类
1. 主键约束: primary key
2. 唯一约束: unique
3. not null
4. check ：检查约束，很多数据库都有，但是mysql支持语法，并没有实际作用 
5. 外键约束: foreign key
#### 主键约束
+ 主键约束一般在int，字符型设置
+ 一旦一个字段被设置成主键约束，则该字段不能重复，也不能为null
+ 一张表最多只能有一个主键，但是可以是复合主键(两个字段合起来当一个主键)
```
  create table `user`(
  id int,
  name varchar(32) not null default '',
  email varchar(32) not null default '',
  primary key (id,name)
  )charset=utf8 engine=muisam;
```
+ 当id,name同时重复才会认为是重复冲突的
1,'aaa','aaa@.com' 可以插入
1,'bbb','bbb@.com' 也可以插入
1,'aaa','bbb@.com' 不可以插入
+ not null 和unique 的效果非常像 primary key
#### not null 约束
#### unique约束(唯一约束)
当我们希望某个字段的值不能出现重复值时，我们就可以将该字段设置成unique
+ 如果我们将某个字段设置为 unique ,但是没有设置  not null, 那么该字段可以为null,并且可以有多个null
+ 如果你要某个字段不能重复，而且不能为 null, 我们可以这样定义字段
字段名 字段类型 not null  unique
+ 一个表中，可以有多个unique约束.
#### 外键约束
用于定义表和和从表之间的关系:外键约束要定义在从表上，**主表则必须具有主键约束或是unique约束**，当定义外键约束后，要求外键列数据必须在主表的主键列存在或是为null
`foreign key (外键(本表)字段)  references 主表(字段);`
+ 外键字段是在本表定义的
+ 外键字段是指向另外一张表的某个字段
![](https://upload-images.jianshu.io/upload_images/18609861-392ecc0c554f4482.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
1. 先创建主表
```
mysql> create table my_class(
    -> class_id int primary key,
    -> class_name varchar(64) not null unique,
    -> class_intro text not null
    -> )charset=utf8 engine=innodb;
Query OK, 0 rows affected (0.02 sec)
```
2. 创建从表
```
mysql> create table stu(
    -> id int primary key,
    -> name varchar(64) not null default '',
    -> class_id int,
    -> /*定义外键*/
    -> foreign key (class_id) references my_class(class_id)
    -> )charset=utf8 engine=innodb;
Query OK, 0 rows affected (0.02 sec)
```
+ 给stu表添加数据时，要求对应的my_class表的class_id已经存在
+ 如果外键没有设置not null，那么外键的值可以是null，而且可以有多个

`mysql> insert into stu values(1,'张三',1);    
ERROR 1452 (23000): Cannot add or update a child row: a foreign key constraint fails (`test`.`stu`, CONSTRAINT `stu_ibfk_1` FOREIGN KEY (`class_id`) REFERENCES `my_class` (`class_id`))`
+ 由于刚刚创建的my_class表中没有class_id值，不能直接向stu中添加数据
+ 表的类型要innodb，这样的类型支持外键
+ 外键字段的类型要和主键字段的类型一致(长度可以不同)
+ 外键字段的值，必须在主键出现过，或者为null
+ 一旦建立了主外键的关系，数据不能随意删除和修改了