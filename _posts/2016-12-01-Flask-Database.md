---
layout: post
title:  Chapter 5 Flask Databases
date:   2016-12-01 22:09:00 +0800
categories:
  - Reading
  - Flask Web Development by Miguel Grinberg
---

## SQL Databases

关系数据库

## NoSQL Databases

## Flask SQLAlchemy

```bash
(venv) $ pip install flask-sqlalchemy
```

在`Flask-SQLAlchemy`中，数据库使用 URL 指定  

| Database | Engine URL |
| :--------| :--------- |
| MySQL | mysql://username:password@hostname/database |
| Postgres | postgresql://username:password@hostname/database |
| SQLite (Unix) | sqlite:////absolute/path/to/database |
| SQLite (Windows) | sqlite:///c:/absolute/path/to/database |

`python`中的数据库配置  

```python
from flask.ext.sqlalchemy import SQLAlchemy

basedir = os.path.abspath(os.path.dirname(__file__))
app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] =\
    'sqlite:///' + os.path.join(basedir, 'data.sqlite')
app.config['SQLALCHEMY_COMMIT_ON_TEARDOWN'] = True
db = SQLAlchemy(app)
```

### Model Definition

模型(model)表示程序中使用的数据持久层实体。
在ORM中则为`python`的一个类，类的属性与数据库表的列对应。

```python
from flask_sqlalchemy import SQLAlchemy

class Role(db.Model):
    __tablename__ = 'roles'
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(64), unique=True)
    users = db.relationship('User', backref='role', lazy='dynamic')

    def __repr__(self):
        return '<Role %r>' % self.name


class User(db.Model):
    __tablename__ = 'users'
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(64), unique=True, index=True)
    role_id = db.Column(db.Integer, db.ForeignKey('roles.id'))

    def __repr__(self):
        return '<User %r>' % self.username
```

* SQLAlchemy常用列类型

  | Type name | Python type | Description |
  | :-------- | :---------- | :---------- |
  | Integer | int | 普通整数，一般是 32 位 |
  | SmallInteger | int | 取值范围小的整数，一般是 16 位 |
  | BigInteger | int 或 long | 不限制精度的整数 |
  | Float |float | 浮点数 |
  | Numeric | decimal.Decimal | 定点数 |
  | String | str | 变长字符串 |
  | Text | str | 变长字符串，对较长或不限长度的字符串做了优化 |
  | Unicode | unicode | 变长Unicode字符串 |
  | UnicodeText | unicode | 变长Unicode字符串，对较长或不限长度的字符串做了优化 |
  | Boolean | bool | 布尔值 |
  | Date | datetime.date | 日期 |
  | Time | datetime.time | 时间 |
  | DateTime | datetime.datetime | 日期和时间 |
  | Interval | datetime.timedelta | 时间间隔 |
  | Enum | str | 一组字符串 |
  | PickleType | 任何Python对象 | 自动使用Pickle序列化 |
  | LargeBinary | str | 二进制文件 |

* SQLAlchemy常用列选项

  | Option name | Description |
  | :---------- | :---------- |
  | primary_key | 如果设为True，这列就是表的主键 |
  | unique | 如果设为True，这列不允许出现重复的值 |
  | index | 如果设为True，为这列创建索引，提升查询效率 |
  | nullable | 如果设为True ，这列允许使用空值；如果设为 False ，这列不允许使用空值 |
  | default | 为这列定义默认值 |

* SQLAlchemy常用关系选项

  | Option name | Description |
  | :---------- | :---------- |
  | backref | 在关系的另一个模型中添加反向引用 |
  | primaryjoin | 明确指定两个模型之间使用的联结条件。只在模棱两可的关系中需要指定 |
  | lazy | 指定如何加载相关记录。可选值有select（首次访问时按需加载）、immediate（源对象加载后就加载）、 joined （加载记录，但使用联结）、 subquery （立即加载，但使用子查询），noload （永不加载）和 dynamic （不加载记录，但提供加载记录的查询） |
  | uselist | 如果设为 Fales ，不使用列表，而使用标量值 |
  | order_by | 指定关系中记录的排序方式 |
  | secondary | 指定多对多关系中关系表的名字 |
  | secondaryjoin | SQLAlchemy 无法自行决定时，指定多对多关系中的二级联结条件 |

