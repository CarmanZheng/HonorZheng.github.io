---
title: flask数据库flask-SQLAlchemy
layout: post
tags: web开发
categories: ''
---

## 一. SQLAlchemy介绍

SQLAlchemy是Python中最有名的ORM工具。

数据库db查看工具SQLiteSpy：https://www.yunqa.de/delphi/products/sqlitespy/index

### 1. 关于ORM

全称Object Relational Mapping（对象关系映射）。

特点是操纵Python对象而不是SQL查询，也就是在代码层面考虑的是对象，而不是SQL，体现的是一种程序化思维，这样使得Python程序更加简洁易读。

### 2.ORM框架

| 数据库 |         python         |
| :----: | :--------------------: |
|   表   |           类           |
|   列   |        类的属性        |
|   行   | 类的实例，字典对象表述 |

字典对象的key对应列，字典对象的value对应值；

具体的实现方式是将数据库表转换为Python类，其中数据列作为属性，数据库操作作为方法。

优点：

- 简洁易读：将数据表抽象为对象（数据模型），更直观易读
- 可移植：封装了多种数据库引擎，面对多个数据库，操作基本一致，代码易维护
- 更安全：有效避免SQL注入

为什么要用sqlalchemy?

虽然性能稍稍不及原生SQL，但是操作数据库真的很方便！

## **二. 使用**

需要安装pymysql ，sqlalchemy，flask-sqlalchemy三个库

概念和数据类型

概念

| 概念    | 对应数据库   | 说明                 |
| :------ | :----------- | :------------------- |
| Engine  | 连接         | 驱动引擎             |
| Session | 连接池，事务 | 由此开始查询         |
| Model   | 表           | 类定义               |
| Column  | 列           |                      |
| Query   | 若干行       | 可以链式添加多个条件 |

常见数据类型

| 数据类型 | 数据库数据类型 | python数据类型    | 说明                   |
| :------- | :------------- | :---------------- | :--------------------- |
| Integer  | int            | int               | 整形，32位             |
| String   | varchar        | string            | 字符串                 |
| Text     | text           | string            | 长字符串               |
| Float    | float          | float             | 浮点型                 |
| Boolean  | tinyint        | bool              | True / False           |
| Date     | date           | datetime.date     | 存储时间年月日         |
| DateTime | datetime       | datetime.datetime | 存储年月日时分秒毫秒等 |
| Time     | time           | datetime.datetime | 存储时分秒             |

创建数据库表

### 1.安装

```python
pip install SQLalchemy
```

### 2.创建连接

```python
from sqlalchemy import create_engine
engine = create_engine("mysql://user:password@hostname/dbname?charset=uft8")
```

这行代码初始化创建了Engine，Engine内部维护了一个Pool（连接池）和Dialect（方言），方言来识别具体连接数据库种类。

![image-20200919134903113](images\image-20200919134903113.png)

创建好了Engine的同时，Pool和Dialect也已经创建好了，但是此时并没有真正与数据库连接，等到执行具体的语句.connect()等时才会连接到数据库。

```python
engine = create_engine("mysql://user:password@hostname/dbname?charset=uft8",
            echo=True,
            pool_size=8,
            pool_recycle=60*30
            )
```

- echo: 当设置为True时会将orm语句转化为sql语句打印，一般debug的时候可用
- pool_size: 连接池的大小，默认为5个，设置为0时表示连接无限制
- pool_recycle: 设置时间以限制数据库多久没连接自动断开

### **3. 创建数据库表类（模型）**

前面有提到ORM的重要特点，那么我们操作表的时候就需要通过操作对象来实现，现在我们来创建一个类，以常见的用户表举例：

```python
from sqlalchemy.ext.declarative import declarative_base
Base = declarative_base()
class Users(Base):
  __tablename__ = "users"
  id = Column(Integer, primary_key=True)
  name = Column(String(64), unique=True)
  email = Column(String(64))
  def __init__(self, name, email):
       self.name = name
       self.email = email
```

declarative_base()是sqlalchemy内部封装的一个方法，通过其构造一个基类，这个基类和它的子类，可以将Python类和数据库表关联映射起来。

数据库表模型类通过__tablename__和表关联起来，Column表示数据表的列。

### 4.数据库与pandas数据交换

https://mp.weixin.qq.com/s/-ZODFjsoW36BzD3cCgTQ3A

opcua数据读入到数据库

```python
from sqlalchemy import create_engine

# todo step1 定义数据库引擎
engine = create_engine('sqlite:///' + r'D:\02_Task\07_EdgeTech\11_SQL\opcua2sql.db')
cur_data_1s.to_sql('wind', engine, index=False, if_exists='append')
print(var_data)
```

数据读出 sql2pandas

```python
import pandas as pd
from sqlalchemy import create_engine

# step1 创建sql引擎
engine = create_engine('sqlite:///' + r'D:\02_Task\07_EdgeTech\11_SQL\opcua2sql.db')
# step2 写明查找内容
sql = 'select datetime,blade1_acc_xa from wind'
# step3 读取数据库数据
df = pd.read_sql_query(sql, engine)
print(df)
```

## 三. MySQL介绍

https://www.bilibili.com/video/BV1DJ411m73j?from=search&seid=48499941822714157

https://blog.csdn.net/weixin_44783814/article/details/106709165

1. 如果一个项目是动态(jsp,php,shtml)的，则数据库服务器是必不可少的环节。
2. 在web应用方面，MySQL是最好的RDBMS(关系型数据库管理系统)

### 一.安装mysql

https://www.cnblogs.com/zhou0000/p/8509100.html

1.检查Linux系统中是否已经安装了MySQL，命令为：

```
　sudo service mysql start
```

　若提示mysql：unrecognized service则说明系统中没有MySQL，需要继续安装。

2.Ubuntu Linux 安装配置MySQL：在Ubuntu上安装MySQL，最简单的方式是在线安装

```
　命令为：sudo apt-get install mysql-server    #安装MySQL服务端，核心程序

　　　　　　sudo apt-get install mysql-client　  #安装MySQL客户端
　　　　　　
　　　　　　sudo apt-get install libmysqlclient-dev
　　　　　　         
           sudo apt install net-tools   #Ubuntu 安装 net-tools，执行：
```

　　在安装过程中会提示输入确认YES，设置root用户密码（之后可以修改）等

　　安装结束后，验证是否安装成功命令：

```
sudo netstat -tap | grep mysql
```

3.安装成功后，可以根据自己的需求，用个gedit修改MySQL的配置文件（my.cnf）,命令为：

```
　　sudo gedit /etc/mysql/my.cnf
```

至此，Linux中MySQL已经安装，配置完成。

错误鸡精：

```
	ERROR 1524 (HY000) 
	解决： 重启数据库
	sudo /etc/init.d/mysql stop 
	sudo /etc/init.d/mysql start 
```



如果依然报错，那就卸载后重装；

### 二.运行mysql

1.打开MySQL

　　启动MySQL：　　

```
sudo service mysql start
```

　　使用root用户登录：

```
mysql -u root -p
```

​	**如果定义普通用户登录，需要赋权到组；**

2.其他操作命令见：http://www.cnblogs.com/zhou0000/p/8287520.html

PS：1. Windows和Linux中MySQL的基本操作命令大体没什么区别

　　2. 注意每条语句结尾要写分号;

### 三.创建用户组

登录MySQL

登录本地用户

```
mysql -u root -p
```

登录外网用户（需要注意服务器可能只允许本地登录，需要修改响应的配置文件）

```mysql
mysql -u zheng -h 10.64.6.4 -p1
```

添加用户

1.允许本地访问的用户（127.0.0.1）

```mysql
create user zheng@localhost identified by '123456';  
```

2.允许外网IP访问的用户

```mysql
create user 'zheng'@'%' identified by '123456'; 
```

用户分配权限

授予用户在本地服务器对该数据库的全部权限

```mysql
grant all privileges on wind to zheng@'localhost';
```

授予用户通过外网IP对于该数据库的全部权限

```mysql
grant all privileges on wind to 'zheng'@'%' identified by "123456";  
```

刷新权限

```mysql
flush privileges;  
```

### 四.mysql常用语句 

https://www.cnblogs.com/xieqijiang/p/10949941.html

最常用的显示命令：

1、显示数据库列表。

```mysql
show databases;
```

2、显示库中的数据表：

```mysql
use 数据库;
show tables;
```

3、显示数据表的结构：

```mysql
describe 表名;
```

4、建库：

```mysql
create database 库名;
```

5、建表：

```mysql
use 库名;
create table 表名 (字段设定列表);
```

6、删库和删表:

```mysql
drop database 库名;
drop table 表名;
```

7、将表中记录清空：

```mysql
delete from 表名;  (这个清空表只是把数据表内容数据清掉，自增id不会被清掉，自增id会保留)
```

```mysql
truncate table 表名；（成功返回0）（自增id也一同会被清掉）
```

truncate与delete的区别：
a.事务：truncate是不可以rollback的，但是delete是可以rollback的；
原因：truncate删除整表数据(ddl语句,隐式提交)，delete是一行一行的删除，可以rollback
b.效果：truncate删除后将重新水平线和索引(id从零开始) ,delete不会删除索引
c.truncate 不能触发任何Delete触发器。
d.delete 删除可以返回行数

truncate与delete的区别：
a.事务：truncate是不可以rollback的，但是delete是可以rollback的；
原因：truncate删除整表数据(ddl语句,隐式提交)，delete是一行一行的删除，可以rollback
b.效果：truncate删除后将重新水平线和索引(id从零开始) ,delete不会删除索引
c.truncate 不能触发任何Delete触发器。
d.delete 删除可以返回行数

8、显示表中的记录：

```mysql
select * from 表名;
```

连接MySQL
格式: mysql -h 主机地址 -u用户名 -p用户密码

例 1:连接到本机上的 MySQL。

```linux
mysql -u root -p mysql;
```

连接到远程主机上的 MYSQL。

```
mysql -h 127.0.0.1 -uroot -pmysql;
```

2、连接到远程主机上的MYSQL。假设远程主机的IP为：110.110.110.110，用户名为root,密码为abcd123。则键入以下命令：

```
mysql -h 110.110.110.110 -u root -p 123;
（注:u与root之间可以不用加空格，其它也一样）
```

3、退出MYSQL命令：

```
 exit （回车）
```

修改新密码

```
在终端输入：mysql -u用户名 -p密码，回车进入Mysql。

\> use mysql;

\> update user set password=PASSWORD('新密码') where user='用户名';

\> flush privileges; #更新权限

\> quit; #退出
```


二、修改密码

格式：

```mysql
mysqladmin -u用户名 -p旧密码 password 新密码
```

1、给root加个密码ab12。首先在DOS下进入目录mysql\bin，然后键入以下命令

```mysql
mysqladmin -u root -password ab12
```

注：因为开始时root没有密码，所以-p旧密码一项就可以省略了。
2、再将root的密码改为djg345。

mysqladmin -u root -p ab12 password djg345
（注意：和上面不同，下面的因为是MYSQL环境中的命令，所以后面都带一个分号作为命令结束符）
3、命令行修改root密码：

mysql> UPDATE mysql.user SET password=PASSWORD(’新密码’) WHERE User=’root’;

mysql> FLUSH PRIVILEGES;
4、显示当前的user：

mysql> SELECT USER();


三、增加新用户

格式：

```
grant select on 数据库.* to 用户名@登录主机 identified by “密码”
```

1、增加一个用户test1密码为abc，让他可以在任何主机上登录，并对所有数据库有查询、插入、修改、删除的权限。首先用root用户连入
MYSQL，然后键入以下命令：

```mysql
grant select,insert,update,delete on *.* to test1”%" Identified by “abc”;
```

但增加的用户是十分危险的，你想如某个人知道test1的密码，那么他就可以在internet上的任何一台电脑上登录你的mysql数据库并对你的数据可以为所欲为了，解决办法见2。

2、增加一个用户test2密码为abc,让他只可以在localhost上登录，并可以对数据库mydb进行查询、插入、修改、删除的操作（localhost指本地主机，即MYSQL数据库所在的那台主机），

这样用户即使用知道test2的密码，他也无法从internet上直接访问数据库，只能通过MYSQL主机上的web页来访问了。

```mysql
grant select,insert,update,delete on mydb.* to test2@localhost identifiedby “abc”;
```

如果你不想test2有密码，可以再打一个命令将密码消掉。

```mysql
grant select,insert,update,delete on mydb.* to test2@localhost identified by “”;
```


删除用户

