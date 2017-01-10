---
layout: post
title:  Chapter 8 User Authentication
date:   2016-12-30 05:53:00 +0800
categories:
  - Reading
  - Flask Web Development by Miguel Grinberg
---

## Flask 的 Authentication 拓展

python 有很多用于用户验证的库，但是没有一个是包含所有相关功能的，
但我们可以组合使用来实现所有的功能

* Flask-Login: Management of user sessions for logged-in users
* Werkzeug: Password hashing and verification
* itsdangerous: Cryptographically secure token generation and verification

## 密码的安全性

为了保护用户的密码安全，不能直接将用户密码存储在数据库中，而是存储密码对应的哈希值。
并且使任何人多无法获取到密码的原值是多少。  
有关密码的哈希算法可以参考[Salted Password Hashing - Doing it Right](http://bit.ly/saltedpass)

### Werkzeug 的密码哈希值

Werkzeug 的 security 模块可以很方便的进行密码的哈希计算。
它只开放了两个函数接口，更别用于注册和验证时  

* `generate_password_hash(password, method=pbkdf2:sha1, salt_length=8)`
* `check_password_hash(hash, password)`

更改`User`Model
`app/model.py`
```python
from werkzeug.security import generate_passward_hash, check_password_hash
from . import db

class User(db.Model):
    __tablename__ = 'users'
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(64), unique=True, index=True)
    role_id = db.Column(db.Integer, db.ForeignKey('roles.id'))
    password_hash = db.Column(db.String(128))

    @property
    def password(self):
        raise AttributeError('password is not a readable attribute')

    @password.setter
    def password(self, password):
        self.password_hash = generate_passward_hash(password)

    def verify_password(self, password):
        return check_password_hash(self.password_hash, password)

    def __repr__(self):
        return '<User %r>' % self.username
```

运行测试：
```shell
(venv) $ python manage.py shell
>>> u = User()
>>> u.password = 'cat'
>>> u.password_hash
'pbkdf2:sha1:1000$Q4LHPUxb$ee691f2fbaad91e669189481132be027db03b214'
>>> u.verify_password('cat')
True
>>> u.verify_password('dog')
False
>>> u1 = User()
>>> u1.password = 'cat'
>>> u1.password_hash
'pbkdf2:sha1:1000$l0lf8OPL$b18e74965913a56aa27c1fbf127cb751dab41581'
```
注意虽然`u`和`u2`的密码相同，但产生的哈希值却是不同的

可以为此写个单元测试，避免手动测试
`tests/test_user_model.py`
```python
import unittest
from app import create_app, db
from app.models import User


class UserModelTestCase(unittest.TestCase):
    def setUp(self):
        self.app = create_app('testing')
        self.app_context = self.app.app_context()
        self.app_context.push()
        db.create_all()

    def tearDown(self):
        db.session.remove()
        db.drop_all()
        self.app_context.pop()

    def test_password_setter(self):
        u = User(password='cat')
        self.assertTrue(u.password_hash is not None)

    def test_no_password_getter(self):
        u = User(password='cat')
        with self.assertRaises(AttributeError):
            u.password

    def test_password_verification():
        u = User(password='cat')
        self.assertTrue(u.verify_password('cat'))
        self.assertFalse(u.verify_password('dog'))

    def test_password_salts_are_random(self):
        u = User(password='cat')
        u2 = User(password='cat')
        self.assertTrue(u.password_hash != u2.password_hash)
```

## Creating an Authentication Blueprint

`app/auth/__init__.py`
```python
from flask import Blueprint

auth = Blueprint('auth', __name__)

from . import views
```

`app/auth/views.py`
```python
from flask import render_template
from . import auth

@auth.route('/login')
def login():
    return render_template('auth/login.html')
```

`app/__init__.py`
```python
def create_app(config_name):
    # ...

    from .auth import auth as auth_blueprint
    app.register_blueprint(auth_blueprint, url_prefix='/auth')

    return app
```

## User Authentication with Flask-Login

安装环境
```shell
pip install flask-login
```
