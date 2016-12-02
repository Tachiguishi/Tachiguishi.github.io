---
layout: post
title:  "Flask Databases"
date:   2016-12-01 22:09:00 +0800
categories:
  - flask
  - python
---

# Databases

## SQL Databases

关系数据库

## NoSQL Databases

## Flask SQLAlchemy

```bash
(venv) $ pip install flask-sqlalchemy
```

在`Flask-SQLAlchemy`中，数据库使用 URL 指定  

Database | Engine URL
-------- | ----------
MySQL | mysql://username:password@hostname/database
Postgres | postgresql://username:password@hostname/database
SQLite (Unix) | sqlite:////absolute/path/to/database
SQLite (Windows) | sqlite:///c:/absolute/path/to/database

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

  Type name | Python type | Description
  --------- | ----------- | -----------
  Integer | int | 普通整数，一般是 32 位
  SmallInteger | int | 取值范围小的整数，一般是 16 位
  BigInteger | int 或 long | 不限制精度的整数
  Float |float | 浮点数
  Numeric | decimal.Decimal | 定点数
  String | str | 变长字符串
  Text | str | 变长字符串，对较长或不限长度的字符串做了优化
  Unicode | unicode | 变长Unicode字符串
  UnicodeText | unicode | 变长Unicode字符串，对较长或不限长度的字符串做了优化
  Boolean | bool | 布尔值
  Date | datetime.date | 日期
  Time | datetime.time | 时间
  DateTime | datetime.datetime | 日期和时间
  Interval | datetime.timedelta | 时间间隔
  Enum | str | 一组字符串
  PickleType | 任何Python对象 | 自动使用Pickle序列化
  LargeBinary | str | 二进制文件

* SQLAlchemy常用列选项

  Option name | Description
  ----------- | -----------
  primary_key | 如果设为True，这列就是表的主键
  unique | 如果设为True，这列不允许出现重复的值
  index | 如果设为True，为这列创建索引，提升查询效率
  nullable | 如果设为True ，这列允许使用空值；如果设为 False ，这列不允许使用空值
  default | 为这列定义默认值

* SQLAlchemy常用关系选项

  Option name | Description
  ----------- | -----------
  backref | 在关系的另一个模型中添加反向引用
  primaryjoin | 明确指定两个模型之间使用的联结条件。只在模棱两可的关系中需要指定
  lazy | 指定如何加载相关记录。可选值有select（首次访问时按需加载）、immediate（源对象加载后就加载）、 joined （加载记录，但使用联结）、 subquery （立即加载，但使用子查询），noload （永不加载）和 dynamic （不加载记录，但提供加载记录的查询）
  uselist | 如果设为 Fales ，不使用列表，而使用标量值
  order_by | 指定关系中记录的排序方式
  secondary | 指定多对多关系中关系表的名字
  secondaryjoin | SQLAlchemy 无法自行决定时，指定多对多关系中的二级联结条件