```mysql
mysql -u用户名 -p密码

mysql> delete from user where user='用户名' and host='localhost';

mysql> flush privileges;
```

//删除用户的数据库

```mysql
mysql> drop database dbname;
```

用户管理

```mysql
use mysql;
```

查看

```mysql
 select host,user,password from user ;
```

创建

```mysql
create user  zx_root   IDENTIFIED by 'xxxxx';  
//identified by 会将纯文本密码加密作为散列值存储
```

修改

```mysql
rename  user  feng  to  newuser；
//mysql 5之后可以使用，之前需要使用update 更新user表
```

删除

```mysql
drop user newuser; 
 //mysql5之前删除用户时必须先使用revoke 删除用户权限，然后删除用户，mysql5之后drop 命令可以删除用户的同时删除用户的相关权限
```

更改密码

```mysql
 set password for zx_root =password('xxxxxx');
```

 mysql> update  mysql.user  set  password=password('xxxx')  where user='otheruser'

查看用户权限

mysql> show grants for zx_root;

赋予权限

mysql> grant select on dmc_db.*  to zx_root;

回收权限

mysql> revoke  select on dmc_db.*  from  zx_root;  //如果权限不存在会报错

#### 表操作

```mysql
mysql> create table MyClass(

id int(4) not null primary key auto_increment,

name char(20) not null,

sex int(4) not null default '0',

degree double(16,2));


```

获取表结构

命令: desc 表名，或者show columns from 表名

例子：

```mysql
mysql> describe MyClass

mysql> desc MyClass;

mysql> show columns from MyClass;
```

删除表

```mysql
命令:drop table <表名>

例如:删除表名为 MyClass 的表

mysql> drop table MyClass;
```

插入数据

```mysql
命令:insert into <表名> [( <字段名 1>[,..<字段名 n > ])] values ( 值 1 )[, ( 值 n )]

例子：

mysql> insert into MyClass values(1,'Tom',96.45),(2,'Joan',82.99), (2,'Wang', 96.59);
```

查询表中的数据

查询所有行

```mysql
mysql> select * from MyClass;
```

查询前几行数据

例如:查看表 MyClass 中前 2 行数据

```mysql
mysql> select * from MyClass order by id limit 0,2;
```

或者

```mysql
mysql> select * from MyClass limit 0,2;
```

删除表中数据

命令:delete from 表名 where 表达式

例如:删除表 MyClass 中编号为 1 的记录

```mysql
mysql> delete from MyClass where id=1;
```

修改表中数据

命令:update 表名 set 字段=新值,... where 条件

```mysql
mysql> update MyClass set name='Mary' where id=1;
```

在表中增加字段

命令:alter table 表名 add 字段 类型 其他;

例如:在表 MyClass 中添加了一个字段 passtest,类型为 int(4),默认值为 0

```mysql
mysql> alter table MyClass add passtest int(4) default '0';
```

更改表名

命令:rename table 原表名 to 新表名;

例如:在表 MyClass 名字更改为 YouClass

```mysql
mysql> rename table MyClass to YouClass;
```

更新字段内容

命令:update 表名 set 字段名 = 新内容

update 表名 set 字段名 = replace(字段名, '旧内容', '新内容');

例如:文章前面加入 4 个空格

```mysql
update article set content=concat(' ', content);
```







数据库导入导出


四、从数据库导出数据库文件


使用“mysqldump”命令

首先进入 DOS 界面,然后进行下面操作。

1)导出所有数据库

格式:mysqldump -u [数据库用户名] -p -A>[备份文件的保存路径]

2)导出数据和数据结构

格式:mysqldump -u [数据库用户名] -p [要备份的数据库名称]>[备份文件的保存路径]

举例:

例 1:将数据库 mydb 导出到 e:\MySQL\mydb.sql 文件中。

打开开始->运行->输入“cmd”,进入命令行模式。

c:\> mysqldump -h localhost -u root -p mydb >e:\MySQL\mydb.sql

然后输入密码,等待一会导出就成功了,可以到目标文件中检查是否成功。

例 2:将数据库 mydb 中的 mytable 导出到 e:\MySQL\mytable.sql 文件中。

c:\> mysqldump -h localhost -u root -p mydb mytable>e:\MySQL\mytable.sql

例 3:将数据库 mydb 的结构导出到 e:\MySQL\mydb_stru.sql 文件中。

c:\> mysqldump -h localhost -u root -p mydb --add-drop-table >e:\MySQL\mydb_stru.sql

备注:-h localhost 可以省略,其一般在虚拟主机上用。

3)只导出数据不导出数据结构

格式:

mysqldump -u [数据库w用户名] -p -t [要备份的数据库名称]>[备份文件的保存路径]

4)导出数据库中的Events

格式:mysqldump -u [数据库用户名] -p -E [数据库用户名]>[备份文件的保存路径]

5)导出数据库中的存储过程和函数

格式:mysqldump -u [数据库用户名] -p -R [数据库用户名]>[备份文件的保存路径]


五、从外部文件导入数据库中


1)使用“source”命令

首先进入“mysql”命令控制台,然后创建数据库,然后使用该数据库。最后执行下面操作。

mysql>source [备份文件的保存路径]

2)使用“<”符号

首先进入“mysql”命令控制台,然后创建数据库,然后退出 MySQL,进入 DOS 界面。最后执行下面操作。

mysql -u root –p < [备份文件的保存路径]


六、备份数据库：

注意，mysqldump命令在DOS的 mysql\bin 目录下执行,不能在mysql环境下执行，因此，不能以分号“;”结尾。若已登陆mysql，请运行退出命令mysql> exit
1.导出整个数据库

导出文件默认是存在mysql\bin目录下

mysqldump -u用户名 -p数据库名 > 导出的文件名

mysqldump -uroot -p123456 database_name > outfile_name.sql
2.导出一个表

mysqldump -u 用户名 -p 数据库名 表名> 导出的文件名

mysqldump -u user_name -p database_name table_name > outfile_name.sql
3.导出一个数据库结构

mysqldump -u user_name -p -d –add-drop-table database_name > outfile_name.sql

-d 没有数据 –add-drop-table 在每个create语句之前增加一个drop table
4.带语言参数导出

mysqldump -uroot -p –default-character-set=latin1 –set-charset=gbk –skip-opt database_name > outfile_name.sql


七、将文本数据转到数据库中

1、文本数据应符合的格式：字段数据之间用tab键隔开，null值用\n来代替.例：

3 rose 大连二中 1976-10-10

4 mike 大连一中 1975-12-23

假设你把这两组数据存为school.txt文件，放在c盘根目录下。
2、数据传入命令

mysql> load data local infile "c:\school.txt" into table 表名;

注意：你最好将文件复制到mysql\bin目录下，并且要先用use命令打表所在的库。

 

八、对表的操作

1、显示数据表的结构：

```mysql
mysql> DESCRIBE 表名; （DESC 表名）
```


2、建立数据表：

mysql> USE 库名; //进入数据库

```mysql
mysql> CREATE TABLE 表名 (字段名 VARCHAR(20), 字段名 CHAR(1));
```

3、删除数据表：

```mysql
mysql> DROP TABLE 表名；
```

4、重命名数据表

```mysql
alter table t1 rename t2;
```

5、显示表中的记录：

```mysql
mysql> SELECT * FROM 表名;
```


6、往表中插入记录：

```mysql
mysql> INSERT INTO 表名 VALUES ('hyq','M');
```

7、更新表中数据：

通过字段名3找到表中的行，将字段名1的值改为a，字段名2处的值改为b

```mysql
mysql-> UPDATE 表名 SET 字段名1='a',字段名2='b' WHERE 字段名3='c';
```

8、将表中记录清空：

```mysql
mysql> DELETE FROM 表名;
```

9、用文本方式将数据装入数据表中：

```mysql
mysql> LOAD DATA LOCAL INFILE “D:/mysql.txt” INTO TABLE 表名;
```

10、 显示表的定义，还可以看到表的约束，例如外键

```mysql
mysql> SHOW CREATE TABLE 表名 ;
```

还可以通过 mysqldump 将表的完整定义转储到文件中，当然包括外键定义。

还可以通过下面的指令列出表 T 的外键约束：

mysql> SHOW TABLE STATUS FROM yourdatabasename LIKE 'T'

外键约束将会在表注释中列出。
存储过程
11、创建存储过程
CREATE PROCEDURE procedureName (in paramentName type, in paramentName type,……)

BEGIN

SQL sentences;

END


12、调用存储过程

mysql> CALL procedureName(paramentList);
例：mysql> CALL addMoney(12, 500);


13、查看特定数据库的存储过程

方法一：mysql> SELECT `name` FROM mysql.proc WHERE db = 'your_db_name' AND `type` = 'PROCEDURE';

方法二：mysql> show procedure status;
14、删除存储过程

mysql> DROP PROCEDURE procedure_name;

mysql> DROP PROCEDURE IF EXISTS procedure_name;
15、查看指定的存储过程定义

mysql> SHOW CREATE PROCEDURE proc_name;

mysql> SHOW CREATE FUNCTION func_name;
---------- 示例一-----------

mysql> DELIMITER $$

mysql> USE `db_name`$$ //选择数据库

mysql> DROP PROCEDURE IF EXISTS `addMoney`$$ //如果存在同名存储过程，则删除之

mysql> CREATE DEFINER= `root`@`localhost` PROCEDURE `addMoney`(IN xid INT(5),IN xmoney INT(6))

mysql> BEGIN

mysql> UPDATE USER u SET u.money = u.money + xmoney WHERE u.id = xid; //分号";"不会导致语句执行，因为当前的分割符被定义为$$

mysql> END$$ //终止

mysql> DELIMITER ; //把分割符改回分号";"
mysql> call addMoney(5,1000); //执行存储过程
---------- 示例二-----------

mysql> delimiter //

mysql> create procedure proc_name (in parameter integer)

mysql> begin

mysql> if parameter=0 then

mysql> select * from user order by id asc;

mysql> else

mysql> select * from user order by id desc;

mysql> end if;

mysql> end;

mysql> // //此处“//”为终止符

mysql> delimiter ;

mysql> show warnings;

mysql> call proc_name(1);

mysql> call proc_name(0);

 

九、修改表的列属性的操作



1、为了改变列a，从INTEGER改为TINYINT NOT NULL(名字一样)，

并且改变列b，从CHAR(10)改为CHAR(20)，同时重命名它，从b改为c:

```mysql
mysql> ALTER TABLE t2 MODIFY a TINYINT NOT NULL, CHANGE b c CHAR(20);
```

2、增加一个新TIMESTAMP列，名为d：

```mysql
mysql> ALTER TABLE t2 ADD d TIMESTAMP;
```

3、在列d上增加一个索引，并且使列a为主键：

```mysql
mysql> ALTER TABLE t2 ADD INDEX (d), ADD PRIMARY KEY (a);
```

4、删除列c：

mysql> ALTER TABLE t2 DROP COLUMN c;
5、增加一个新的AUTO_INCREMENT整数列，命名为c：

mysql> ALTER TABLE t2 ADD c INT UNSIGNED NOT NULL AUTO_INCREMENT,ADD INDEX (c);

注意，我们索引了c，因为AUTO_INCREMENT柱必须被索引，并且另外我们声明c为NOT NULL，

因为索引了的列不能是NULL
十、一个建库和建表以及插入数据的实例
drop database if exists school; //如果存在SCHOOL则删除

create database school; //建立库SCHOOL

use school; //打开库SCHOOL

create table teacher //建立表TEACHER

(

id int(3) auto_increment not null primary key,

name char(10) not null,

address varchar(50) default ‘深圳’,

year date

); //建表结束

//以下为插入字段

insert into teacher values('','allen','大连一中','1976-10-10');

insert into teacher values('','jack','大连二中','1975-12-23');

如果你在mysql提示符键入上面的命令也可以，但不方便调试。

（1）你可以将以上命令原样写入一个文本文件中，假设为school.sql，然后复制到c:\下，并在DOS状态进入目录\mysql\bin，然后键入以下命令：

mysql -uroot -p密码 < c:\school.sql

如果成功，空出一行无任何显示；如有错误，会有提示。（以上命令已经调试，你只要将//的注释去掉即可使用）。

