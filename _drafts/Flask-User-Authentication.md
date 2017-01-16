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

提交登录密令的 POST 请求最后也做了重定向，不过目标 URL 有两种可能。
用户访问未授权的 URL 时会显示登录表单，Flask-Login
会把原地址保存在查询字符串的 next 参数中，这个参数可从 request.args 字典中读取。
如果查询字符串中没有 next 参数，则重定向到首页。
如果用户输入的电子邮件或密码不正确，程序会设定一个 Flash 消息，再次渲染表单，让用户重试登录

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

更新数据库模型
```shell
python manage.py db migrate -m "Login_support"
python manage.py db upgrade
```

手动添加一条用户记录
```shell
(venv) $ python manage.py shell
>>> u = User(email='john@example.com', username='john', password='cat')
>>> db.session.add(u)
>>> db.session.commit()
```

运行测试
```shell
python manage.py runserver
```

## New User Registration

### Adding a User Registration Form

在`app/auth/forms.py`中添加表格
```python
from flask_wtf import Form
from wtforms import StringField, PasswordField, BooleanField, SubmitField
from wtforms.validators import Required, Length, Email, Regexp, EqualTo
from wtforms import ValidationError
from ..models import User


class RegistrationForm(Form):
    email = StringField('Email', validators=[Required(), Length(1,64), Eamil()])
    username = StringField('Username',
                validators=[Required(),
                Length(1,64),
                Regexp('^[A-Za-z][A-Za-z0-9_.]*$',
                        0,
                        'Username must have only letters, '
                        'numbers, dots or underscores')])
    password = PasswordField('Password',
                validators=[Required(),
                            EqualTo('password2',
                                message='Passwords must match.')])
    password2 = PasswordField('Confirm password', validators=[Required()])
    submit = SubmitField('Register')

    def validate_email(self, field):
        if User.query.filter_by(email=field.data).first():
            raise ValidationError('Email already registered.')

    def validate_username(self, field):
        if User.query.filter_by(username=field.data).first():
            raise ValidationError('Username already in use.')
```

添加 `app/templates/auth/register.html`
```html
{% extends "base.html" %}
{% import "bootstrap/wtf.html" as wtf %}

{% block title %}Flask - Register{% endblock %}

{% block page_content %}
<div class="page-header">
    <h1>Register</h1>
</div>
<div class="col-md-4">
    {{ wtf.quick_form(form) }}
</div>
{% endblock %}

```

在`app/templates/auth/login.html`中添加到注册页面的连接
```html
<p>
    New user?
    <a href="{{url_for('auth.register')}}">
        Click here to register
    </a>
</p>
```

### Registering New Users

`app/auth/view.py`中添加注册route
```python
@auth.route('/register', methods['GET', 'POST'])
def register():
    form = RegistrationForm()
    if form.validate_on_submit():
        user = User(email=form.email.data,
                    username=form.username.data,
                    password=form.password.data)
        db.session.add(user)
        flash('You can now login.')
        return redirect(url_for('auth.login'))
    return render_template('auth/register.html', form=form)
```

## Account Confirmation

用户注册时向用户发送一封确认邮件，用户点击邮箱中的确认邮件通过验证

### Generating Confirmation Tokens with itsdangerous

通常需要用户点击一个`http://www.example.com/auth/confirm/<id>`这样的连接即可验证。
但是如果直接这样很容易被用户伪造验证。所以可以通过一些加密方式将`<id>`加密，生成`token`。
这里需要用到`itsdangerous`包

```shell
(venv) $ python manage.py shell
>>> from manage import app
>>> from itsdangerous import TimedJSONWebSignatureSerializer as Serializer
>>> s = Serializer(app.config['SECRET_KEY'], expires_in = 3600)
>>> token = s.dumps({ 'confirm': 23 })
>>> token
'eyJhbGciOiJIUzI1NiIsImV4cCI6MTM4MTcxODU1OCwiaWF0IjoxMzgxNzE0OTU4fQ.ey ...'
>>> data = s.loads(token)
>>> data
{u'confirm': 23}
```

修改`app/models.py`，添加`confirmed`列
```python
from werkzeug.security import generate_password_hash, check_password_hash
from itsdangerous import TimedJSONWebSignatureSerializer as Serializer
from flask import current_app
from flask_login import UserMixin
from . import db, login_manager


class User(UserMixin, db.Model):
    # ...
    confirmed = db.Column(db.Boolean, default=False)

    def generate_confirmation_token(self, expiration=3600):
        s = Serializer(current_app.config['SECRET_KEY'], expiration)
        return s.dumps({'confirm': self.id})

    def confirm(self, token):
        s = Serializer(current_app.config['SECRET_KEY'])
        try:
            data = s.loads(token)
        except:
            return False
        if data.get('confirm') != self.id:
            return False
        self.confirmed = True
        db.session.add(self)
        return True
```

运行`migrage`和`upgrade`命令更新数据库

### Sending Confirmation Emails

`app/auth/view.py`修改注册route
```python
from ..email import send_email

@auth.route('/register', methods = ['GET', 'POST'])
def register():
    form = RegistrationForm()
    if form.validate_on_submit():
        # ...
        db.session.add(user)
        db.session.commit()
        token = user.generate_confirmation_token()
        send_email(user.email, 'Confirm Your Account',
                  'auth/email/confirm', user=user, token=token)
        flash('A confirmation email has been sent to you by email.')
        return redirect(url_for('auth.login'))
    return render_template('auth/register.html', form=form)
```
