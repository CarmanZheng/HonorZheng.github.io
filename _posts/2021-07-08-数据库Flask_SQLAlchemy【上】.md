---
title: 数据库Flask-SQLAlchemy【上】
layout: post
tags: 数据库
categories: ''
---

## 一、Flask-SQLAlchemy介绍

需要安装pymysql ，sqlalchemy，flask-sqlalchemy三个库

## 二、Flask-SQLAlchemy常规操作

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





| 选项名      | 说明                                        |
| ----------- | ------------------------------------------- |
| primary_key | 设置为True,为主键                           |
| unique      | 设置为True，这列不允许有重复值              |
| index       | 设置为True，为这列创建索引，提升查询效率    |
| nullable    | 设置为True，这列允许使用空值; False，不允许 |
| default     | 为这列设置默认值                            |





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

#### 2.通过sqlalchemy语句-- 增删改查

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

## 三、查询

#### 1.常用的SQLAlchemy查询过滤器

| 过滤器      | 说明                                             |
| ----------- | ------------------------------------------------ |
| filter()    | 把过滤器添加到原查询上，返回一个新查询           |
| filter_by() | 把等值过滤器添加到原查询上，返回一个新查询       |
| limit       | 使用指定的值限定原查询返回的结果                 |
| offset()    | 偏移原查询返回的结果，返回一个新查询             |
| order_by()  | 根据指定条件对原查询结果进行排序，返回一个新查询 |
| group_by()  | 根据指定条件对原查询结果进行分组，返回一个新查询 |

#### 2.常用的SQLAlchemy查询执行器

| 方法           | 说明                                         |
| -------------- | -------------------------------------------- |
| all()          | 以列表形式返回查询的所有结果                 |
| first()        | 返回查询的第一个结果，如果未查到，返回None   |
| first_or_404() | 返回查询的第一个结果，如果未查到，返回404    |
| get()          | 返回指定主键对应的行，如不存在，返回None     |
| get_or_404()   | 返回指定主键对应的行，如不存在，返回404      |
| count()        | 返回查询结果的数量                           |
| paginate()     | 返回一个Paginate对象，它包含指定范围内的结果 |

查询:filter_by精确查询

返回名字等于wang的所有人

```python
User.query.filter_by(name='wang').all()
```

first()返回查询到的第一个对象

```python
User.query.first()
```

all()返回查询到的所有对象

```python
User.query.all()
```

filter模糊查询，返回名字结尾字符为g的所有数据。

```python
User.query.filter(User.name.endswith('g')).all()
```

get():参数为主键，如果主键不存在没有返回内容

```python
User.query.get()
```

逻辑非，返回名字不等于wang的所有数据

```python
User.query.filter(User.name!='wang').all()
```

not_ 相当于取反

```python
from sqlalchemy import not_
User.query.filter(not_(User.name=='chen')).all()
```

逻辑与，需要导入and_，返回and()条件满足的所有数据

```python
from sqlalchemy import and_
User.query.filter(and_(User.name!='wang',User.email.endswith('163.com'))).all()
```

逻辑或，需要导入or_

```python
from sqlalchemy import or_
User.query.filter(or_(User.name!='wang',User.email.endswith('163.com'))).all()
```

查询数据后删除

```python
user = User.query.first()
db.session.delete(user)
db.session.commit()
User.query.all()
```

更新数据

```python
user = User.query.first()
user.name = 'dong'
db.session.commit()
User.query.first()
```

关联查询示例：

> 角色和用户的关系是一对多的关系，一个角色可以有多个用户，一个用户只能属于一个角色。

- 查询角色的所有用户

```python
#查询roles表id为1的角色
ro1 = Role.query.get(1)
#查询该角色的所有用户
ro1.us.all()
```

- 查询用户所属角色

```python
#查询users表id为3的用户
us1 = User.query.get(3)
#查询用户属于什么角色
us1.role
```

## 四、分页

目前使用vue进行前端分页，后面需要用到的时候进行数据库分页的补充。

Flask-SQLAlchemy提供的paginate()方法，返回值是一个Pagination类对象。

这个类包含很多的属性，可以用来在模板中生成分页的链接，因此可以将其作为参数传入模板。

#### 1、paginate对象

```python
paginate = Article.query.paginate(page=None, per_page=None, error_out=True, max_per_page=None)

# page   		当前页码，默认为1，表示第1页开始；  
# per_page       每一页有几个项，即一页显示多少项，默认20，表示默认显示20项
# error_out		是否抛出错误（默认为True），有错误时抛出404
# max_per_page  每页最大显示项数
# prev_num	 	上一页页码
# next_num		下一页页码
# pages			总页码数
# items			当前页码数据对象
```


这边说明一下这个方法对应的参数：

```html
{% macro page(data, url) -%}
    {% if data %}
        <ul class="pagination pagination-sm no-margin pull-right">
            <li><a href="{{ url_for(url,page=1) }}">首页</a></li>
            {% if data.has_prev %}
                <li><a href="{{ url_for(url,page=data.prev_num) }}">上一页</a></li>
            {% else %}
                <li class="disabled"><a href="#">上一页</a></li>
            {% endif %}
 
            {% for pagenum in data.iter_pages() %}
                {% if pagenum == data.page %}
                    <li class="active"><a href="#">{{ pagenum }}</a></li>
                {% else %}
                    <li><a href="{{ url_for(url,page=pagenum) }}">{{ pagenum }}</a></li>
                {% endif %}
            {% endfor %}
 
            {% if data.has_next %}
                <li><a href="{{ url_for(url,page=data.next_num) }}">下一页</a></li>
            {% else %}
                <li class="disabled"><a href="#">下一页</a></li>
            {% endif %}
 
            <li><a href="{{ url_for(url,page=data.pages) }}">尾页</a></li>
        </ul>
    {% endif %}
{%- endmacro %}
```