（2）或者进入命令行后使用 mysql> source c:\school.sql; 也可以将school.sql文件导入数据库中。
MySQL 为关系型数据库(Relational Database Management System), 这种所谓的"关系型"可以理解为"表格"的概念, 一个关系型数据库由一个或数个表格组成, 如图所示的一个表格:
\
表头(header): 每一列的名称;
列(row): 具有相同数据类型的数据的集合;
行(col): 每一行用来描述某个人、物的具体信息;
值(value): 行的具体信息, 每个值必须与该列的数据类型相同;
键(key): 表中用来识别某个特定的人\物的方法, 键的值在当前列中具有唯一性。

MySQL脚本的基本组成
与常规的脚本语言类似, MySQL 也具有一套对字符、单词以及特殊符号的使用规定, MySQL 通过执行 SQL 脚本来完成对数据库的操作, 该脚本由一条或多条MySQL语句(SQL语句 + 扩展语句)组成, 保存时脚本文件后缀名一般为 .sql。在控制台下, MySQL 客户端也可以对语句进行单句的执行而不用保存为.sql文件。

标识符
标识符用来命名一些对象, 如数据库、表、列、变量等, 以便在脚本中的其他地方引用。MySQL标识符命名规则稍微有点繁琐, 这里我们使用万能命名规则: 标识符由字母、数字或下划线(_)组成, 且第一个字符必须是字母或下划线。
对于标识符是否区分大小写取决于当前的操作系统, Windows下是不敏感的, 但对于大多数 linux\unix 系统来说, 这些标识符大小写是敏感的。

MySQL中的数据类型
MySQL有三大类数据类型, 分别为数字、日期\时间、字符串, 这三大类中又更细致的划分了许多子类型:
数字类型
整数: tinyint、smallint、mediumint、int、bigint
浮点数: float、double、real、decimal
日期和时间: date、time、datetime、timestamp、year
字符串类型
字符串: char、varchar
文本: tinytext、text、mediumtext、longtext
二进制(可用来存储图片、音乐等): tinyblob、blob、mediumblob、longblob

 

使用MySQL数据库

登录到MySQL
当 MySQL 服务已经运行时, 我们可以通过MySQL自带的客户端工具登录到MySQL数据库中, 首先打开命令提示符, 输入以下格式的命名:
mysql -h 主机名 -u 用户名 -p
-h : 该命令用于指定客户端所要登录的MySQL主机名, 登录当前机器该参数可以省略;
-u : 所要登录的用户名;
-p : 告诉服务器将会使用一个密码来登录, 如果所要登录的用户名密码为空, 可以忽略此选项。

以登录刚刚安装在本机的MySQL数据库为例, 在命令行下输入 mysql -u root -p 按回车确认, 如果安装正确且MySQL正在运行, 会得到以下响应:

Enter password:

若密码存在, 输入密码登录, 不存在则直接按回车登录, 按照本文中的安装方法, 默认 root 账号是无密码的。登录成功后你将会看到 Welecome to the MySQL monitor... 的提示语。

然后命令提示符会一直以 mysql> 加一个闪烁的光标等待命令的输入, 输入 exit 或 quit 退出登录。


创建一个数据库
使用 create database 语句可完成对数据库的创建, 创建命令的格式如下:

create database 数据库名 [其他选项];

例如我们需要创建一个名为 samp_db 的数据库, 在命令行下执行以下命令:

create database samp_db character set gbk;

为了便于在命令提示符下显示中文, 在创建时通过 character set gbk 将数据库字符编码指定为 gbk。创建成功时会得到 Query OK, 1 row affected(0.02 sec) 的响应。

注意: MySQL语句以分号(;)作为语句的结束, 若在语句结尾不添加分号时, 命令提示符会以 -> 提示你继续输入(有个别特例, 但加分号是一定不会错的);

提示: 可以使用 show databases; 命令查看已经创建了哪些数据库。


选择所要操作的数据库
要对一个数据库进行操作, 必须先选择该数据库, 否则会提示错误:

ERROR 1046(3D000): No database selected

两种方式对数据库进行使用的选择:

一: 在登录数据库时指定, 命令: mysql -D 所选择的数据库名 -h 主机名 -u 用户名 -p

例如登录时选择刚刚创建的数据库: mysql -D samp_db -u root -p

二: 在登录后使用 use 语句指定, 命令: use 数据库名;

use 语句可以不加分号, 执行 use samp_db 来选择刚刚创建的数据库, 选择成功后会提示: Database changed

创建数据库表
使用 create table 语句可完成对表的创建, create table 的常见形式:

create table 表名称(列声明);

以创建 students 表为例, 表中将存放 学号(id)、姓名(name)、性别(sex)、年龄(age)、联系电话(tel) 这些内容:

create table students
（
id int unsigned not null auto_increment primary key,
name char(8) not null,
sex char(4) not null,
age tinyint unsigned not null,
tel char(13) null default "-"
);

对于一些较长的语句在命令提示符下可能容易输错, 因此我们可以通过任何文本编辑器将语句输入好后保存为 createtable.sql 的文件中, 通过命令提示符下的文件重定向执行执行该脚本。

打开命令提示符, 输入: mysql -D samp_db -u root -p < createtable.sql

(提示: 1.如果连接远程主机请加上 -h 指令; 2. createtable.sql 文件若不在当前工作目录下需指定文件的完整路径。)

语句解说:

create table tablename(columns) 为创建数据库表的命令, 列的名称以及该列的数据类型将在括号内完成;

括号内声明了5列内容, id、name、sex、age、tel为每列的名称, 后面跟的是数据类型描述, 列与列的描述之间用逗号(,)隔开;

以 "id int unsigned not null auto_increment primary key" 行进行介绍:

"id" 为列的名称;
"int" 指定该列的类型为 int(取值范围为 -8388608到8388607), 在后面我们又用 "unsigned" 加以修饰, 表示该类型为无符号型, 此时该列的取值范围为 0到16777215;
"not null" 说明该列的值不能为空, 必须要填, 如果不指定该属性, 默认可为空;
"auto_increment" 需在整数列中使用, 其作用是在插入数据时若该列为 NULL, MySQL将自动产生一个比现存值更大的唯一标识符值。在每张表中仅能有一个这样的值且所在列必须为索引列。
"primary key" 表示该列是表的主键, 本列的值必须唯一, MySQL将自动索引该列。

下面的 char(8) 表示存储的字符长度为8, tinyint的取值范围为 -127到128, default 属性指定当该列值为空时的默认值。

提示: 1. 使用 show tables; 命令可查看已创建了表的名称; 2. 使用 describe 表名; 命令可查看已创建的表的详细信息。

操作MySQL数据库
向表中插入数据
insert 语句可以用来将一行或多行数据插到数据库表中, 使用的一般形式如下:

insert [into] 表名 [(列名1, 列名2, 列名3, ...)] values (值1, 值2, 值3, ...);

其中 [] 内的内容是可选的, 例如, 要给 samp_db 数据库中的 students 表插入一条记录, 执行语句:

insert into students values(NULL, "王刚", "男", 20, "13811371377");

按回车键确认后若提示 Query Ok, 1 row affected (0.05 sec) 表示数据插入成功。 若插入失败请检查是否已选择需要操作的数据库。

有时我们只需要插入部分数据, 或者不按照列的顺序进行插入, 可以使用这样的形式进行插入:

insert into students (name, sex, age) values("孙丽华", "女", 21);


查询表中的数据
select 语句常用来根据一定的查询规则到数据库中获取数据, 其基本的用法为:

select 列名称 from 表名称 [查询条件];

例如要查询 students 表中所有学生的名字和年龄, 输入语句 select name, age from students; 执行结果如下:

```mysql
mysql> select name, age from students;
```

+--------+-----+
| name | age |
+--------+-----+
| 王刚 | 20 |
| 孙丽华 | 21 |
| 王永恒 | 23 |
| 郑俊杰 | 19 |
| 陈芳 | 22 |
| 张伟朋 | 21 |
+--------+-----+
6 rows in set (0.00 sec)

mysql>
也可以使用通配符 * 查询表中所有的内容, 语句: select * from students;


按特定条件查询:
where 关键词用于指定查询条件, 用法形式为: select 列名称 from 表名称 where 条件;

以查询所有性别为女的信息为例, 输入查询语句: select * from students where sex="女";

where 子句不仅仅支持 "where 列名 = 值" 这种名等于值的查询形式, 对一般的比较运算的运算符都是支持的, 例如 =、>、<、>=、<、!= 以及一些扩展运算符 is [not] null、in、like 等等。 还可以对查询条件使用 or 和 and 进行组合查询, 以后还会学到更加高级的条件查询方式, 这里不再多做介绍。

示例:

查询年龄在21岁以上的所有人信息: select * from students where age > 21;

查询名字中带有 "王" 字的所有人信息: select * from students where name like "%王%";

查询id小于5且年龄大于20的所有人信息: select * from students where id<5 and age>20;


更新表中的数据
update 语句可用来修改表中的数据, 基本的使用形式为:

update 表名称 set 列名称=新值 where 更新条件;

使用示例:

将id为5的手机号改为默认的"-": update students set tel=default where id=5;

将所有人的年龄增加1: update students set age=age+1;

将手机号为 13288097888 的姓名改为 "张伟鹏", 年龄改为 19: update students set name="张伟鹏", age=19 where tel="13288097888";


删除表中的数据
delete 语句用于删除表中的数据, 基本用法为:

delete from 表名称 where 删除条件;

使用示例:

删除id为2的行: delete from students where id=2;

删除所有年龄小于21岁的数据: delete from students where age<20;

删除表中的所有数据: delete from students;


创建后表的修改
alter table 语句用于创建后对表的修改, 基础用法如下:

添加列
基本形式: alter table 表名 add 列名 列数据类型 [after 插入位置];

示例:

在表的最后追加列 address: alter table students add address char(60);

在名为 age 的列后插入列 birthday: alter table students add birthday date after age;

修改列
基本形式: alter table 表名 change 列名称 列新名称 新数据类型;

示例:

将表 tel 列改名为 telphone: alter table students change tel telphone char(13) default "-";

将 name 列的数据类型改为 char(16): alter table students change name name char(16) not null;

删除列
基本形式: alter table 表名 drop 列名称;

示例:

删除 birthday 列: alter table students drop birthday;

重命名表
基本形式: alter table 表名 rename 新表名;

示例:

重命名 students 表为 workmates: alter table students rename workmates;

删除整张表
基本形式: drop table 表名;

示例: 删除 workmates 表: drop table workmates;

删除整个数据库
基本形式: drop database 数据库名;

示例: 删除 samp_db 数据库: drop database samp_db;

附录
修改 root 用户密码
按照本文的安装方式, root 用户默认是没有密码的, 重设 root 密码的方式也较多, 这里仅介绍一种较常用的方式。

使用 mysqladmin 方式:

打开命令提示符界面, 执行命令: mysqladmin -u root -p password 新密码

执行后提示输入旧密码完成密码修改, 当旧密码为空时直接按回车键确认即可。

可视化管理工具 MySQL Workbench
尽管我们可以在命令提示符下通过一行行的输入或者通过重定向文件来执行mysql语句, 但该方式效率较低, 由于没有执行前的语法自动检查, 输入失误造成的一些错误的可能性会大大增加, 这时不妨试试一些可视化的MySQL数据库管理工具, MySQL Workbench 就是 MySQL 官方 为 MySQL 提供的一款可视化管理工具, 你可以在里面通过可视化的方式直接管理数据库中的内容, 并且 MySQL Workbench 的 SQL 脚本编辑器支持语法高亮以及输入时的语法检查, 当然, 它的功能强大, 绝不仅限于这两点。

MySQL Workbench官方介绍: http://www.mysql.com/products/workbench/

MySQL Workbench 下载页: http://dev.mysql.com/downloads/tools/workbench/

下面是在linux下的常用指令：

Mysql安装目录

数据库目录

/var/lib/mysql/

配置文件

/usr/share/mysql（mysql.server命令及配置文件）

相关命令

/usr/bin(mysqladmin mysqldump等命令)

启动脚本

/etc/init.d/mysql（启动脚本文件mysql的目录）

---



## 建表约束

### 主键约束

```mysql
-- 主键约束
-- 使某个字段不重复且不得为空，确保表内所有数据的唯一性。
CREATE TABLE user (
    id INT PRIMARY KEY,
    name VARCHAR(20)
);

-- 联合主键
-- 联合主键中的每个字段都不能为空，并且加起来不能和已设置的联合主键重复。
CREATE TABLE user (
    id INT,
    name VARCHAR(20),
    password VARCHAR(20),
    PRIMARY KEY(id, name)
);

-- 自增约束
-- 自增约束的主键由系统自动递增分配。
CREATE TABLE user (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(20)
);

-- 添加主键约束
-- 如果忘记设置主键，还可以通过SQL语句设置（两种方式）：
ALTER TABLE user ADD PRIMARY KEY(id);
ALTER TABLE user MODIFY id INT PRIMARY KEY;

-- 删除主键
ALTER TABLE user drop PRIMARY KEY;
```

