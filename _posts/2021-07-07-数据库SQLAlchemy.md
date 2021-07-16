---
title: 数据库SQLAlchemy
layout: post
tags:  数据库
categories: ''
---

## 一. SQLAlchemy介绍

SQLAlchemy是用Python编程语言开发的一个开源项目。它提供了SQL工具包和ORM（对象关系映射）工具，使用MIT许可证发行。

SQLAlchemy最初在2006年2月发行，发行后便很快的成为Python社区中最广泛使用的ORM工具之一，丝毫不亚于Django自带的ORM框架。

SQLAlchemy采用简单的Python语言，提供高效和高性能的数据库访问，实现了完整的企业级持久模型。它的理念是，SQL数据库的量级和性能比对象集合重要，而对象集合的抽象又重要于表和行。

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
# 安装SQLalchemy，此时以mysql作为驱动; 其他驱动情况需要安装对应的工具
pip install SQLalchemy
pip3 install pymysql
```

### 2.创建连接

```python
from sqlalchemy import create_engine
engine = create_engine("mysql://user:password@hostname/dbname?charset=uft8")
```

```
dialect+driver://username:password@host:port/database
```

- dialect：数据库，如：sqlite、mysql、oracle等
- driver：数据库驱动，用于连接数据库的，本文使用pymysql
- username：用户名
- password：密码
- host：IP地址
- port：端口
- database：数据库

```s
HOST = 'localhost'
PORT = 3306
USERNAME = 'root'
PASSWORD = '123456'
DB = 'myclass'

# dialect + driver://username:passwor@host:port/database
DB_URI = f'mysql+pymysql://{USERNAME}:{PASSWORD}@{HOST}:{PORT}/{DB}'
```

建议将配置信息放到你的配置文件中，如config.py

这行代码初始化创建了Engine，Engine内部维护了一个Pool（连接池）和Dialect（方言），方言来识别具体连接数据库种类。

![image-20200919134903113](D:\04_Tianchi\Tianchi_task\HonorZheng.github.io\_posts\images\image-20200919134903113.png)

创建好了Engine的同时，Pool和Dialect也已经创建好了，但是此时并没有真正与数据库连接，等到执行具体的语句.connect()等时才会连接到数据库。

```python
engine = create_engine("mysql://user:password@hostname/dbname?charset=uft8",
            echo=True,
            pool_size=8,
            pool_recycle=60*30
            )
#- echo: 当设置为True时会将orm语句转化为sql语句打印，一般debug的时候可用
#- pool_size: 连接池的大小，默认为5个，设置为0时表示连接无限制
#- pool_recycle: 设置时间以限制数据库多久没连接自动断开
```

```python
from sqlalchemy import create_engine
from config import DB_URI


engine = create_engine(DB_URI)  # 创建引擎
conn = engine.connect()  # 连接
result = conn.execute('SELECT 1')  # 执行SQL
print(result.fetchone())  
conn.close()  # 关闭连接
```



### **3. 创建数据库表类（模型）**

前面有提到ORM的重要特点，那么我们操作表的时候就需要通过操作对象来实现，现在我们来创建一个类，以常见的用户表举例：

```python
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import create_engine, Column, Integer, String
from sqlalchemy.orm import sessionmaker
from config import DB_URI

engine = create_engine(DB_URI)
Base = declarative_base(engine)  # SQLORM基类
session = sessionmaker(engine)()  # 构建session对象，注意最后面的小括号


class Student(Base):
    __tablename__ = 'student'  # 表名
    id = Column(Integer, primary_key=True, autoincrement=True)
    name = Column(String(50))
    age = Column(Integer)
    sex = Column(String(10))
    