## Database Operations

接下来我们将在`python shell`中进行操学习  

`hello.py`  

```python
import os
from flask import Flask, render_template, session, redirect, url_for, flash
from flask_script import Manager
from flask_bootstrap import Bootstrap
from flask_moment import Moment
from flask_wtf import Form
from wtforms import StringField, SubmitField
from wtforms.validators import Required
from flask_sqlalchemy import SQLAlchemy

basedir = os.path.abspath(os.path.dirname(__file__))

app = Flask(__name__)
app.config['SECRET_KEY'] = 'hard to guess string'
app.config['SQLALCHEMY_DATABASE_URI'] =\
    'sqlite:///' + os.path.join(basedir, 'data.sqlite')
app.config['SQLALCHEMY_COMMIT_ON_TEARDOWN'] = True

manager = Manager(app)
bootstrap = Bootstrap(app)
moment = Moment(app)
db = SQLAlchemy(app)


class Role(db.Model):
    __tablename__ = 'roles'
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(64), unique=True)
    users = db.relationship('User', backref='role', lazy='dynamic')

    def __repr__(self):
        return '<Role %r>' % self.name


class User(db.Model):
    __tablename__ = 'users'
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(64), unique=True, index=True)
    role_id = db.Column(db.Integer, db.ForeignKey('roles.id'))

    def __repr__(self):
        return '<User %r>' % self.username


class NameForm(Form):
    name = StringField('What is your name?', validators=[Required()])
    submit = SubmitField('Submit')


@app.errorhandler(404)
def page_not_found(e):
    return render_template('404.html'), 404


@app.errorhandler(500)
def internal_server_error(e):
    return render_template('500.html'), 500


@app.route('/', methods=['GET', 'POST'])
def index():
    form = NameForm()
    if form.validate_on_submit():
        old_name = session.get('name')
        if old_name is not None and old_name != form.name.data:
            flash('Looks like you have changed your name!')
        session['name'] = form.name.data
        return redirect(url_for('index'))
    return render_template('index.html', form=form, name=session.get('name'))


if __name__ == '__main__':
    db.create_all()
    manager.run()
```

### Create Table

`Flask-SQLAlchemy`使用`create_all()`函数

```
(venv) $ python hello.py shell
>>> from hello import db
>>> db.create_all()
```

这是你会看到一个名为`data.sqlite`的文件被创建，其中包含`roles`和`users`两个表单  
如果数据库文件`data.sqlite`已经存，
运行`db.create_all()`并不会重新创建或更新其中的数据表  
可以手动删除所有的数据表然后重新创建，但是这样同时会删除所有的数据记录

```
>>> db.drop_all()
>>> db.create_all()
```

### Insert Rows

```
>>> from hello import Role, User
>>> admin_role = Role(name='Admin')
>>> mod_role = Role(name='Moderator')
>>> user_role = Role(name='User')
>>> user_john = User(username='john', role=admin_role)
>>> user_susan = User(username='susan', role=user_role)
>>> user_david = User(username='david', role=user_role)
```

模型(model)的构造函数接受键/值对的形式赋值的属性  
注意到`User`模型中可以使用`role`来进行复制, 而没有使用其真正的属性`role_id`  
复制给`role`的不是`admin_role.id`而直接是`admin_role`  
模型的主键`id`并没有被人工赋值，这是因为主键是由`Flask-SQLAlchemy`管理的  
由于这时所有实例化的对象都还只存在于`python`之中，并没有被写入数据库，
所以`id`仍未被赋值  

