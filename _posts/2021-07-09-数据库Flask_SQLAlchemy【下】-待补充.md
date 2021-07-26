---
title: 数据库Flask-SQLAlchemy【下】
layout: post
tags: 数据库
categories: ''
---

## 一、基础使用

参考文献：https://blog.csdn.net/u011146423/article/details/87605812

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

## 二、具体操作实例

### 1 、实例代码

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

  