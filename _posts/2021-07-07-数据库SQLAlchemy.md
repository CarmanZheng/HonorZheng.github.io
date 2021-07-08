---
title: VPN
layout: post
tags: 网络通信
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

![image-20200919134903113](D:\04_Tianchi\Tianchi_task\HonorZheng.github.io\_posts\images\image-20200919134903113.png)

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

## 三、SQLAlchemy常规操作

### 1.基础配置

Flask-SQLAlchemy存在以下的配置值，Flask-SQLAlchemy从主Flask配置中加载这些值，可以通过各种方式进行填充。

| 配置名称                         | 介绍                                                         |
| -------------------------------- | ------------------------------------------------------------ |
| `SQLALCHEMY_DATABASE_URI`        | 用于连接数据的数据库。例如：`sqlite:tmp/test.db``mysql://username:password@server/db` `mysql+pymysql://root:123456@localhost:3306/TTC` |
| `SQLALCHEMY_BINDS`               | 一个映射绑定 (bind) 键到 SQLAlchemy 连接 URIs 的字典。 更多的信息请参阅 [*绑定多个数据库*](http://www.pythondoc.com/flask-sqlalchemy/binds.html#binds)。 |
| `SQLALCHEMY_ECHO`                | 如果设置成 True，SQLAlchemy 将会记录所有发到标准输出(stderr)的语句，这对调试很有帮助。 |
| `SQLALCHEMY_RECORD_QUERIES`      | 可以用于显示地禁用或者启用查询记录。查询记录在调试或者测试模式下自动启用。更多信息请参阅 `get_debug_queries()`。 |
| `SQLALCHEMY_NATIVE_UNICODE`      | 可以用于显示地禁用支持原生的 unicode。这是 某些数据库适配器必须的（像在 Ubuntu 某些版本上的 PostgreSQL），当使用不合适的指定无编码的数据库 默认值时。 |
| `SQLALCHEMY_POOL_SIZE`           | 数据库连接池的大小。默认是数据库引擎的默认值 （通常是 5）。  |
| `SQLALCHEMY_POOL_TIMEOUT`        | 指定数据库连接池的超时时间。默认是 10。                      |
| `SQLALCHEMY_POOL_RECYCLE`        | 自动回收连接的秒数。这对 MySQL 是必须的，默认 情况下 MySQL 会自动移除闲置 8 小时或者以上的连接。 需要注意地是如果使用 MySQL 的话， Flask-SQLAlchemy 会自动地设置这个值为 2 小时。 |
| `SQLALCHEMY_MAX_OVERFLOW`        | 控制在连接池达到最大值后可以创建的连接数。当这些额外的 连接回收到连接池后将会被断开和抛弃。 |
| `SQLALCHEMY_TRACK_MODIFICATIONS` | 如果设置成 True (默认情况)，Flask-SQLAlchemy 将会追踪对象的修改并且发送信号。这需要额外的内存， 如果不必要的可以禁用它。 |

### 2.连接 URI 格式

SQLAlchemy 把一个引擎的源表示为一个连同设定引擎选项的可选字符串参数的 URI。URI 的形式是:

```mysql
dialect+driver://username:password@host:port/database
```

该字符串中的许多部分是可选的。如果没有指定驱动器，会选择默认的（确保在这种情况下 *不* 包含 `+` ）。

Postgres:

```
postgresql://scott:tiger@localhost/mydatabase
```

MySQL:

```
mysql://scott:tiger@localhost/mydatabase
mysql+pymysql://root:123456@localhost:3306/flask_ttc
```

Oracle:

```
oracle://scott:tiger@127.0.0.1:1521/sidname
```

SQLite (注意开头的四个斜线):

```
sqlite:absolute/path/to/foo.db
```

### 3.数据库数据类型

| 类型         | 介绍                                     |
| ------------ | ---------------------------------------- |
| Integer      | 一个整数                                 |
| String(size) | 一个字符串，并可以设置它的最大长度       |
| Text         | 更长的unicode文本                        |
| DateTime     | 通过Python的datetime对象来表示时间日期   |
| Float        | 存储一个浮点数                           |
| Boolean      | 存储一个布尔值                           |
| PickleType   | 存储一个序列化（ Pickle ）后的Python对象 |
| LargeBinary  | 存储巨大长度的二进制数据                 |

app.py

```python
from flask import Flask
from flask_script import Manager
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)  # 创建一个Flask app对象
# 数据库链接的配置，此项必须，格式为（数据库+驱动://用户名:密码@数据库主机地址:端口/数据库名称）
app.config['SQLALCHEMY_DATABASE_URI'] = 'mysql+pymysql://root:123456@localhost:3306/flask_ttc'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False  # 跟踪对象的修改，在本例中用不到调高运行效率，所以设置为False
db = SQLAlchemy(app=app)  # 为哪个Flask app对象创建SQLAlchemy对象，赋值为db
manager = Manager(app=app)  # 初始化manager模块

@app.route('/')
def hello_world():
    reutrn 'Hello World!'

if __name__ == '__main__':
    manager.run()  # 运行服务器
```

model.py   创建数据模型

```python
from app import db  # 导入app文件中的SQLAlchemy对象

class Student(db.Model):  # 继承SQLAlchemy.Model对象，一个对象代表了一张表
    s_id= db.Column(db.Integer, primary_key=True, autoincrement=True, unique=True)  # id 整型，主键，自增，唯一
    s_name = db.Column(db.String(20))  # 名字 字符串长度为20
    s_age = db.Column(db.Integer, default=20)  # 年龄 整型，默认为20

    __tablename__ = 'student'  # 该参数可选，不设置会默认的设置表名，如果设置会覆盖默认的表名
    def __init__(self, name, age):  # 初始化方法，可以对对象进行创建
        self.s_name = name
        self.s_age = age
    def __repr__(self):  # 输出方法，与__str__类似，但是能够重现它所代表的对象
        return '<Student %r, %r, %r>' % (self.s_id, self.s_name, self.sage)
```

创建数据表，这里是通过python shell创建的，实际中需要写入`模块.py`文件中；

```python
>>> from app import db  # 引入SQLAlchemy
>>> from models import *  # 映入加载了model的SQLAlchemy对象，db
>>> db.create_all()  # 创建表
# 至此生成了student表
```

### 4.表操作

#### 1.通过原生MySQL语句

此种方式在sqlalchemy走投无路之时，相当好用；平时还是使用sqlalchemy语句；

```python
sql = 'select * from student;'
stus = db.session.execute(sql)
```

#### 2.通过sqlalchemy语句

```python
# 增 - Create
stu = Student('TTC', 22)     # 1.创建数据对象
db.session.add(stu)			# 2.会话加入数据
db.session.commit()			# 3.提交数据	
# 如果添加的对象是列表，使用db.session.add_all(list)
stus = []
db.session.add_all(stus)
db.session.commit()
```

```python
# 删 - Delete
stu = Student.query.get(1)  # 1.找出需要删除的对象
db.session.delete(stu)  	# 2.删除
db.session.commit()  		# 3.提交事务
```

```python
# 改 - Update
stu = Student.query.get(1)   # 1.找到需要修改的对象
stu.s_age = 18  			# 2.修改值
db.session.commit() 		 # 3.提交
```

```python
# 查
# 过滤查询
stu = Student.query.filter(s_name='TTC').first()
# 如果查询一个不存在的会返回一个None

# 主键查询
stu = Student.query.get(1)  

# 比较查询
# __lt__ 小于  __le__小于等于   __gt__ 大于  __ge__ 大于等于
Student.query.filter(Student.s_age.__lt__(16))  # 小于16岁 
Student.query.filter(Student.s_age.__le__(16))  # 小于等于16岁
Student.query.filter(Student.s_age.__gt__(16))  # 大于16岁
Student.query.filter(Student.s_age.__ge__(16))  # 大于等于16岁

# in查询【查询匹配】
Student.query.filter(Student.s_age.in_([16, 17, 18, 19, 20])) # 使用in_方法获取与列表中值相匹配的值

# 查询并排序
Student.query.order_by('s_age') # 按年龄排序，默认升序，在前面加-号为降序'-s_age'

# 按规则输出【起始位置，条数】
Student.query.filter(s_age=18).offset(2).limit(3)  # 跳过二条开始查询，限制输出3条

# 多条件查询
Student.query.filter(Student.s_age == 18, Student.s_name == 'TTC')
# 也可以通过and_并且  or_或者  not_非 的方法来进行查询
Student.query.filter(and_(Student.s_age == 18, Student.s_name == 'TTC'))
Student.query.filter(or_(Student.s_age == 18, Student.s_name == 'TTC'))
Student.query.filter(not_(Student.s_age == 18, Student.s_name == 'TTC'))
```

### 5.模型关系

#### 1.一对多(one-to-many)关系

​	最常见的关系就是一对多的关系。因为关系在它们建立之前就已经声明，可以使用字符串来指代还没有创建的类。

比如班级和学生的关系即为一对多的关系，一个班级对应多名学生。

models.py

```python
from app import db

class Grade(db.Model):
    """班级表"""
    g_id = db.Column(db.Integer, primary_key=True, autoincrement=True)  # 班级id
    g_name = db.Column(db.String(20))  # 班级名称
    g_desc = db.Column(db.String(100))  # 描述
    students = db.relationship('Student', backref='grade', lazy=True)  # 关系

    __tablename__ = 'grade'  # 表名

    def __init__(self, name, desc):  # 初始化方法
        self.g_name = name  # 名称
        self.g_desc = desc  # 描述

    def __repr__(self):  # 输出方法，显示对象内容
        return '<Grade %r, %r, %r>' % (self.g_id, self.g_name, self.g_desc)

class Student(db.Model):
    """学生表"""
    s_id = db.Column(db.Integer, primary_key=True, autoincrement=True)  # 学生ID
    s_name = db.Column(db.String(20))  # 学生姓名
    s_age = db.Column(db.Integer, default=20)  # 学生年龄
    g_id = db.Column(db.Integer, db.ForeignKey('grade.g_id'))  # 外键

    __tablename__ = 'student'  # 表名

    def __init__(self, name, age, g_id):  # 初始化方法
        self.s_name = name  # 姓名
        self.s_age = age  # 年龄
        self.g_id = g_id  # 班级ID

    def __repr__(self):  # 输出方法，显示对象内容
        return '<Student %r, %r, %r>' % (self.s_id, self.s_name, self.s_age)
```

通过学生获取班级

```csharp
g_id = Student.query.get(12).grade.g_id
```

通过班级获取学生

```csharp
stus = Grade.query.get(2).students
```

#### 2.一对一（one-to-one）关系

需要使用一对一的关系，和一对多的关系的写法是一样的，只是在relationship中有一个参数不同，需要将`uselist=Flase`，这样两个表之间的关系就变成了一对一的关系。其参数的含义就是不使用列表，两个表之间只有一条对应一条的关联。

#### 3.多对多（many-to-many）关系

如果想使用多对多关系，需要定义一一个用于关系的辅助表，对于这个辅助表，建议不使用模型，而是采用一个实际的表：

我们接着之前的写，加入学生(Student)和课程(Course)的关系

```python
from app import db  # 引入模块
class Student(db.Model):
    """学生表"""
    s_id = db.Column(db.Integer, primary_key=True, autoincrement=True)  # 学生ID
    s_name = db.Column(db.String(20))  # 学生姓名
    s_age = db.Column(db.Integer, default=20)  # 学生年龄
    g_id = db.Column(db.Integer, db.ForeignKey('grade.g_id'))  # 班级ID

    __tablename__ = 'student'  # 学生表

    def __init__(self, name, age, g_id):  # 初始化方法
        self.s_name = name  # 名称
        self.s_age = age  # 年龄
        self.g_id = g_id  # 班级ID

    def __repr__(self):  # 格式化输出显示对象中的值
        return '<Student %r, %r, %r>' % (self.s_id, self.s_name, self.s_age)

# 辅助表 记录学生表和课程表的多对多关系
sc = db.Table('sc',  # 表名
              db.Column('s_id', db.Integer, db.ForeignKey('student.s_id'), primary_key=True),
              db.Column('c_id', db.Integer, db.ForeignKey('course.c_id'), primary_key=True)
              )

class Course(db.Model):
    """课程表"""
    c_id = db.Column(db.Integer, primary_key=True, autoincrement=True)  # 课程的ID
    c_name = db.Column(db.String(20))  # 课程名称
    students = db.relationship('Student', secondary=sc, backref='courses')  # 与学生表的关系
    # secondary指定副表名称
    def __init__(self, name):  # 初始化方法
        self.c_name = name  # 课程名

    def __repr__(self):  # 格式化输出显示对象中的值
        return '<Course %r, %r >' % (self.c_id, self.c_name)
```

##### 1.多对多关系的数据插入方式

```csharp
stu = Student.query.get(s_id)  # 获取学生对象
cou = Course.query.get(c_id)  # 获取课程对象
# 方式一
cou.students.append(stu)  # 建立学生和课程的关系
db.session.add(cou)  # 为会话添加事务
# 方式二
stu.courses.append(cou)
db.session.add(stu)
# 方式一和方式二等效

db.session.commit()  # 提交事务
```

##### 2.多对多关系的数据删除方式

```csharp
stu = Student.query.get(s_id)
cou = Course.query.get(c_id)
# 方式一
cou.students.remove(stu)

# 方式二
stu.students.remove(cou)

# 方式一和方式二等效

db.session.commit()
```

##### 3.多对多关系互相查询

通过学生查课程

```csharp
cous = Student.query.get(s_id).courses
```

通过课程查学生

```csharp
stus = Course.query.get(c_id).students
```


引用链接：https://www.jianshu.com/p/8c038f0134f8