```
>>> print(admin_role.id)
None
>>> print(mod_role.id)
None
>>> print(user_role.id)
None
```

`Flask-SQLAlchemy`通过`session`管理数据库的读写操作  
要将数据写入数据库，手写需要将其添加到`session`中，让后提交

```
>>> db.session.add(admin_role)
>>> db.session.add(mod_role)
>>> db.session.add(user_role)
>>> db.session.add(user_john)
>>> db.session.add(user_susan)
>>> db.session.add(user_david)
```

或者  

```
>>> db.session.add_all([admin_role, mod_role, user_role,
... user_john, user_susan, user_david])
```

最后调用`commit()`函数

```
>>> db.session.commit()
```

> 注意这里的 `db.session` 和 `Flask` 的站点 `session`是完全不同的东西

`commit()`函数写入数据库时，只要发生任何一个错误，
那么添加到`db.session`中的数据都会写入失败  
如果你每次只将相关的数据添加到`db.session`中，将有效的保证数据的一致性

> `db.session.rollback()`

### Update Rows

只用将更新的数据添加到`session`中然后提交即可

```
>>> admin_role.name = 'Administrator'
>>> db.session.add(admin_role)
>>> db.session.commit()
```

### Delete Rows

```
>>> db.session.delete(mod_role)
>>> db.session.commit()
```

### Query Rows

Flask-SQLAlchemy 的每个`model`类都有一个`query`对象，
最基本的模型查询是取回对应表中的所有记录  
注意，这里直接使用模型类调用`query`对象的

```
>>> Role.query.all()
[<Role u'Administrator'>, <Role u'User'>]
>>> User.query.all()
[<User u'john'>, <User u'susan'>, <User u'david'>]
```

使用`filters`进行精确查找

```
>>> User.query.filter_by(role=user_role).all()
[<User u'susan'>, <User u'david'>]
>>> Role.query.filter_by(name='User').first()
<Role u'User'>
```

可以查看`Flask-SQLAlchemy`究竟是使用何种`SQL`语句进行查询的

```
>>> str(User.query.filter_by(role=user_role))
'SELECT users.id AS users_id, users.username AS users_username,
... users.role_id AS users_role_id \nFROM users \nWHERE ? = users.role_id'
```

