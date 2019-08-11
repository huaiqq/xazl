### 基本介绍
修改表，就是指，当我们的表创建好了，根据业务逻辑的需要， 我们要对表进行修改(比如 修改表名， 增加字段，删除字段，修改字段长度，修改字段类型)
### 基本语法
+ alter table tablename add column datatype [...];
+ alter table tablename modify column datatype [...];
+ alter table tablename drop column;
+ 修改表的名称: rename table 表名 to 新表名;
+ 修改表的字符集: alter table tablename character set utf8;
+ 查看表的结构: desc tablename;
### 案例
![](https://upload-images.jianshu.io/upload_images/18609861-3bd43b59e0aeb2d1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
1.在上面员工表增加一个image列(要求在resume后面)
    alter table employee add 
    image varchar(64) not null default '' after resume;
2.修改job列，将其长度改为60（）
    alter table employee modify 
    job varchar(60) not null default '';
    其实就是重写字段将原来的覆盖
3.删除sex列
    alter table employee drop sex;
4.表名修改为worker
    rename table employee to worker;
5.修改表的字符集
    alter table worker character set utf8;
6.将列名name修改为user_name
    alter table worker change 
    name user_name varchar(64) not null default '';

```
+ modify不能修改字段名字，要使用change
---------------
### 注意
+ 如果我们删除了某个字段，那么表的该字段内容就被删除，找不回，所以要小心
+ 如果我们把一个字段的长度减小 varchar(64) ==> varchar(32) , 如果你当前这个字段没有任何数据，是可以ok，但是如果已经有数据则看实际情况，就是如果有数据超过 32 了，则会提示错误
+ 对字段类型的修改， 比如 varchar ==> int 那么要看你的varchar 的内容是否可以转成int, 'hello'=>int 就不能成功