### 唯一主键

```mysql
-- 建表时创建唯一主键
CREATE TABLE user (
    id INT,
    name VARCHAR(20),
    UNIQUE(name)
);

-- 添加唯一主键
-- 如果建表时没有设置唯一建，还可以通过SQL语句设置（两种方式）：
ALTER TABLE user ADD UNIQUE(name);
ALTER TABLE user MODIFY name VARCHAR(20) UNIQUE;

-- 删除唯一主键
ALTER TABLE user DROP INDEX name;
```

### 非空约束

```
-- 建表时添加非空约束
-- 约束某个字段不能为空
CREATE TABLE user (
    id INT,
    name VARCHAR(20) NOT NULL
);

-- 移除非空约束
ALTER TABLE user MODIFY name VARCHAR(20);
```

### 默认约束

```
-- 建表时添加默认约束
-- 约束某个字段的默认值
CREATE TABLE user2 (
    id INT,
    name VARCHAR(20),
    age INT DEFAULT 10
);

-- 移除非空约束
ALTER TABLE user MODIFY age INT;
```

### 外键约束

```
-- 班级
CREATE TABLE classes (
    id INT PRIMARY KEY,
    name VARCHAR(20)
);

-- 学生表
CREATE TABLE students (
    id INT PRIMARY KEY,
    name VARCHAR(20),
    -- 这里的 class_id 要和 classes 中的 id 字段相关联
    class_id INT,
    -- 表示 class_id 的值必须来自于 classes 中的 id 字段值
    FOREIGN KEY(class_id) REFERENCES classes(id)
);

-- 1. 主表（父表）classes 中没有的数据值，在副表（子表）students 中，是不可以使用的；
-- 2. 主表中的记录被副表引用时，主表不可以被删除。
```

## 数据库的三大设计范式

### 1NF

只要字段值还可以继续拆分，就不满足第一范式。

范式设计得越详细，对某些实际操作可能会更好，但并非都有好处，需要对项目的实际情况进行设定。

### 2NF

在满足第一范式的前提下，其他列都必须完全依赖于主键列。如果出现不完全依赖，只可能发生在联合主键的情况下：

```
-- 订单表
CREATE TABLE myorder (
    product_id INT,
    customer_id INT,
    product_name VARCHAR(20),
    customer_name VARCHAR(20),
    PRIMARY KEY (product_id, customer_id)
);
```

实际上，在这张订单表中，`product_name` 只依赖于 `product_id` ，`customer_name` 只依赖于 `customer_id` 。也就是说，`product_name` 和 `customer_id` 是没用关系的，`customer_name` 和 `product_id` 也是没有关系的。

这就不满足第二范式：其他列都必须完全依赖于主键列！

```
CREATE TABLE myorder (
    order_id INT PRIMARY KEY,
    product_id INT,
    customer_id INT
);

CREATE TABLE product (
    id INT PRIMARY KEY,
    name VARCHAR(20)
);

CREATE TABLE customer (
    id INT PRIMARY KEY,
    name VARCHAR(20)
);
```

拆分之后，`myorder` 表中的 `product_id` 和 `customer_id` 完全依赖于 `order_id` 主键，而 `product` 和 `customer` 表中的其他字段又完全依赖于主键。满足了第二范式的设计！

### 3NF

在满足第二范式的前提下，除了主键列之外，其他列之间不能有传递依赖关系。

```
CREATE TABLE myorder (
    order_id INT PRIMARY KEY,
    product_id INT,
    customer_id INT,
    customer_phone VARCHAR(15)
);
```

表中的 `customer_phone` 有可能依赖于 `order_id` 、 `customer_id` 两列，也就不满足了第三范式的设计：其他列之间不能有传递依赖关系。

```
CREATE TABLE myorder (
    order_id INT PRIMARY KEY,
    product_id INT,
    customer_id INT
);

CREATE TABLE customer (
    id INT PRIMARY KEY,
    name VARCHAR(20),
    phone VARCHAR(15)
);
```

修改后就不存在其他列之间的传递依赖关系，其他列都只依赖于主键列，满足了第三范式的设计！

## 查询练习

### 准备数据

```
-- 创建数据库
CREATE DATABASE select_test;
-- 切换数据库
USE select_test;

-- 创建学生表
CREATE TABLE student (
    no VARCHAR(20) PRIMARY KEY,
    name VARCHAR(20) NOT NULL,
    sex VARCHAR(10) NOT NULL,
    birthday DATE, -- 生日
    class VARCHAR(20) -- 所在班级
);

-- 创建教师表
CREATE TABLE teacher (
    no VARCHAR(20) PRIMARY KEY,
    name VARCHAR(20) NOT NULL,
    sex VARCHAR(10) NOT NULL,
    birthday DATE,
    profession VARCHAR(20) NOT NULL, -- 职称
    department VARCHAR(20) NOT NULL -- 部门
);

-- 创建课程表
CREATE TABLE course (
    no VARCHAR(20) PRIMARY KEY,
    name VARCHAR(20) NOT NULL,
    t_no VARCHAR(20) NOT NULL, -- 教师编号
    -- 表示该 tno 来自于 teacher 表中的 no 字段值
    FOREIGN KEY(t_no) REFERENCES teacher(no) 
);

-- 成绩表
CREATE TABLE score (
    s_no VARCHAR(20) NOT NULL, -- 学生编号
    c_no VARCHAR(20) NOT NULL, -- 课程号
    degree DECIMAL,	-- 成绩
    -- 表示该 s_no, c_no 分别来自于 student, course 表中的 no 字段值
    FOREIGN KEY(s_no) REFERENCES student(no),	
    FOREIGN KEY(c_no) REFERENCES course(no),
    -- 设置 s_no, c_no 为联合主键
    PRIMARY KEY(s_no, c_no)
);

-- 查看所有表
SHOW TABLES;

-- 添加学生表数据
INSERT INTO student VALUES('101', '曾华', '男', '1977-09-01', '95033');
INSERT INTO student VALUES('102', '匡明', '男', '1975-10-02', '95031');
INSERT INTO student VALUES('103', '王丽', '女', '1976-01-23', '95033');
INSERT INTO student VALUES('104', '李军', '男', '1976-02-20', '95033');
INSERT INTO student VALUES('105', '王芳', '女', '1975-02-10', '95031');
INSERT INTO student VALUES('106', '陆军', '男', '1974-06-03', '95031');
INSERT INTO student VALUES('107', '王尼玛', '男', '1976-02-20', '95033');
INSERT INTO student VALUES('108', '张全蛋', '男', '1975-02-10', '95031');
INSERT INTO student VALUES('109', '赵铁柱', '男', '1974-06-03', '95031');

-- 添加教师表数据
INSERT INTO teacher VALUES('804', '李诚', '男', '1958-12-02', '副教授', '计算机系');
INSERT INTO teacher VALUES('856', '张旭', '男', '1969-03-12', '讲师', '电子工程系');
INSERT INTO teacher VALUES('825', '王萍', '女', '1972-05-05', '助教', '计算机系');
INSERT INTO teacher VALUES('831', '刘冰', '女', '1977-08-14', '助教', '电子工程系');

-- 添加课程表数据
INSERT INTO course VALUES('3-105', '计算机导论', '825');
INSERT INTO course VALUES('3-245', '操作系统', '804');
INSERT INTO course VALUES('6-166', '数字电路', '856');
INSERT INTO course VALUES('9-888', '高等数学', '831');

-- 添加添加成绩表数据
INSERT INTO score VALUES('103', '3-105', '92');
INSERT INTO score VALUES('103', '3-245', '86');
INSERT INTO score VALUES('103', '6-166', '85');
INSERT INTO score VALUES('105', '3-105', '88');
INSERT INTO score VALUES('105', '3-245', '75');
INSERT INTO score VALUES('105', '6-166', '79');
INSERT INTO score VALUES('109', '3-105', '76');
INSERT INTO score VALUES('109', '3-245', '68');
INSERT INTO score VALUES('109', '6-166', '81');

-- 查看表结构
SELECT * FROM course;
SELECT * FROM score;
SELECT * FROM student;
SELECT * FROM teacher;
```

### 1 到 10

```
-- 查询 student 表的所有行
SELECT * FROM student;

-- 查询 student 表中的 name、sex 和 class 字段的所有行
SELECT name, sex, class FROM student;

-- 查询 teacher 表中不重复的 department 列
-- department: 去重查询
SELECT DISTINCT department FROM teacher;

-- 查询 score 表中成绩在60-80之间的所有行（区间查询和运算符查询）
-- BETWEEN xx AND xx: 查询区间, AND 表示 "并且"
SELECT * FROM score WHERE degree BETWEEN 60 AND 80;
SELECT * FROM score WHERE degree > 60 AND degree < 80;

-- 查询 score 表中成绩为 85, 86 或 88 的行
-- IN: 查询规定中的多个值
SELECT * FROM score WHERE degree IN (85, 86, 88);

-- 查询 student 表中 '95031' 班或性别为 '女' 的所有行
-- or: 表示或者关系
SELECT * FROM student WHERE class = '95031' or sex = '女';

-- 以 class 降序的方式查询 student 表的所有行
-- DESC: 降序，从高到低
-- ASC（默认）: 升序，从低到高
SELECT * FROM student ORDER BY class DESC;
SELECT * FROM student ORDER BY class ASC;

-- 以 c_no 升序、degree 降序查询 score 表的所有行
SELECT * FROM score ORDER BY c_no ASC, degree DESC;

-- 查询 "95031" 班的学生人数
-- COUNT: 统计
SELECT COUNT(*) FROM student WHERE class = '95031';

-- 查询 score 表中的最高分的学生学号和课程编号（子查询或排序查询）。
-- (SELECT MAX(degree) FROM score): 子查询，算出最高分
SELECT s_no, c_no FROM score WHERE degree = (SELECT MAX(degree) FROM score);

--  排序查询
-- LIMIT r, n: 表示从第r行开始，查询n条数据
SELECT s_no, c_no, degree FROM score ORDER BY degree DESC LIMIT 0, 1;
```

### 分组计算平均成绩

**查询每门课的平均成绩。**

```
-- AVG: 平均值
SELECT AVG(degree) FROM score WHERE c_no = '3-105';
SELECT AVG(degree) FROM score WHERE c_no = '3-245';
SELECT AVG(degree) FROM score WHERE c_no = '6-166';

-- GROUP BY: 分组查询
SELECT c_no, AVG(degree) FROM score GROUP BY c_no;
```

### 分组条件与模糊查询

**查询 `score` 表中至少有 2 名学生选修，并以 3 开头的课程的平均分数。**

```
SELECT * FROM score;
-- c_no 课程编号
+------+-------+--------+
| s_no | c_no  | degree |
+------+-------+--------+
| 103  | 3-105 |     92 |
| 103  | 3-245 |     86 |
| 103  | 6-166 |     85 |
| 105  | 3-105 |     88 |
| 105  | 3-245 |     75 |
| 105  | 6-166 |     79 |
| 109  | 3-105 |     76 |
| 109  | 3-245 |     68 |
| 109  | 6-166 |     81 |
+------+-------+--------+
```

分析表发现，至少有 2 名学生选修的课程是 `3-105` 、`3-245` 、`6-166` ，以 3 开头的课程是 `3-105` 、`3-245` 。也就是说，我们要查询所有 `3-105` 和 `3-245` 的 `degree` 平均分。

```
-- 首先把 c_no, AVG(degree) 通过分组查询出来
SELECT c_no, AVG(degree) FROM score GROUP BY c_no
+-------+-------------+
| c_no  | AVG(degree) |
+-------+-------------+
| 3-105 |     85.3333 |
| 3-245 |     76.3333 |
| 6-166 |     81.6667 |
+-------+-------------+

-- 再查询出至少有 2 名学生选修的课程
-- HAVING: 表示持有
HAVING COUNT(c_no) >= 2

-- 并且是以 3 开头的课程
-- LIKE 表示模糊查询，"%" 是一个通配符，匹配 "3" 后面的任意字符。
AND c_no LIKE '3%';

-- 把前面的SQL语句拼接起来，
-- 后面加上一个 COUNT(*)，表示将每个分组的个数也查询出来。
SELECT c_no, AVG(degree), COUNT(*) FROM score GROUP BY c_no
HAVING COUNT(c_no) >= 2 AND c_no LIKE '3%';
+-------+-------------+----------+
| c_no  | AVG(degree) | COUNT(*) |
+-------+-------------+----------+
| 3-105 |     85.3333 |        3 |
| 3-245 |     76.3333 |        3 |
+-------+-------------+----------+
```