[SQLAlchemy的详细文档](http://docs.sqlalchemy.org)

* 常用的SQLAlchemy查询过滤器  

| Option | Description |
| :----- | :---------- |
| filter() | 把过滤器添加到原查询上，返回一个新查询 |
| filter_by() | 把等值过滤器添加到原查询上，返回一个新查询 |
| limit() | 使用指定的值限制原查询返回的结果数量，返回一个新查询 |
| offset() | 偏移原查询返回的结果，返回一个新查询 |
| order_by() | 根据指定条件对原查询结果进行排序，返回一个新查询 |
| group_by() | 根据指定条件对原查询结果进行分组，返回一个新查询 |


* 最常使用的SQLAlchemy查询执行函数

| Option | Description |
| :----- | :---------- |
| all() | 以列表形式返回查询的所有结果 |
| first() | 返回查询的第一个结果，如果没有结果，则返回 None |
| first_or_404() | 返回查询的第一个结果，如果没有结果，则终止请求，返回 404 错误响应 |
| get() | 返回指定主键对应的行，如果没有对应的行，则返回 None |
| get_or_404() | 返回指定主键对应的行，如果没找到指定的主键，则终止请求，返回 404 错误响应 |
| count() | 返回查询结果的数量 |
| paginate() | 返回一个 Paginate 对象，它包含指定范围内的结果 |

关系查询

```
>>> user_role = Role.query.filter_by(name='User').first()
>>> user_role
<Role u'User'>
>>> users = user_role.users
>>> users
<sqlalchemy.orm.dynamic.AppenderBaseQuery object at 0xff5ac04c>
```

这里在执行`user_role.users`查询时出错  
这跟我们定义`Role`类时添加的`lazy = dynamic`参数有关

该用以下方式执行即可
```
>>> user_role.users.order_by(User.username).all()
[<User u'david'>, <User u'susan'>]
>>> user_role.users.count()
2
```

## Database Use in View Functions

视图函数

```python
@app.route('/', methods=['GET', 'POST'])
def index():
    form = NameForm()
    if form.validate_on_submit():
        user = User.query.filter_by(username=form.name.data).first()
        if user is None:
            user = User(username=form.name.data)
            db.session.add(user)
            session['known'] = False
        else:
            session['known'] = True
        session['name'] = form.name.data
        return redirect(url_for('index'))
    return render_template('index.html', form=form, name=session.get('name'),
                           known=session.get('known', False))
```

`templates/index.html`
```liquid
{% raw %}
{% extends "base.html" %}
{% import "bootstrap/wtf.html" as wtf %}

{% block title %}Flasky{% endblock %}

{% block page_content %}
<div class="page-header">
    <h1>Hello, {% if name %}{{ name }}{% else %}Stranger{% endif %}!</h1>
    {% if not known %}
    <p>Pleased to meet you!</p>
    {% else %}
    <p>Happy to see you again!</p>
    {% endif %}
</div>
{{ wtf.quick_form(form) }}
{% endblock %}
{% endraw %}
```

## Integration with the Python Shell

```python
from flask.ext.script import Shell

def make_shell_context():
    return dict(app=app, db=db, User=User, Role=Role)

manager.add_command("shell", Shell(make_context=make_shell_context))
```

`make_shell_context()`函数注册了程序、数据库实例以及模型，
因此这些对象能直接导入`shell`

```
$ python hello.py shell
>>> app
<Flask 'app'>
>>> db
<SQLAlchemy engine='sqlite:////home/flask/flasky/data.sqlite'>
>>> User
<class 'app.User'>
```

## Database Migrations with Flask-Migrate

Flask-SQLAlchemy 只有在数据库表格不存在时才会创建表，
这就表示如果想更新表的结构就必须删除原表然后重建。但这样同时也会删除原表中的所有数据

SQLAlchemy 的开发者根据代码的版本控制原理写了[Alembic](http://bit.ly/alembic-doc)
来管理数据库，我们可以使用它的Flask版本`Flask-Migrate`

### Creating a Migration Repository

```shell
(venv) $ pip install flask-migrate
```

```python
from flask.ext.migrate import Migrate, MigrateCommand
# ...
migrate = Migrate(app, db)
manager.add_command('db', MigrateCommand)
```

然后在命令行中运行

```shell
(venv) $ python hello.py db init
```

该命令会生成`migrations`文件夹，其中的文件和你的其它代码文件一样需要添加到你的版本控制器中

### Creating a Migration Script

在 Alembic 中数据库的变更使用`migration script`表示。
脚本有两个函数`upgrade()`和`downgrade()`  
可以使用`revision`命令手动生成脚本，也可以使用`migrate`命令自动生成脚本。  
手动生成的脚本只有一个框架，`upgrade()`和`downgrade()`都是空函数，需要自己手动补充。
自动生成的脚本并不一定会完全准确，所以在生活后一定要检查一遍  


```shell
(venv) $ python hello.py db migrate -m "initial migration"
```

该命令会在`migrations/versions`文件夹下生成一个脚本。
脚本名为12位随机字符加上`-m`参数后跟的字符串  
每次对数据库表进行修改后都需要运行该命令产生`migration script`，然后运行`upgrade()`命令更新实际的数据库

### Upgrading the Database

一旦确定`magration script`准确无误，就可以运行`upgrade()`将数据库运用到程序中

```shell
(venv) $ python hello.py db upgrade
```

如果是第一次运行`upgrade`其效果和`db.create_all()`相同
