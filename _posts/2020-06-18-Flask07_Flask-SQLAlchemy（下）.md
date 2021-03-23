---
title: Flask_Flask-SQLAlchemy配置与基本使用
layout: post
tags: Flask
categories: ''

---

# Flask_Flask-SQLAlchemy 配置与基本使用

参考文献：https://blog.csdn.net/u011146423/article/details/87605812

## 一、Flask-SQLAlchemy 配置与基本使用

### 1.1、配置键

Flask-SQLAlchemy 扩展能够识别的配置键的清单：[SQLAlchemy 操作数据库的ORM组件](https://blog.csdn.net/u011146423/article/details/84643151)

| `SQLALCHEMY_DATABASE_URI`        | 用于连接数据的数据库。例如：`sqlite:tmp/test.db``mysql://username:password@server/db` |
| -------------------------------- | ------------------------------------------------------------ |
| `SQLALCHEMY_BINDS`               | 一个映射绑定 (bind) 键到 SQLAlchemy 连接 URIs 的字典。 更多的信息请参阅 [*绑定多个数据库*](http://www.pythondoc.com/flask-sqlalchemy/binds.html#binds)。 |
| `SQLALCHEMY_ECHO`                | 如果设置成 True，SQLAlchemy 将会记录所有发到标准输出(stderr)的语句，这对调试很有帮助。 |
| `SQLALCHEMY_RECORD_QUERIES`      | 可以用于显示地禁用或者启用查询记录。查询记录在调试或者测试模式下自动启用。更多信息请参阅 `get_debug_queries()`。 |
| `SQLALCHEMY_NATIVE_UNICODE`      | 可以用于显示地禁用支持原生的 unicode。这是 某些数据库适配器必须的（像在 Ubuntu 某些版本上的 PostgreSQL），当使用不合适的指定无编码的数据库 默认值时。 |
| `SQLALCHEMY_POOL_SIZE`           | 数据库连接池的大小。默认是数据库引擎的默认值 （通常是 5）。  |
| `SQLALCHEMY_POOL_TIMEOUT`        | 指定数据库连接池的超时时间。默认是 10。                      |
| `SQLALCHEMY_POOL_RECYCLE`        | 自动回收连接的秒数。这对 MySQL 是必须的，默认 情况下 MySQL 会自动移除闲置 8 小时或者以上的连接。 需要注意地是如果使用 MySQL 的话， Flask-SQLAlchemy 会自动地设置这个值为 2 小时。 |
| `SQLALCHEMY_MAX_OVERFLOW`        | 控制在连接池达到最大值后可以创建的连接数。当这些额外的 连接回收到连接池后将会被断开和抛弃。 |
| `SQLALCHEMY_TRACK_MODIFICATIONS` | 如果设置成 True (默认情况)，Flask-SQLAlchemy 将会追踪对象的修改并且发送信号。这需要额外的内存， 如果不必要的可以禁用它。 |

### 1.2、基本使用

```python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
 
app=Flask(__name__)
 
# 连接数据库
app.config['SQLALCHEMY_DATABASE_URI'] = '数据库类型://数据库用户名:数据库密码@数据库地址:数据库端口/数据库名字'
# 设置是否跟踪数据库的修改情况，一般不跟踪
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
# 数据库操作时是否显示原始SQL语句，一般都是打开的，因为我们后台要日志
app.config['SQLALCHEMY_ECHO'] = True
 
# 实例化orm框架的操作对象，后续数据库操作，都要基于操作对象来完成
db = SQLAlchemy(app)
 
# 声明模型类
class User(db.Model):
    # 给表重新定义一个名称，默认名称是类名的小写，比如该类默认的表名是user。
    __tablename__ = "users"
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(16), unique=True)
    email = db.Column(db.String(32), unique=True)
    password = db.Column(db.String(16))
 
@app.route("/")
def index():
    return "hello"
 
if __name__ == '__main__':
    db.create_all()    # 创建当前应用中声明的所有模型类对应的数据表，db.drop_all()是删除表
    app.run(debug=True)
```

## 二、连接 URI 格式

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
```

Oracle:

```
oracle://scott:tiger@127.0.0.1:1521/sidname
```

SQLite (注意开头的四个斜线):

```
sqlite:absolute/path/to/foo.db
```

## 三、SQLAlchemy的数据类型

下图显示了最常用的SQLAlchemy列的类型： 
![这里写图片描述](https://img-blog.csdn.net/20161107010358428)

下图显示了最常用的SQLAlchemy列选项： 
![这里写图片描述](https://img-blog.csdn.net/20161107010602160)

> Flask-SQLAlchemy 要求每个模型都要定义主键，这一列经常命名为 id。

下面看SQLAlchemy提供的过滤器和查询函数： 
![这里写图片描述](https://img-blog.csdn.net/20161107022638126)

![这里写图片描述](https://img-blog.csdn.net/20161107023259291)

创建数据库报错处理：

Syntax error or access violation: 1071 Specified key was too long; max key length is 767 bytes

为什么出现这个问题：

（1）在mysql 5.5.3之前，mysql的InnoDB引擎，要求设置的主键长度不得超过767bytes。mysql的MyIsam引擎的主键长度不得超过1000 bytes。

（2）在mysql中，gbk字符集会占用2个字节。utf8字符会占用3个字节。而且从mysql5.5.3之后的版本，mysql 开始支持utf8m4字符，代表着一个字符占用4个字节。

也就是说：

```python
（255+10+10）*3 = 825  //在用utf8作为字符集的时候，超过了规定的767 bytes



（255+10+10）*2 = 550  //当该用gbk作为字符集的时候



（255+10+10）*4 = 1100  //当用utf8m4作为字符集的时候，也超标了
```

解决办法：

  修改字段的大小，最大设置为255；比如 varchar(255)，String(255)

  my.cnf、（windows是my.ini） 配置：

  default-storage-engine=INNODB

  innodb_large_prefix=on # 启用innodb_large_prefix选项，将约束项扩展至3072byte

## 四、没有没有

## 五、具体操作实例

### 5.1 、实例代码

```python
# -*- coding:utf-8 -*-
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
 
app = Flask(__name__)
# url的格式为：数据库的协议：//用户名：密码@ip地址：端口号（默认可以不写）/数据库名
app.config["SQLALCHEMY_DATABASE_URI"] = "mysql://root:mysql@localhost/first_flask"
# 动态追踪数据库的修改. 性能不好. 且未来版本中会移除. 目前只是为了解决控制台的提示才写的
app.config["SQLALCHEMY_TRACK_MODIFICATIONS"] = False
# 创建数据库的操作对象
db = SQLAlchemy(app)
 
 
class Role(db.Model):
 
    __tablename__ = "roles"
    id = db.Column(db.Integer,primary_key=True)
    name = db.Column(db.String(16),unique=True)
    # 给Role类创建一个uses属性，关联users表。
    # backref是反向的给User类创建一个role属性，关联roles表。这是flask特殊的属性。
    users = db.relationship('User',backref="role")
    # 相当于__str__方法。
    def __repr__(self):
        return "Role: %s %s" % (self.id,self.name)
 
 
class User(db.Model):
    # 给表重新定义一个名称，默认名称是类名的小写，比如该类默认的表名是user。
    __tablename__ = "users"
    id = db.Column(db.Integer,primary_key=True)
    name = db.Column(db.String(16),unique=True)
    email = db.Column(db.String(32),unique=True)
    password = db.Column(db.String(16))
    # 创建一个外键，和django不一样。flask需要指定具体的字段创建外键，不能根据类名创建外键
    role_id = db.Column(db.Integer,db.ForeignKey("roles.id"))
 
    def __repr__(self):
        return "User: %s %s %s %s" % (self.id,self.name,self.password,self.role_id)
 
 
@app.route('/')
def hello_world():
    return 'Hello World!'
 
if __name__ == '__main__':
    # 删除所有的表
    db.drop_all()
    # 创建表
    db.create_all()
 
    ro1 = Role(name = "admin")
    # 先将ro1对象添加到会话中，可以回滚。
    db.session.add(ro1)
    
    '''
        在执行 db.session.commit() 提交到数据库出错时，需要执行数据库回滚，保证后续操作正常。
        try: 
            .
            .
        except Exception as e:
            db.session.rollback() # 执行数据库回滚
    '''
 
    try:
        ro2 = Role()
        ro2.name = 'user'
        db.session.add(ro2)
        # 最后插入完数据一定要提交
        db.session.commit()
    except Exception as e:
        db.session.rollback() 
        raise e
 
    try:
        us1 = User(name='wang', email='wang@163.com', password='123456', role_id=ro1.id)
        us2 = User(name='zhang', email='zhang@189.com', password='201512', role_id=ro2.id)
        us3 = User(name='chen', email='chen@126.com', password='987654', role_id=ro2.id)
        us4 = User(name='zhou', email='zhou@163.com', password='456789', role_id=ro1.id)
        us5 = User(name='tang', email='tang@itheima.com', password='158104', role_id=ro2.id)
        us6 = User(name='wu', email='wu@gmail.com', password='5623514', role_id=ro2.id)
        us7 = User(name='qian', email='qian@gmail.com', password='1543567', role_id=ro1.id)
        us8 = User(name='liu', email='liu@itheima.com', password='867322', role_id=ro1.id)
        us9 = User(name='li', email='li@163.com', password='4526342', role_id=ro2.id)
        us10 = User(name='sun', email='sun@163.com', password='235523', role_id=ro2.id)
        db.session.add_all([us1, us2, us3, us4, us5, us6, us7, us8, us9, us10])
        db.session.commit()
    except Exception as e:
        db.session.rollback()
        raise e
 
    app.run(debug=True)

```

下面插播一条bug：当把表格创建完成，注释这两句话：

```python
# 删除所有的表
db.drop_all()
# 创建表
db.create_all()
```

然后向表格里面插入数据，此时会出现这样的错误：

```
sqlalchemy.exc.IntegrityError: (_mysql_exceptions.IntegrityError) (1062, "Duplicate entry 'admin' for key 'name'") [SQL: u'INSERT INTO roles (name) VALUES (%s)'] [parameters: ('admin',)]
```

网上好多资料说把字段约束unique=True去掉，但是根本原因不在这。原因就是因为app.run(debug=True)，开启debug模式之后，当修改代码的时候，比如将删除表和创建表这两句话注释，然后打开插入数据的注释，这个过程debug模式默认就已经把程序运行一遍了；此时数据库就已经有了数据，当再次手动执行的时候，又往数据库中插入了一条数据，这时候就会报错。因为字段的约束是唯一性的unique，所以解决的办法有两种：

  第一种：就是不要将删除表和创建表这两句话注释，每次执行都要带着这两个句话，无论是debug模式自动执行还是我们手动执行程序，都会先删除表然后再创建表。

  第二种：关闭debug模式就是这样app.run()，但是这样每次修改代码都需要重启服务。

### 5.2、数据库的查询

**1.以下的方法都是返回一个新的查询，需要配合执行器使用。**

filter(): 过滤，功能比较强大，多表关联查询。
filter_by()：过滤，一般用在单表查询的过滤场景。

order_by()：排序。默认是升序，降序需要导包：from sqlalchemy import * 。然后引入desc方法。比如order_by(desc("email")).按照邮箱字母的降序排序。

group_by()：分组。

**2.以下都是一些常用的执行器：配合上面的过滤器使用。**

get()：获得id等于几的函数。

  比如：查询id=1的对象。

  get(1)，切记：括号里没有“id=”，直接传入id的数值就ok。因为该函数的功能就是查询主键等于几的对象。

all()：查询所有的数据。

first()：查询第一个数据。

count()：返回查询结果的数量。

paginate()：分页查询，返回一个分页对象。

  paginate(参数1，参数2，参数3)=>参数1：当前是第几页；参数2：每页显示几条记录；参数3：是否要返回错误。

  返回的分页对象有三个属性：items：获得查询的结果，pages：获得一共有多少页，page：获得当前页。

### 5.3、常用的逻辑符

需要导入包才能用的有：from sqlalchemy import * 

not_、and_、or_ 还有上面说的排序desc。

常用的内置的有：in_：表示某个字段在什么范围之中。

### 5.4、其他关系的一些数据库查询

endswith()：以什么结尾。

startswith()：以什么开头。

contains()：包含

### **5.5、实际的用法**

```mysql

1. 查询所有用户数据
User.query.all()
 
2. 查询有多少个用户
User.query.count()
 
3. 查询第1个用户
User.query.first()
 
4. 查询id为4的用户[3种方式]
User.query.get(4)
User.query.filter_by(id=4).first()　　　　
User.query.filter(User.id==4).first()
 
filter:(类名.属性名==)
filter_by:(属性名=)
 
filter_by: 用于查询简单的列名，不支持比较运算符
filter比filter_by的功能更强大，支持比较运算符，支持or_、in_等语法。
 
5. 查询名字结尾字符为g的所有数据[开始/包含]
User.query.filter(User.name.endswith('g')).all()
User.query.filter(User.name.contains('g')).all()
 
6. 查询名字不等于wang的所有数据[2种方式]
from sqlalchemy import not_
PS：逻辑查询的格式：逻辑符_(类属性其他的一些判断)
User.query.filter(not_(User.name=='wang')).all()
User.query.filter(User.name!='wang').all()
 
7. 查询名字和邮箱都以 li 开头的所有数据[2种方式]
from sqlalchemy import and_
User.query.filter(and_(User.name.startswith('li'), User.email.startswith('li'))).all()
User.query.filter(User.name.startswith('li'), User.email.startswith('li')).all()
 
8. 查询password是 `123456` 或者 `email` 以 `itheima.com` 结尾的所有数据
from sqlalchemy import or_
User.query.filter(or_(User.password=='123456', User.email.endswith('itheima.com'))).all()
 
9. 查询id为 [1, 3, 5, 7, 9] 的用户列表
User.query.filter(User.id.in_([1, 3, 5, 7, 9])).all()
 
10. 查询name为liu的角色数据：关系引用
User.query.filter_by(name='liu').first().role.name
 
11. 查询所有用户数据，并以邮箱排序
User.query.order_by('email').all()  默认升序
User.query.order_by(desc('email')).all() 降序
 
12. 查询第2页的数据, 每页只显示3条数据
help(User.query.paginate)
pages = User.query.paginate(2, 3, False)
PS：三个参数: 1. 当前要查询的页数 2. 每页的数量 3. 是否要返回错误
pages.items # 获取查询的结果
pages.pages # 总页数
pages.page #
```

 3.使用第三方扩展框架迁移数据库文件。

使用框架需要配置的代码如下：

```python
# -*- coding:utf-8 -*-
from flask import Flask
from flask_sqlalchemy import SQLAlchemy  # 操作数据库的扩展包
from flask_script import Manager  # 用命令操作的扩展包
from flask_migrate import Migrate,MigrateCommand  # 操作数据库迁移文件的扩展包
 
app = Flask(__name__)
app.debug = True
app.config["SQLALCHEMY_DATABASE_URI"] = "mysql://root:mysql@localhost/second_flask"
app.config["SQLALCHEMY_TRACK_MODIFICATIONS"] = False
 
db = SQLAlchemy(app)
manager = Manager(app)
# 创建迁移对象
migrate = Migrate(app,db)
# 将迁移文件的命令添加到‘db’中
manager.add_command('db',MigrateCommand)
 
 
class Role(db.Model):
    __tablename__ = "table_roles"
    id = db.Column(db.Integer,primary_key=True)
    name = db.Column(db.String(16),unique=True)
    info = db.Column(db.String(100))
    Users = db.relationship("User",backref='role')
 
 
class User(db.Model):
    __tablename__ = "table_users"
    id = db.Column(db.Integer,primary_key=True)
    name = db.Column(db.String(16),unique=True)
    info = db.Column(db.String(200))
    role_id = db.Column(db.Integer,db.ForeignKey("table_roles.id"))
 
 
@app.route('/')
def hello_world():
    return 'Hello World!'
 
 
if __name__ == '__main__':
    manager.run()
```

使用迁移命令如下：



比如上面的代码所在的文件名称为database.py。

1.python database.py db init  # 生成管理迁移文件的migrations目录

2.python database.py db migrate -m "注释"  # 在migrations/versions中生成一个文件，该文件记录数据表的创建和更新的不同版本的代码。

3.python database.py db upgrade  # 在数据库中生成对应的表格。

4.当需要改表格的时候，改完先执行第二步，然后再执行第三步。

5.需要修改数据表的版本号的时候需要做的操作如下：

  python database.py db upgrade 版本号　　# 向上修改版本号

  python database.py db downgrade 版本号 　# 向下修改版本号

可能用到的其他的语句：

  python database.py db history  　# 查看历史版本号

  python database.py db current 　# 查看当前版本号

## 六、分页

```
Flask``-``SQLAlchemy提供的paginate()方法，返回值是一个Pagination类对象，这个类在Flask``-``SQLAlchemy中定义。这个类包含很多的属性，可以用来在模板中生成分页的链接，因此可以将其作为参数传入模板。
```

### 6.1、paginate

```
paginate(page=None, per_page=None, error_out=True, max_per_page=None)
```


这边说明一下这个方法对应的参数：

- page

  - 指定页码，从`1`开始

- per_page

  - 每一页有几个项

- error_out（默认为True）

  - 是否抛出错误

  - 当其为

    ```
    True
    ```

    时,在以下情况会抛出

    ```
    404
    ```

    - 没有匹配项或者`page`不等于`1`
    - `page`比`1`小或者`per_page`是负数
    - `page`和`per_page`不是整数

  - 当其为

    ```
    False
    ```

    时

    - `page`和`per_page`的默认值分别为`20`和`1`

- max_per_page

  - 当指定了`max_per_page`时，`per_page`会受到这个值的限制（不知道是什么场景下使用，求指点）

### 6.2、Pagination

`Pagination`是调用`paginate`方法后返回的对象。它拥有以下方法，我们可以通过它快速地实现分页的功能。

它拥有以下属性和方法。

- has_next

  - 是否还有下一页

- has_prev

  - 是否还有下一页

- items

  - 当前页的元素集合

- ```
  iter_pages
  ```

  (

  left_edge=2

  , 

  left_current=2

  , 

  right_current=5

  , 

  right_edge=2

  )

  - 迭代分页中的页码。

- next(error_out=False)

  - 返回下一页的`Pagination`对象

- next_num

  - 下一页的页码

- page

  - 当前页的页码

- pages

  - 匹配的元素在当前配置一共有多少页

- per_page

  - 每一页显示的元素个数

- prev(error_out=False)

  - 上一页的`Pagination`对象

- prev_num

  - 上一页的页码

- query

  - 创建`Pagination`对象对应的`query`对象

- total

  - 匹配的元素总数

    ```python
    @admin.route('/movie_list/<int:page>')
    @admin_login_req
    def movielist(page=None):
        if page is None:
            page = 1
        # 多表查询 Movie.tag_id 与 Tag.id 进行关联
        # flask_SQLAlchemy 中的 paginage() 分页方法实现分页
        movie_list = Movie.query.join(Tag).filter(
            Tag.id == Movie.tag_id
        ).order_by(
            Movie.addtime.desc()
        ).paginate(page=page, per_page=1)
        return render('admin/movie_list.html', movie_list=movie_list)
     
    '''
    page_data = Movie.query.join(Tag,Movie.tag_id == Tag.id).order_by(
                    Movie.addtime.desc()
                ).paginate(page=page, per_page=10)
    '''
    ```

    

  - HTML中的应用： 

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

    

  - HTML中使用分页：

    ```html
    {% extends 'admin/admin.html' %}
    {# 通过 import 导入 分页模板 #}
    {% import 'admin/admin.page.html' as pg %}
    {% block content %}
        <section class="content-header">
            <h1>微电影管理系统</h1>
            <ol class="breadcrumb">
                <li><a href="#"><i class="fa fa-dashboard"></i> 电影管理</a></li>
                <li class="active">电影列表</li>
            </ol>
        </section>
        <section class="content" id="showcontent">
            <div class="row">
                <div class="col-md-12">
                    <div class="box box-primary">
                        <div class="box-header">
                            {# 获取字段中设置的项目 #}
                            <h3 class="box-title">{{ form.title.label }}</h3>
                            <div class="box-tools">
                                <div class="input-group input-group-sm" style="width: 150px;">
                                    <input type="text" name="table_search" class="form-control pull-right" placeholder="请输入关键字...">
                                    <div class="input-group-btn">
                                        <button type="submit" class="btn btn-default"><i class="fa fa-search"></i>
                                        </button>
                                    </div>
                                </div>
                            </div>
                        </div>
                        <div class="box-body table-responsive no-padding">
                            <table class="table table-hover">
                                <tbody>
                                <tr>
                                    <th>编号</th>
                                    <th>片名</th>
                                    <th>片长</th>
                                    <th>标签</th>
                                    <th>地区</th>
                                    <th>星级</th>
                                    <th>播放数量</th>
                                    <th>评论数量</th>
                                    <th>上映时间</th>
                                    <th>操作事项</th>
                                </tr>
                                {% for movieitem in movie_list.items %}
                                    <tr>
                                        <td>{{ loop.index }}</td>
                                        {# 设置默认值(value='请输入电影名称') #}
                                        <td>{{ movieitem.title(value='请输入电影名称') }}</td>
                                        <td>{{ movieitem.length }}分钟</td>
                                        {# 获取 Movie表中关联的 Tag表的name属性 #}
                                        <td>{{ movieitem.tag.name }}</td>
                                        <td>{{ movieitem.area }}</td>
                                        <td>{{ movieitem.star }}</td>
                                        <td>{{ movieitem.playnum }}</td>
                                        <td>{{ movieitem.commentnum }}</td>
                                        <td>{{ movieitem.release_time }}</td>
                                        <td>
                                            <a class="label label-success">编辑</a>
                                            &nbsp;
                                            <a class="label label-danger">删除</a>
                                        </td>
                                    </tr>
                                {% endfor %}
                                </tbody>
                            </table>
                        </div>
                        <div class="box-footer clearfix">
                            {# 使用导入的分页模板 #}
                            {{ pg.page(movie_list,'admin.movielist') }}
                        </div>
                    </div>
                </div>
            </div>
        </section>
    {% endblock %}
    ```

    