### 多表查询 - 1

**查询所有学生的 `name`，以及该学生在 `score` 表中对应的 `c_no` 和 `degree` 。**

```
SELECT no, name FROM student;
+-----+-----------+
| no  | name      |
+-----+-----------+
| 101 | 曾华      |
| 102 | 匡明      |
| 103 | 王丽      |
| 104 | 李军      |
| 105 | 王芳      |
| 106 | 陆军      |
| 107 | 王尼玛    |
| 108 | 张全蛋    |
| 109 | 赵铁柱    |
+-----+-----------+

SELECT s_no, c_no, degree FROM score;
+------+-------+--------+
| s_no | c_no  | degree |
+------+-------+--------+
| 103  | 3-105 |     92 |
| 103  | 3-245 |     86 |
| 103  | 6-166 |     85 |
| 105  | 3-105 |     88 |
| 105  | 3-245 |     75 |
| 105  | 6-166 |     79 |
| 109  | 3-105 |     76 |
| 109  | 3-245 |     68 |
| 109  | 6-166 |     81 |
+------+-------+--------+
```

通过分析可以发现，只要把 `score` 表中的 `s_no` 字段值替换成 `student` 表中对应的 `name` 字段值就可以了，如何做呢？

```
-- FROM...: 表示从 student, score 表中查询
-- WHERE 的条件表示为，只有在 student.no 和 score.s_no 相等时才显示出来。
SELECT name, c_no, degree FROM student, score 
WHERE student.no = score.s_no;
+-----------+-------+--------+
| name      | c_no  | degree |
+-----------+-------+--------+
| 王丽      | 3-105 |     92 |
| 王丽      | 3-245 |     86 |
| 王丽      | 6-166 |     85 |
| 王芳      | 3-105 |     88 |
| 王芳      | 3-245 |     75 |
| 王芳      | 6-166 |     79 |
| 赵铁柱    | 3-105 |     76 |
| 赵铁柱    | 3-245 |     68 |
| 赵铁柱    | 6-166 |     81 |
+-----------+-------+--------+
```

### 多表查询 - 2

**查询所有学生的 `no` 、课程名称 ( `course` 表中的 `name` ) 和成绩 ( `score` 表中的 `degree` ) 列。**

只有 `score` 关联学生的 `no` ，因此只要查询 `score` 表，就能找出所有和学生相关的 `no` 和 `degree` ：

```
SELECT s_no, c_no, degree FROM score;
+------+-------+--------+
| s_no | c_no  | degree |
+------+-------+--------+
| 103  | 3-105 |     92 |
| 103  | 3-245 |     86 |
| 103  | 6-166 |     85 |
| 105  | 3-105 |     88 |
| 105  | 3-245 |     75 |
| 105  | 6-166 |     79 |
| 109  | 3-105 |     76 |
| 109  | 3-245 |     68 |
| 109  | 6-166 |     81 |
+------+-------+--------+
```

然后查询 `course` 表：

```
+-------+-----------------+
| no    | name            |
+-------+-----------------+
| 3-105 | 计算机导论      |
| 3-245 | 操作系统        |
| 6-166 | 数字电路        |
| 9-888 | 高等数学        |
+-------+-----------------+
```

只要把 `score` 表中的 `c_no` 替换成 `course` 表中对应的 `name` 字段值就可以了。

```
-- 增加一个查询字段 name，分别从 score、course 这两个表中查询。
-- as 表示取一个该字段的别名。
SELECT s_no, name as c_name, degree FROM score, course
WHERE score.c_no = course.no;
+------+-----------------+--------+
| s_no | c_name          | degree |
+------+-----------------+--------+
| 103  | 计算机导论      |     92 |
| 105  | 计算机导论      |     88 |
| 109  | 计算机导论      |     76 |
| 103  | 操作系统        |     86 |
| 105  | 操作系统        |     75 |
| 109  | 操作系统        |     68 |
| 103  | 数字电路        |     85 |
| 105  | 数字电路        |     79 |
| 109  | 数字电路        |     81 |
+------+-----------------+--------+
```

### 三表关联查询

**查询所有学生的 `name` 、课程名 ( `course` 表中的 `name` ) 和 `degree` 。**

只有 `score` 表中关联学生的学号和课堂号，我们只要围绕着 `score` 这张表查询就好了。

```
SELECT * FROM score;
+------+-------+--------+
| s_no | c_no  | degree |
+------+-------+--------+
| 103  | 3-105 |     92 |
| 103  | 3-245 |     86 |
| 103  | 6-166 |     85 |
| 105  | 3-105 |     88 |
| 105  | 3-245 |     75 |
| 105  | 6-166 |     79 |
| 109  | 3-105 |     76 |
| 109  | 3-245 |     68 |
| 109  | 6-166 |     81 |
+------+-------+--------+
```

只要把 `s_no` 和 `c_no` 替换成 `student` 和 `srouse` 表中对应的 `name` 字段值就好了。

首先把 `s_no` 替换成 `student` 表中的 `name` 字段：

```
SELECT name, c_no, degree FROM student, score WHERE student.no = score.s_no;
+-----------+-------+--------+
| name      | c_no  | degree |
+-----------+-------+--------+
| 王丽      | 3-105 |     92 |
| 王丽      | 3-245 |     86 |
| 王丽      | 6-166 |     85 |
| 王芳      | 3-105 |     88 |
| 王芳      | 3-245 |     75 |
| 王芳      | 6-166 |     79 |
| 赵铁柱    | 3-105 |     76 |
| 赵铁柱    | 3-245 |     68 |
| 赵铁柱    | 6-166 |     81 |
+-----------+-------+--------+
```

再把 `c_no` 替换成 `course` 表中的 `name` 字段：

```
-- 课程表
SELECT no, name FROM course;
+-------+-----------------+
| no    | name            |
+-------+-----------------+
| 3-105 | 计算机导论      |
| 3-245 | 操作系统        |
| 6-166 | 数字电路        |
| 9-888 | 高等数学        |
+-------+-----------------+

-- 由于字段名存在重复，使用 "表名.字段名 as 别名" 代替。
SELECT student.name as s_name, course.name as c_name, degree 
FROM student, score, course
WHERE student.NO = score.s_no
AND score.c_no = course.no;
```

### 子查询加分组求平均分

**查询 `95031` 班学生每门课程的平均成绩。**

在 `score` 表中根据 `student` 表的学生编号筛选出学生的课堂号和成绩：

```
-- IN (..): 将筛选出的学生号当做 s_no 的条件查询
SELECT s_no, c_no, degree FROM score
WHERE s_no IN (SELECT no FROM student WHERE class = '95031');
+------+-------+--------+
| s_no | c_no  | degree |
+------+-------+--------+
| 105  | 3-105 |     88 |
| 105  | 3-245 |     75 |
| 105  | 6-166 |     79 |
| 109  | 3-105 |     76 |
| 109  | 3-245 |     68 |
| 109  | 6-166 |     81 |
+------+-------+--------+
```

这时只要将 `c_no` 分组一下就能得出 `95031` 班学生每门课的平均成绩：

```
SELECT c_no, AVG(degree) FROM score
WHERE s_no IN (SELECT no FROM student WHERE class = '95031')
GROUP BY c_no;
+-------+-------------+
| c_no  | AVG(degree) |
+-------+-------------+
| 3-105 |     82.0000 |
| 3-245 |     71.5000 |
| 6-166 |     80.0000 |
+-------+-------------+
```

### 子查询 - 1

**查询在 `3-105` 课程中，所有成绩高于 `109` 号同学的记录。**

首先筛选出课堂号为 `3-105` ，在找出所有成绩高于 `109` 号同学的的行。

```
SELECT * FROM score 
WHERE c_no = '3-105'
AND degree > (SELECT degree FROM score WHERE s_no = '109' AND c_no = '3-105');
```

### 子查询 - 2

**查询所有成绩高于 `109` 号同学的 `3-105` 课程成绩记录。**

```
-- 不限制课程号，只要成绩大于109号同学的3-105课程成绩就可以。
SELECT * FROM score
WHERE degree > (SELECT degree FROM score WHERE s_no = '109' AND c_no = '3-105');
```

### YEAR 函数与带 IN 关键字查询

**查询所有和 `101` 、`108` 号学生同年出生的 `no` 、`name` 、`birthday` 列。**

```
-- YEAR(..): 取出日期中的年份
SELECT no, name, birthday FROM student
WHERE YEAR(birthday) IN (SELECT YEAR(birthday) FROM student WHERE no IN (101, 108));
```

### 多层嵌套子查询

**查询 `'张旭'` 教师任课的学生成绩表。**

首先找到教师编号：

```
SELECT NO FROM teacher WHERE NAME = '张旭'
```

通过 `sourse` 表找到该教师课程号：

```
SELECT NO FROM course WHERE t_no = ( SELECT NO FROM teacher WHERE NAME = '张旭' );
```

通过筛选出的课程号查询成绩表：

```
SELECT * FROM score WHERE c_no = (
    SELECT no FROM course WHERE t_no = ( 
        SELECT no FROM teacher WHERE NAME = '张旭' 
    )
);
```

### 多表查询

**查询某选修课程多于5个同学的教师姓名。**

首先在 `teacher` 表中，根据 `no` 字段来判断该教师的同一门课程是否有至少5名学员选修：

```
-- 查询 teacher 表
SELECT no, name FROM teacher;
+-----+--------+
| no  | name   |
+-----+--------+
| 804 | 李诚   |
| 825 | 王萍   |
| 831 | 刘冰   |
| 856 | 张旭   |
+-----+--------+

SELECT name FROM teacher WHERE no IN (
    -- 在这里找到对应的条件
);
```

查看和教师编号有有关的表的信息：

```
SELECT * FROM course;
-- t_no: 教师编号
+-------+-----------------+------+
| no    | name            | t_no |
+-------+-----------------+------+
| 3-105 | 计算机导论      | 825  |
| 3-245 | 操作系统        | 804  |
| 6-166 | 数字电路        | 856  |
| 9-888 | 高等数学        | 831  |
+-------+-----------------+------+
```

我们已经找到和教师编号有关的字段就在 `course` 表中，但是还无法知道哪门课程至少有5名学生选修，所以还需要根据 `score` 表来查询：

```
-- 在此之前向 score 插入一些数据，以便丰富查询条件。
INSERT INTO score VALUES ('101', '3-105', '90');
INSERT INTO score VALUES ('102', '3-105', '91');
INSERT INTO score VALUES ('104', '3-105', '89');

-- 查询 score 表
SELECT * FROM score;
+------+-------+--------+
| s_no | c_no  | degree |
+------+-------+--------+
| 101  | 3-105 |     90 |
| 102  | 3-105 |     91 |
| 103  | 3-105 |     92 |
| 103  | 3-245 |     86 |
| 103  | 6-166 |     85 |
| 104  | 3-105 |     89 |
| 105  | 3-105 |     88 |
| 105  | 3-245 |     75 |
| 105  | 6-166 |     79 |
| 109  | 3-105 |     76 |
| 109  | 3-245 |     68 |
| 109  | 6-166 |     81 |
+------+-------+--------+

-- 在 score 表中将 c_no 作为分组，并且限制 c_no 持有至少 5 条数据。
SELECT c_no FROM score GROUP BY c_no HAVING COUNT(*) > 5;
+-------+
| c_no  |
+-------+
| 3-105 |
+-------+
```

根据筛选出来的课程号，找出在某课程中，拥有至少5名学员的教师编号：

```
SELECT t_no FROM course WHERE no IN (
    SELECT c_no FROM score GROUP BY c_no HAVING COUNT(*) > 5
);
+------+
| t_no |
+------+
| 825  |
+------+
```

在 `teacher` 表中，根据筛选出来的教师编号找到教师姓名：

```
SELECT name FROM teacher WHERE no IN (
    -- 最终条件
    SELECT t_no FROM course WHERE no IN (
        SELECT c_no FROM score GROUP BY c_no HAVING COUNT(*) > 5
    )
);
```

### 子查询 - 3

**查询 “计算机系” 课程的成绩表。**

思路是，先找出 `course` 表中所有 `计算机系` 课程的编号，然后根据这个编号查询 `score` 表。