Base.metadata.create_all()  # 将模型映射到数据库中
```

declarative_base()是sqlalchemy内部封装的一个方法，通过其构造一个基类，这个基类和它的子类，可以将Python类和数据库表关联映射起来。

数据库表模型类通过__tablename__和表关联起来，Column表示数据表的列。

### 4.增删改查（CRUD)操作

#### 1.增

创建表后，接下来我们要添加数据，代码如下：

```python
student = Student(name='Tony', age=18, sex='male')  # 创建一个student对象
session.add(student)  # 添加到session
session.commit()  # 提交到数据库
```

也可以批量添加数据：

```python
session.add_all([
    Student(name='Jane', age=16, sex='female'),
    Student(name='Ben', age=20, sex='male')
])
session.commit()
```

#### 2.删

删除数据使用delete()方法，同样也需要执行session.commit()提交事务

```python
# 删除名称为Ben的数据
session.query(Student).filter(Student.name == 'Ben').delete()
session.commit()

item_list = session.query(Student.name, Student.age).all()
print(item_list)
```

执行结果如下

```python
[('Tony', 22), ('Jane', 16)]
```

#### 3.改

修改数据可以使用update()方法，update完成后记得执行session.commit()

```python
# 修改Tony的age为22
item = session.query(Student.name, Student.age).filter(Student.name == 'Tony').first()
print(item) 
item.age = 22
session.commit()
```

执行结果如下

```python
('Tony', 22)
```

#### 4.查

sqlalchemy提供了query()方法来查询数据

##### **1.获取所有数据**

```python
item_list = session.query(Student).all()
print(item_list)
for item in item_list:
    print(item.name, item.age)
```

执行结果如下

```
[<mymodel.Student object at 0x000002A0E6A38088>, <mymodel.Student object at 0x000002A0E6A38208>, <mymodel.Student object at 0x000002A0E6A38288>]
Tony 18
Jane 16
Ben 20
```

查询得到的item_list是一个包含多个Student对象的列表

查询得到的item_list是一个包含多个Student对象的列表

##### **2.指定查询列**

```python
item_list = session.query(Student.name).all()
print(item_list)

# [('Tony',), ('Jane',), ('Ben',)]
```

##### **3.获取返回查询到数据的第一行**

```python
item = session.query(Student.name).first()
print(item)  

# ('Tony',)
```

##### **4.使用filter()方法进行筛选过滤**

```python
item_list = session.query(Student.name).filter(Student.age >= 18).all()
print(item_list)

# [('Tony',), ('Ben',)]
```

##### **5.使用order_by()进行排序**

```python
item_list = session.query(Student.name, Student.age).order_by(Student.age.desc()).all() # desc()表示倒序
print(item_list)

# [('Ben', 20), ('Tony', 18), ('Jane', 16)]
```

**6.多个查询条件（and和or）**

```python
# 默认为and, 在filter()中用,分隔多个条件表示and
item_list = session.query(Student.name, Student.age, Student.sex).filter(
    Student.age >= 10, Student.sex == 'female'
).all()
print(item_list)  # [('Jane', 16, 'female')]



from sqlalchemy import or_

# 使用or_连接多个条件
item_list = session.query(Student.name, Student.age, Student.sex).filter(
    or_(Student.age >= 20, Student.sex == 'female')
).all()
print(item_list)  # [('Jane', 16, 'female'), ('Ben', 20, 'male')]
```

##### **6.equal/like/in**

```python
# 等于
item_list = session.query(Student.name, Student.age, Student.sex).filter(
    Student.age == 18
).all()
print(item_list)  # [('Tony', 18, 'male')]

# 不等于
item_list = session.query(Student.name, Student.age, Student.sex).filter(
    Student.age != 18
).all()
print(item_list)  # [('Jane', 16, 'female'), ('Ben', 20, 'male')]

# like
item_list = session.query(Student.name, Student.age, Student.sex).filter(
    Student.name.like('%To%')
).all()
print(item_list)  # [('Tony', 18, 'male')]

# in
item_list = session.query(Student.name, Student.age, Student.sex).filter(
    Student.age.in_([16, 20])
).all()
print(item_list) # [('Jane', 16, 'female'), ('Ben', 20, 'male')]
```

##### **7.count计算个数**

```python
count = session.query(Student).count()
print(count)  # 3
```



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

## 
