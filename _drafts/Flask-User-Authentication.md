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

| Method | Description |
| :----- | :---------- |
| is_authenticated() | Must return True if the user has login credentials or False otherwise. |
| is_active() | Must return True if the user is allowed to log in or False otherwise. A False return value can be used for disabled accounts. |
| is_anonymous() | Must always return  False for regular users. |
| get_id() | Must return a unique identifier for the user, encoded as a Unicode string. |

为了方便使用，Flask-Login提供了`UserMixin`类自动调用以上函数。

### Preparing the User Model for Logins

修改后的`app/models.py`
```python
from flask_login import UserMixin

class User(UserMixin, db.Model):
    __tablename__ = 'users'
    id = db.Column(db.Integer, primary_key=True)
    email = db.Column(db.String(64), unique=True, index=True)
    username = db.Column(db.String(64), unique=True, index=True)
    role_id = db.Column(db.Integer, db.ForeignKey('roles.id'))
    password_hash = db.Column(db.String(128))
```

在工厂初始化函数中初始化Flask-Login
`app/__init__.py`
```python
from flask.ext.login import LoginManager

login_manager = LoginManager()
login_manager.session_protection = 'strong'
login_manager.login_view = 'auth.login'

def create_app(config_name):
    # ...
    login_manager.init_app(app)
    # ...
```

获取用户信息接口
`app/models.py`
```python
from . import login_manager

@login_manager.user_loader
def load_user(user_id):
    return User.query.get(int(user_id))
```

### Protecting Routes

添加`login_required`修饰知名某页面只有在登录后才能访问，例如：
```python
from flask.ext.login import login_required

@app.route('/secret')
@login_required
def secret():
    return 'Only authenticated users are allowed!'
```

### Adding a Login Form

添加登录表单`app/auth/forms.py`
```python
from flask_wtf import Form
from wtforms import StringField, PasswordField, BooleanField, SubmitField
from wtforms.validators import Required, Length, Email

class LoginForm(Form):
    email = StringField('Email', validators=[Required(), Length(1,64), Email()])
    password = PasswordField('Password', validators=[Required()])
    rember_me = BooleanField('Keep me logged in')
    submit = SubmitField('Log in')
```

修改模板`app/templates/base.html`，添加语句
```html
<ul class="nav navbar-nav navbar-right">
    {% if current_user.is_authenticated() %}
    <li><a href="{{ url_for('auth.logout') }}">Sign Out</a></li>
    {% else %}
    <li><a href="{{ url_for('auth.login') }}">Sign In</a></li>
    {% endif %}
</ul>
```

### Signing Users In

`app/auth/views.py`
```python
from flask import render_template, redirect, request, url_for, flash
from flask_login import login_user, logout_user, login_required
from . import auth
from ..models import User
from .forms import LoginForm

@auth.route('/login', methods=['GET', 'POST'])
def login():
    form = LoginForm()
    if form.validate_on_submit():
        user = User.query.filter_by(email=form.email.data).first()
        if user is not None and user.verify_password(form.password.data):
            login_user(user, form.remember_me.data)
            return redirect(request.args.get('next'))
        flash('Invalid username or password')
    return render_template('auth/login.html', form=form)
```

`app/templates/auth/login.html`
```html
{% extends "base.html" %}
{% import "bootstrap/wtf.html" as wtf %}

{% block title %}Flasky - Login{% endblock %}

{% block page_content %}
<div class="page-header">
    <h1>Login</h1>
</div>
<div class="col-md-4">
    {{ wtf.quick_form(form) }}
</div>
{% endblock %}
```

### Signing Users Out

`app/auth/views.py`
```python
@auth.route('/logout')
@login_required
def logout():
    logout_user()
    flash('You have been logged out')
    return redirect(url_for('main.index'))
```

### Testing Logins

`app/templates/index.html`
```html
{% extends "base.html" %}

{% block title %}Flasky{% endblock %}

{% block page_content %}
<div class="page-header">
    <h1>
    Hello,
    {% if current_user.is_authenticated %}
        {{ current_user.username }}
    {% else %}
        Stranger
    {% endif %}
    !
    </h1>
</div>
{% endblock %}
```