```
-- 通过 teacher 表查询所有 `计算机系` 的教师编号
SELECT no, name, department FROM teacher WHERE department = '计算机系'
+-----+--------+--------------+
| no  | name   | department   |
+-----+--------+--------------+
| 804 | 李诚   | 计算机系     |
| 825 | 王萍   | 计算机系     |
+-----+--------+--------------+

-- 通过 course 表查询该教师的课程编号
SELECT no FROM course WHERE t_no IN (
    SELECT no FROM teacher WHERE department = '计算机系'
);
+-------+
| no    |
+-------+
| 3-245 |
| 3-105 |
+-------+

-- 根据筛选出来的课程号查询成绩表
SELECT * FROM score WHERE c_no IN (
    SELECT no FROM course WHERE t_no IN (
        SELECT no FROM teacher WHERE department = '计算机系'
    )
);
+------+-------+--------+
| s_no | c_no  | degree |
+------+-------+--------+
| 103  | 3-245 |     86 |
| 105  | 3-245 |     75 |
| 109  | 3-245 |     68 |
| 101  | 3-105 |     90 |
| 102  | 3-105 |     91 |
| 103  | 3-105 |     92 |
| 104  | 3-105 |     89 |
| 105  | 3-105 |     88 |
| 109  | 3-105 |     76 |
+------+-------+--------+
```

### UNION 和 NOTIN 的使用

**查询 `计算机系` 与 `电子工程系` 中的不同职称的教师。**

```
-- NOT: 代表逻辑非
SELECT * FROM teacher WHERE department = '计算机系' AND profession NOT IN (
    SELECT profession FROM teacher WHERE department = '电子工程系'
)
-- 合并两个集
UNION
SELECT * FROM teacher WHERE department = '电子工程系' AND profession NOT IN (
    SELECT profession FROM teacher WHERE department = '计算机系'
);
```

### ANY 表示至少一个 - DESC ( 降序 )

**查询课程 `3-105` 且成绩 至少 高于 `3-245` 的 `score` 表。**

```
SELECT * FROM score WHERE c_no = '3-105';
+------+-------+--------+
| s_no | c_no  | degree |
+------+-------+--------+
| 101  | 3-105 |     90 |
| 102  | 3-105 |     91 |
| 103  | 3-105 |     92 |
| 104  | 3-105 |     89 |
| 105  | 3-105 |     88 |
| 109  | 3-105 |     76 |
+------+-------+--------+

SELECT * FROM score WHERE c_no = '3-245';
+------+-------+--------+
| s_no | c_no  | degree |
+------+-------+--------+
| 103  | 3-245 |     86 |
| 105  | 3-245 |     75 |
| 109  | 3-245 |     68 |
+------+-------+--------+

-- ANY: 符合SQL语句中的任意条件。
-- 也就是说，在 3-105 成绩中，只要有一个大于从 3-245 筛选出来的任意行就符合条件，
-- 最后根据降序查询结果。
SELECT * FROM score WHERE c_no = '3-105' AND degree > ANY(
    SELECT degree FROM score WHERE c_no = '3-245'
) ORDER BY degree DESC;
+------+-------+--------+
| s_no | c_no  | degree |
+------+-------+--------+
| 103  | 3-105 |     92 |
| 102  | 3-105 |     91 |
| 101  | 3-105 |     90 |
| 104  | 3-105 |     89 |
| 105  | 3-105 |     88 |
| 109  | 3-105 |     76 |
+------+-------+--------+
```

### 表示所有的 ALL

**查询课程 `3-105` 且成绩高于 `3-245` 的 `score` 表。**

```
-- 只需对上一道题稍作修改。
-- ALL: 符合SQL语句中的所有条件。
-- 也就是说，在 3-105 每一行成绩中，都要大于从 3-245 筛选出来全部行才算符合条件。
SELECT * FROM score WHERE c_no = '3-105' AND degree > ALL(
    SELECT degree FROM score WHERE c_no = '3-245'
);
+------+-------+--------+
| s_no | c_no  | degree |
+------+-------+--------+
| 101  | 3-105 |     90 |
| 102  | 3-105 |     91 |
| 103  | 3-105 |     92 |
| 104  | 3-105 |     89 |
| 105  | 3-105 |     88 |
+------+-------+--------+
```

### 复制表的数据作为条件查询

**查询某课程成绩比该课程平均成绩低的 `score` 表。**

```
-- 查询平均分
SELECT c_no, AVG(degree) FROM score GROUP BY c_no;
+-------+-------------+
| c_no  | AVG(degree) |
+-------+-------------+
| 3-105 |     87.6667 |
| 3-245 |     76.3333 |
| 6-166 |     81.6667 |
+-------+-------------+

-- 查询 score 表
SELECT degree FROM score;
+--------+
| degree |
+--------+
|     90 |
|     91 |
|     92 |
|     86 |
|     85 |
|     89 |
|     88 |
|     75 |
|     79 |
|     76 |
|     68 |
|     81 |
+--------+

-- 将表 b 作用于表 a 中查询数据
-- score a (b): 将表声明为 a (b)，
-- 如此就能用 a.c_no = b.c_no 作为条件执行查询了。
SELECT * FROM score a WHERE degree < (
    (SELECT AVG(degree) FROM score b WHERE a.c_no = b.c_no)
);
+------+-------+--------+
| s_no | c_no  | degree |
+------+-------+--------+
| 105  | 3-245 |     75 |
| 105  | 6-166 |     79 |
| 109  | 3-105 |     76 |
| 109  | 3-245 |     68 |
| 109  | 6-166 |     81 |
+------+-------+--------+
```

### 子查询 - 4

**查询所有任课 ( 在 `course` 表里有课程 ) 教师的 `name` 和 `department`** 。

```
SELECT name, department FROM teacher WHERE no IN (SELECT t_no FROM course);
+--------+-----------------+
| name   | department      |
+--------+-----------------+
| 李诚   | 计算机系        |
| 王萍   | 计算机系        |
| 刘冰   | 电子工程系      |
| 张旭   | 电子工程系      |
+--------+-----------------+
```

### 条件加组筛选

**查询 `student` 表中至少有 2 名男生的 `class` 。**

```
-- 查看学生表信息
SELECT * FROM student;
+-----+-----------+-----+------------+-------+
| no  | name      | sex | birthday   | class |
+-----+-----------+-----+------------+-------+
| 101 | 曾华      | 男  | 1977-09-01 | 95033 |
| 102 | 匡明      | 男  | 1975-10-02 | 95031 |
| 103 | 王丽      | 女  | 1976-01-23 | 95033 |
| 104 | 李军      | 男  | 1976-02-20 | 95033 |
| 105 | 王芳      | 女  | 1975-02-10 | 95031 |
| 106 | 陆军      | 男  | 1974-06-03 | 95031 |
| 107 | 王尼玛    | 男  | 1976-02-20 | 95033 |
| 108 | 张全蛋    | 男  | 1975-02-10 | 95031 |
| 109 | 赵铁柱    | 男  | 1974-06-03 | 95031 |
| 110 | 张飞      | 男  | 1974-06-03 | 95038 |
+-----+-----------+-----+------------+-------+

-- 只查询性别为男，然后按 class 分组，并限制 class 行大于 1。
SELECT class FROM student WHERE sex = '男' GROUP BY class HAVING COUNT(*) > 1;
+-------+
| class |
+-------+
| 95033 |
| 95031 |
+-------+
```

### NOTLIKE 模糊查询取反

**查询 `student` 表中不姓 "王" 的同学记录。**

```
-- NOT: 取反
-- LIKE: 模糊查询
mysql> SELECT * FROM student WHERE name NOT LIKE '王%';
+-----+-----------+-----+------------+-------+
| no  | name      | sex | birthday   | class |
+-----+-----------+-----+------------+-------+
| 101 | 曾华      | 男  | 1977-09-01 | 95033 |
| 102 | 匡明      | 男  | 1975-10-02 | 95031 |
| 104 | 李军      | 男  | 1976-02-20 | 95033 |
| 106 | 陆军      | 男  | 1974-06-03 | 95031 |
| 108 | 张全蛋    | 男  | 1975-02-10 | 95031 |
| 109 | 赵铁柱    | 男  | 1974-06-03 | 95031 |
| 110 | 张飞      | 男  | 1974-06-03 | 95038 |
+-----+-----------+-----+------------+-------+
```

### YEAR 与 NOW 函数

**查询 `student` 表中每个学生的姓名和年龄。**

```
-- 使用函数 YEAR(NOW()) 计算出当前年份，减去出生年份后得出年龄。
SELECT name, YEAR(NOW()) - YEAR(birthday) as age FROM student;
+-----------+------+
| name      | age  |
+-----------+------+
| 曾华      |   42 |
| 匡明      |   44 |
| 王丽      |   43 |
| 李军      |   43 |
| 王芳      |   44 |
| 陆军      |   45 |
| 王尼玛    |   43 |
| 张全蛋    |   44 |
| 赵铁柱    |   45 |
| 张飞      |   45 |
+-----------+------+
```

### MAX 与 MIN 函数

**查询 `student` 表中最大和最小的 `birthday` 值。**

```
SELECT MAX(birthday), MIN(birthday) FROM student;
+---------------+---------------+
| MAX(birthday) | MIN(birthday) |
+---------------+---------------+
| 1977-09-01    | 1974-06-03    |
+---------------+---------------+
```

### 多段排序

**以 `class` 和 `birthday` 从大到小的顺序查询 `student` 表。**

```
SELECT * FROM student ORDER BY class DESC, birthday;
+-----+-----------+-----+------------+-------+
| no  | name      | sex | birthday   | class |
+-----+-----------+-----+------------+-------+
| 110 | 张飞      | 男  | 1974-06-03 | 95038 |
| 103 | 王丽      | 女  | 1976-01-23 | 95033 |
| 104 | 李军      | 男  | 1976-02-20 | 95033 |
| 107 | 王尼玛    | 男  | 1976-02-20 | 95033 |
| 101 | 曾华      | 男  | 1977-09-01 | 95033 |
| 106 | 陆军      | 男  | 1974-06-03 | 95031 |
| 109 | 赵铁柱    | 男  | 1974-06-03 | 95031 |
| 105 | 王芳      | 女  | 1975-02-10 | 95031 |
| 108 | 张全蛋    | 男  | 1975-02-10 | 95031 |
| 102 | 匡明      | 男  | 1975-10-02 | 95031 |
+-----+-----------+-----+------------+-------+
```

### 子查询 - 5

**查询 "男" 教师及其所上的课程。**

```
SELECT * FROM course WHERE t_no in (SELECT no FROM teacher WHERE sex = '男');
+-------+--------------+------+
| no    | name         | t_no |
+-------+--------------+------+
| 3-245 | 操作系统     | 804  |
| 6-166 | 数字电路     | 856  |
+-------+--------------+------+
```

### MAX 函数与子查询

**查询最高分同学的 `score` 表。**

```
-- 找出最高成绩（该查询只能有一个结果）
SELECT MAX(degree) FROM score;

-- 根据上面的条件筛选出所有最高成绩表，
-- 该查询可能有多个结果，假设 degree 值多次符合条件。
SELECT * FROM score WHERE degree = (SELECT MAX(degree) FROM score);
+------+-------+--------+
| s_no | c_no  | degree |
+------+-------+--------+
| 103  | 3-105 |     92 |
+------+-------+--------+
```

### 子查询 - 6

**查询和 "李军" 同性别的所有同学 `name` 。**

```
-- 首先将李军的性别作为条件取出来
SELECT sex FROM student WHERE name = '李军';
+-----+
| sex |
+-----+
| 男  |
+-----+

-- 根据性别查询 name 和 sex
SELECT name, sex FROM student WHERE sex = (
    SELECT sex FROM student WHERE name = '李军'
);
+-----------+-----+
| name      | sex |
+-----------+-----+
| 曾华      | 男  |
| 匡明      | 男  |
| 李军      | 男  |
| 陆军      | 男  |
| 王尼玛    | 男  |
| 张全蛋    | 男  |
| 赵铁柱    | 男  |
| 张飞      | 男  |
+-----------+-----+
```

### 子查询 - 7

**查询和 "李军" 同性别且同班的同学 `name` 。**

```
SELECT name, sex, class FROM student WHERE sex = (
    SELECT sex FROM student WHERE name = '李军'
) AND class = (
    SELECT class FROM student WHERE name = '李军'
);
+-----------+-----+-------+
| name      | sex | class |
+-----------+-----+-------+
| 曾华      | 男  | 95033 |
| 李军      | 男  | 95033 |
| 王尼玛    | 男  | 95033 |
+-----------+-----+-------+
```

### 子查询 - 8

**查询所有选修 "计算机导论" 课程的 "男" 同学成绩表。**

需要的 "计算机导论" 和性别为 "男" 的编号可以在 `course` 和 `student` 表中找到。

```
SELECT * FROM score WHERE c_no = (
    SELECT no FROM course WHERE name = '计算机导论'
) AND s_no IN (
    SELECT no FROM student WHERE sex = '男'
);
+------+-------+--------+
| s_no | c_no  | degree |
+------+-------+--------+
| 101  | 3-105 |     90 |
| 102  | 3-105 |     91 |
| 104  | 3-105 |     89 |
| 109  | 3-105 |     76 |
+------+-------+--------+
```

### 按等级查询

建立一个 `grade` 表代表学生的成绩等级，并插入数据：

```
CREATE TABLE grade (
    low INT(3),
    upp INT(3),
    grade char(1)
);

INSERT INTO grade VALUES (90, 100, 'A');
INSERT INTO grade VALUES (80, 89, 'B');
INSERT INTO grade VALUES (70, 79, 'C');
INSERT INTO grade VALUES (60, 69, 'D');
INSERT INTO grade VALUES (0, 59, 'E');

SELECT * FROM grade;
+------+------+-------+
| low  | upp  | grade |
+------+------+-------+
|   90 |  100 | A     |
|   80 |   89 | B     |
|   70 |   79 | C     |
|   60 |   69 | D     |
|    0 |   59 | E     |
+------+------+-------+
```

**查询所有学生的 `s_no` 、`c_no` 和 `grade` 列。**

思路是，使用区间 ( `BETWEEN` ) 查询，判断学生的成绩 ( `degree` ) 在 `grade` 表的 `low` 和 `upp` 之间。

```
SELECT s_no, c_no, grade FROM score, grade 
WHERE degree BETWEEN low AND upp;
+------+-------+-------+
| s_no | c_no  | grade |
+------+-------+-------+
| 101  | 3-105 | A     |
| 102  | 3-105 | A     |
| 103  | 3-105 | A     |
| 103  | 3-245 | B     |
| 103  | 6-166 | B     |
| 104  | 3-105 | B     |
| 105  | 3-105 | B     |
| 105  | 3-245 | C     |
| 105  | 6-166 | C     |
| 109  | 3-105 | C     |
| 109  | 3-245 | D     |
| 109  | 6-166 | B     |
+------+-------+-------+
```

### 连接查询

准备用于测试连接查询的数据：

```
CREATE DATABASE testJoin;

CREATE TABLE person (
    id INT,
    name VARCHAR(20),
    cardId INT
);

CREATE TABLE card (
    id INT,
    name VARCHAR(20)
);

INSERT INTO card VALUES (1, '饭卡'), (2, '建行卡'), (3, '农行卡'), (4, '工商卡'), (5, '邮政卡');
SELECT * FROM card;
+------+-----------+
| id   | name      |
+------+-----------+
|    1 | 饭卡      |
|    2 | 建行卡    |
|    3 | 农行卡    |
|    4 | 工商卡    |
|    5 | 邮政卡    |
+------+-----------+

INSERT INTO person VALUES (1, '张三', 1), (2, '李四', 3), (3, '王五', 6);
SELECT * FROM person;
+------+--------+--------+
| id   | name   | cardId |
+------+--------+--------+
|    1 | 张三   |      1 |
|    2 | 李四   |      3 |
|    3 | 王五   |      6 |
+------+--------+--------+
```

分析两张表发现，`person` 表并没有为 `cardId` 字段设置一个在 `card` 表中对应的 `id` 外键。如果设置了的话，`person` 中 `cardId` 字段值为 `6` 的行就插不进去，因为该 `cardId` 值在 `card` 表中并没有。

#### 内连接

要查询这两张表中有关系的数据，可以使用 `INNER JOIN` ( 内连接 ) 将它们连接在一起。

```
-- INNER JOIN: 表示为内连接，将两张表拼接在一起。
-- on: 表示要执行某个条件。
SELECT * FROM person INNER JOIN card on person.cardId = card.id;
+------+--------+--------+------+-----------+
| id   | name   | cardId | id   | name      |
+------+--------+--------+------+-----------+
|    1 | 张三   |      1 |    1 | 饭卡      |
|    2 | 李四   |      3 |    3 | 农行卡    |
+------+--------+--------+------+-----------+

-- 将 INNER 关键字省略掉，结果也是一样的。
-- SELECT * FROM person JOIN card on person.cardId = card.id;
```

> 注意：`card` 的整张表被连接到了右边。

#### 左外连接

完整显示左边的表 ( `person` ) ，右边的表如果符合条件就显示，不符合则补 `NULL` 。

```
-- LEFT JOIN 也叫做 LEFT OUTER JOIN，用这两种方式的查询结果是一样的。
SELECT * FROM person LEFT JOIN card on person.cardId = card.id;
+------+--------+--------+------+-----------+
| id   | name   | cardId | id   | name      |
+------+--------+--------+------+-----------+
|    1 | 张三   |      1 |    1 | 饭卡      |
|    2 | 李四   |      3 |    3 | 农行卡    |
|    3 | 王五   |      6 | NULL | NULL      |
+------+--------+--------+------+-----------+
```

#### 右外链接

完整显示右边的表 ( `card` ) ，左边的表如果符合条件就显示，不符合则补 `NULL` 。

```
SELECT * FROM person RIGHT JOIN card on person.cardId = card.id;
+------+--------+--------+------+-----------+
| id   | name   | cardId | id   | name      |
+------+--------+--------+------+-----------+
|    1 | 张三   |      1 |    1 | 饭卡      |
|    2 | 李四   |      3 |    3 | 农行卡    |
| NULL | NULL   |   NULL |    2 | 建行卡    |
| NULL | NULL   |   NULL |    4 | 工商卡    |
| NULL | NULL   |   NULL |    5 | 邮政卡    |
+------+--------+--------+------+-----------+
```

#### 全外链接

完整显示两张表的全部数据。

```
-- MySQL 不支持这种语法的全外连接
-- SELECT * FROM person FULL JOIN card on person.cardId = card.id;
-- 出现错误：
-- ERROR 1054 (42S22): Unknown column 'person.cardId' in 'on clause'

-- MySQL全连接语法，使用 UNION 将两张表合并在一起。
SELECT * FROM person LEFT JOIN card on person.cardId = card.id
UNION
SELECT * FROM person RIGHT JOIN card on person.cardId = card.id;
+------+--------+--------+------+-----------+
| id   | name   | cardId | id   | name      |
+------+--------+--------+------+-----------+
|    1 | 张三   |      1 |    1 | 饭卡      |
|    2 | 李四   |      3 |    3 | 农行卡    |
|    3 | 王五   |      6 | NULL | NULL      |
| NULL | NULL   |   NULL |    2 | 建行卡    |
| NULL | NULL   |   NULL |    4 | 工商卡    |
| NULL | NULL   |   NULL |    5 | 邮政卡    |
+------+--------+--------+------+-----------+
```

## 事务

在 MySQL 中，事务其实是一个最小的不可分割的工作单元。事务能够**保证一个业务的完整性**。

比如我们的银行转账：

```
-- a -> -100
UPDATE user set money = money - 100 WHERE name = 'a';

-- b -> +100
UPDATE user set money = money + 100 WHERE name = 'b';
```

在实际项目中，假设只有一条 SQL 语句执行成功，而另外一条执行失败了，就会出现数据前后不一致。

因此，在执行多条有关联 SQL 语句时，**事务**可能会要求这些 SQL 语句要么同时执行成功，要么就都执行失败。

### 如何控制事务 - COMMIT / ROLLBACK

在 MySQL 中，事务的**自动提交**状态默认是开启的。

```
-- 查询事务的自动提交状态
SELECT @@AUTOCOMMIT;
+--------------+
| @@AUTOCOMMIT |
+--------------+
|            1 |
+--------------+
```

**自动提交的作用**：当我们执行一条 SQL 语句的时候，其产生的效果就会立即体现出来，且不能**回滚**。

什么是回滚？举个例子：

```
CREATE DATABASE bank;

USE bank;

CREATE TABLE user (
    id INT PRIMARY KEY,
    name VARCHAR(20),
    money INT
);

INSERT INTO user VALUES (1, 'a', 1000);

SELECT * FROM user;
+----+------+-------+
| id | name | money |
+----+------+-------+
|  1 | a    |  1000 |
+----+------+-------+
```

可以看到，在执行插入语句后数据立刻生效，原因是 MySQL 中的事务自动将它**提交**到了数据库中。那么所谓**回滚**的意思就是，撤销执行过的所有 SQL 语句，使其回滚到**最后一次提交**数据时的状态。

在 MySQL 中使用 `ROLLBACK` 执行回滚：

```
-- 回滚到最后一次提交
ROLLBACK;

SELECT * FROM user;
+----+------+-------+
| id | name | money |
+----+------+-------+
|  1 | a    |  1000 |
+----+------+-------+
```

由于所有执行过的 SQL 语句都已经被提交过了，所以数据并没有发生回滚。那如何让数据可以发生回滚？

```
-- 关闭自动提交
SET AUTOCOMMIT = 0;

-- 查询自动提交状态
SELECT @@AUTOCOMMIT;
+--------------+
| @@AUTOCOMMIT |
+--------------+
|            0 |
+--------------+
```

将自动提交关闭后，测试数据回滚：

```
INSERT INTO user VALUES (2, 'b', 1000);

-- 关闭 AUTOCOMMIT 后，数据的变化是在一张虚拟的临时数据表中展示，
-- 发生变化的数据并没有真正插入到数据表中。
SELECT * FROM user;
+----+------+-------+
| id | name | money |
+----+------+-------+
|  1 | a    |  1000 |
|  2 | b    |  1000 |
+----+------+-------+

-- 数据表中的真实数据其实还是：
+----+------+-------+
| id | name | money |
+----+------+-------+
|  1 | a    |  1000 |
+----+------+-------+

-- 由于数据还没有真正提交，可以使用回滚
ROLLBACK;

-- 再次查询
SELECT * FROM user;
+----+------+-------+
| id | name | money |
+----+------+-------+
|  1 | a    |  1000 |
+----+------+-------+
```

那如何将虚拟的数据真正提交到数据库中？使用 `COMMIT` :

```
INSERT INTO user VALUES (2, 'b', 1000);
-- 手动提交数据（持久性），
-- 将数据真正提交到数据库中，执行后不能再回滚提交过的数据。
COMMIT;

-- 提交后测试回滚
ROLLBACK;

-- 再次查询（回滚无效了）
SELECT * FROM user;
+----+------+-------+
| id | name | money |
+----+------+-------+
|  1 | a    |  1000 |
|  2 | b    |  1000 |
+----+------+-------+
```

> **总结**
>
> 1. **自动提交**
>
>    - 查看自动提交状态：`SELECT @@AUTOCOMMIT` ；
>    - 设置自动提交状态：`SET AUTOCOMMIT = 0` 。
>
> 2. **手动提交**
>
>    `@@AUTOCOMMIT = 0` 时，使用 `COMMIT` 命令提交事务。
>
> 3. **事务回滚**
>
>    `@@AUTOCOMMIT = 0` 时，使用 `ROLLBACK` 命令回滚事务。

**事务的实际应用**，让我们再回到银行转账项目：

```
-- 转账
UPDATE user set money = money - 100 WHERE name = 'a';

-- 到账
UPDATE user set money = money + 100 WHERE name = 'b';

SELECT * FROM user;
+----+------+-------+
| id | name | money |
+----+------+-------+
|  1 | a    |   900 |
|  2 | b    |  1100 |
+----+------+-------+
```

这时假设在转账时发生了意外，就可以使用 `ROLLBACK` 回滚到最后一次提交的状态：

```
-- 假设转账发生了意外，需要回滚。
ROLLBACK;

SELECT * FROM user;
+----+------+-------+
| id | name | money |
+----+------+-------+
|  1 | a    |  1000 |
|  2 | b    |  1000 |
+----+------+-------+
```

这时我们又回到了发生意外之前的状态，也就是说，事务给我们提供了一个可以反悔的机会。假设数据没有发生意外，这时可以手动将数据真正提交到数据表中：`COMMIT` 。

### 手动开启事务 - BEGIN / START TRANSACTION

事务的默认提交被开启 ( `@@AUTOCOMMIT = 1` ) 后，此时就不能使用事务回滚了。但是我们还可以手动开启一个事务处理事件，使其可以发生回滚：

```
-- 使用 BEGIN 或者 START TRANSACTION 手动开启一个事务
-- START TRANSACTION;
BEGIN;
UPDATE user set money = money - 100 WHERE name = 'a';
UPDATE user set money = money + 100 WHERE name = 'b';

-- 由于手动开启的事务没有开启自动提交，
-- 此时发生变化的数据仍然是被保存在一张临时表中。
SELECT * FROM user;
+----+------+-------+
| id | name | money |
+----+------+-------+
|  1 | a    |   900 |
|  2 | b    |  1100 |
+----+------+-------+

-- 测试回滚
ROLLBACK;

SELECT * FROM user;
+----+------+-------+
| id | name | money |
+----+------+-------+
|  1 | a    |  1000 |
|  2 | b    |  1000 |
+----+------+-------+
```

仍然使用 `COMMIT` 提交数据，提交后无法再发生本次事务的回滚。

```
BEGIN;
UPDATE user set money = money - 100 WHERE name = 'a';
UPDATE user set money = money + 100 WHERE name = 'b';

SELECT * FROM user;
+----+------+-------+
| id | name | money |
+----+------+-------+
|  1 | a    |   900 |
|  2 | b    |  1100 |
+----+------+-------+

-- 提交数据
COMMIT;

-- 测试回滚（无效，因为表的数据已经被提交）
ROLLBACK;
```

### 事务的 ACID 特征与使用

**事务的四大特征：**

- **A 原子性**：事务是最小的单位，不可以再分割；
- **C 一致性**：要求同一事务中的 SQL 语句，必须保证同时成功或者失败；
- **I 隔离性**：事务1 和 事务2 之间是具有隔离性的；
- **D 持久性**：事务一旦结束 ( `COMMIT` ) ，就不可以再返回了 ( `ROLLBACK` ) 。

### 事务的隔离性

**事务的隔离性可分为四种 ( 性能从低到高 )** ：

1. **READ UNCOMMITTED ( 读取未提交 )**

   如果有多个事务，那么任意事务都可以看见其他事务的**未提交数据**。

2. **READ COMMITTED ( 读取已提交 )**

   只能读取到其他事务**已经提交的数据**。

3. **REPEATABLE READ ( 可被重复读 )**

   如果有多个连接都开启了事务，那么事务之间不能共享数据记录，否则只能共享已提交的记录。

4. **SERIALIZABLE ( 串行化 )**

   所有的事务都会按照**固定顺序执行**，执行完一个事务后再继续执行下一个事务的**写入操作**。

查看当前数据库的默认隔离级别：

```
-- MySQL 8.x, GLOBAL 表示系统级别，不加表示会话级别。
SELECT @@GLOBAL.TRANSACTION_ISOLATION;
SELECT @@TRANSACTION_ISOLATION;
+--------------------------------+
| @@GLOBAL.TRANSACTION_ISOLATION |
+--------------------------------+
| REPEATABLE-READ                | -- MySQL的默认隔离级别，可以重复读。
+--------------------------------+

-- MySQL 5.x
SELECT @@GLOBAL.TX_ISOLATION;
SELECT @@TX_ISOLATION;
```

修改隔离级别：

```
-- 设置系统隔离级别，LEVEL 后面表示要设置的隔离级别 (READ UNCOMMITTED)。
SET GLOBAL TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;

-- 查询系统隔离级别，发现已经被修改。
SELECT @@GLOBAL.TRANSACTION_ISOLATION;
+--------------------------------+
| @@GLOBAL.TRANSACTION_ISOLATION |
+--------------------------------+
| READ-UNCOMMITTED               |
+--------------------------------+
```

#### 脏读

测试 **READ UNCOMMITTED ( 读取未提交 )** 的隔离性：

```
INSERT INTO user VALUES (3, '小明', 1000);
INSERT INTO user VALUES (4, '淘宝店', 1000);

SELECT * FROM user;
+----+-----------+-------+
| id | name      | money |
+----+-----------+-------+
|  1 | a         |   900 |
|  2 | b         |  1100 |
|  3 | 小明      |  1000 |
|  4 | 淘宝店    |  1000 |
+----+-----------+-------+

-- 开启一个事务操作数据
-- 假设小明在淘宝店买了一双800块钱的鞋子：
START TRANSACTION;
UPDATE user SET money = money - 800 WHERE name = '小明';
UPDATE user SET money = money + 800 WHERE name = '淘宝店';

-- 然后淘宝店在另一方查询结果，发现钱已到账。
SELECT * FROM user;
+----+-----------+-------+
| id | name      | money |
+----+-----------+-------+
|  1 | a         |   900 |
|  2 | b         |  1100 |
|  3 | 小明      |   200 |
|  4 | 淘宝店    |  1800 |
+----+-----------+-------+
```

由于小明的转账是在新开启的事务上进行操作的，而该操作的结果是可以被其他事务（另一方的淘宝店）看见的，因此淘宝店的查询结果是正确的，淘宝店确认到账。但就在这时，如果小明在它所处的事务上又执行了 `ROLLBACK` 命令，会发生什么？

```
-- 小明所处的事务
ROLLBACK;

-- 此时无论对方是谁，如果再去查询结果就会发现：
SELECT * FROM user;
+----+-----------+-------+
| id | name      | money |
+----+-----------+-------+
|  1 | a         |   900 |
|  2 | b         |  1100 |
|  3 | 小明      |  1000 |
|  4 | 淘宝店    |  1000 |
+----+-----------+-------+
```

这就是所谓的**脏读**，一个事务读取到另外一个事务还未提交的数据。这在实际开发中是不允许出现的。

#### 读取已提交

把隔离级别设置为 **READ COMMITTED** ：

```
SET GLOBAL TRANSACTION ISOLATION LEVEL READ COMMITTED;
SELECT @@GLOBAL.TRANSACTION_ISOLATION;
+--------------------------------+
| @@GLOBAL.TRANSACTION_ISOLATION |
+--------------------------------+
| READ-COMMITTED                 |
+--------------------------------+
```

这样，再有新的事务连接进来时，它们就只能查询到已经提交过的事务数据了。但是对于当前事务来说，它们看到的还是未提交的数据，例如：

```
-- 正在操作数据事务（当前事务）
START TRANSACTION;
UPDATE user SET money = money - 800 WHERE name = '小明';
UPDATE user SET money = money + 800 WHERE name = '淘宝店';

-- 虽然隔离级别被设置为了 READ COMMITTED，但在当前事务中，
-- 它看到的仍然是数据表中临时改变数据，而不是真正提交过的数据。
SELECT * FROM user;
+----+-----------+-------+
| id | name      | money |
+----+-----------+-------+
|  1 | a         |   900 |
|  2 | b         |  1100 |
|  3 | 小明      |   200 |
|  4 | 淘宝店    |  1800 |
+----+-----------+-------+


-- 假设此时在远程开启了一个新事务，连接到数据库。
$ mysql -u root -p12345612

-- 此时远程连接查询到的数据只能是已经提交过的
SELECT * FROM user;
+----+-----------+-------+
| id | name      | money |
+----+-----------+-------+
|  1 | a         |   900 |
|  2 | b         |  1100 |
|  3 | 小明      |  1000 |
|  4 | 淘宝店    |  1000 |
+----+-----------+-------+
```

但是这样还有问题，那就是假设一个事务在操作数据时，其他事务干扰了这个事务的数据。例如：

```
-- 小张在查询数据的时候发现：
SELECT * FROM user;
+----+-----------+-------+
| id | name      | money |
+----+-----------+-------+
|  1 | a         |   900 |
|  2 | b         |  1100 |
|  3 | 小明      |   200 |
|  4 | 淘宝店    |  1800 |
+----+-----------+-------+

-- 在小张求表的 money 平均值之前，小王做了一个操作：
START TRANSACTION;
INSERT INTO user VALUES (5, 'c', 100);
COMMIT;

-- 此时表的真实数据是：
SELECT * FROM user;
+----+-----------+-------+
| id | name      | money |
+----+-----------+-------+
|  1 | a         |   900 |
|  2 | b         |  1100 |
|  3 | 小明      |  1000 |
|  4 | 淘宝店    |  1000 |
|  5 | c         |   100 |
+----+-----------+-------+

-- 这时小张再求平均值的时候，就会出现计算不相符合的情况：
SELECT AVG(money) FROM user;
+------------+
| AVG(money) |
+------------+
|  820.0000  |
+------------+
```

虽然 **READ COMMITTED** 让我们只能读取到其他事务已经提交的数据，但还是会出现问题，就是**在读取同一个表的数据时，可能会发生前后不一致的情况。\**这被称为\**不可重复读现象 ( READ COMMITTED )** 。

#### 幻读

将隔离级别设置为 **REPEATABLE READ ( 可被重复读取 )** :

```
SET GLOBAL TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SELECT @@GLOBAL.TRANSACTION_ISOLATION;
+--------------------------------+
| @@GLOBAL.TRANSACTION_ISOLATION |
+--------------------------------+
| REPEATABLE-READ                |
+--------------------------------+
```

测试 **REPEATABLE READ** ，假设在两个不同的连接上分别执行 `START TRANSACTION` :

```
-- 小张 - 成都
START TRANSACTION;
INSERT INTO user VALUES (6, 'd', 1000);

-- 小王 - 北京
START TRANSACTION;

-- 小张 - 成都
COMMIT;
```

当前事务开启后，没提交之前，查询不到，提交后可以被查询到。但是，在提交之前其他事务被开启了，那么在这条事务线上，就不会查询到当前有操作事务的连接。相当于开辟出一条单独的线程。

无论小张是否执行过 `COMMIT` ，在小王这边，都不会查询到小张的事务记录，而是只会查询到自己所处事务的记录：

```
SELECT * FROM user;
+----+-----------+-------+
| id | name      | money |
+----+-----------+-------+
|  1 | a         |   900 |
|  2 | b         |  1100 |
|  3 | 小明      |  1000 |
|  4 | 淘宝店    |  1000 |
|  5 | c         |   100 |
+----+-----------+-------+
```

这是**因为小王在此之前开启了一个新的事务 ( `START TRANSACTION` ) \**，那么\**在他的这条新事务的线上，跟其他事务是没有联系的**，也就是说，此时如果其他事务正在操作数据，它是不知道的。

然而事实是，在真实的数据表中，小张已经插入了一条数据。但是小王此时并不知道，也插入了同一条数据，会发生什么呢？

```
INSERT INTO user VALUES (6, 'd', 1000);
-- ERROR 1062 (23000): Duplicate entry '6' for key 'PRIMARY'
```

报错了，操作被告知已存在主键为 `6` 的字段。这种现象也被称为**幻读，一个事务提交的数据，不能被其他事务读取到**。

#### 串行化

顾名思义，就是所有事务的**写入操作**全都是串行化的。什么意思？把隔离级别修改成 **SERIALIZABLE** :

```
SET GLOBAL TRANSACTION ISOLATION LEVEL SERIALIZABLE;
SELECT @@GLOBAL.TRANSACTION_ISOLATION;
+--------------------------------+
| @@GLOBAL.TRANSACTION_ISOLATION |
+--------------------------------+
| SERIALIZABLE                   |
+--------------------------------+
```

还是拿小张和小王来举例：

```
-- 小张 - 成都
START TRANSACTION;

-- 小王 - 北京
START TRANSACTION;

-- 开启事务之前先查询表，准备操作数据。
SELECT * FROM user;
+----+-----------+-------+
| id | name      | money |
+----+-----------+-------+
|  1 | a         |   900 |
|  2 | b         |  1100 |
|  3 | 小明      |  1000 |
|  4 | 淘宝店    |  1000 |
|  5 | c         |   100 |
|  6 | d         |  1000 |
+----+-----------+-------+

-- 发现没有 7 号王小花，于是插入一条数据：
INSERT INTO user VALUES (7, '王小花', 1000);
```

此时会发生什么呢？由于现在的隔离级别是 **SERIALIZABLE ( 串行化 )** ，串行化的意思就是：假设把所有的事务都放在一个串行的队列中，那么所有的事务都会按照**固定顺序执行**，执行完一个事务后再继续执行下一个事务的**写入操作** ( **这意味着队列中同时只能执行一个事务的写入操作** ) 。

根据这个解释，小王在插入数据时，会出现等待状态，直到小张执行 `COMMIT` 结束它所处的事务，或者出现等待超时。

## 