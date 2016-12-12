---
layout: post
title:  "Flask Email"
date:   2016-12-13 06:18:00 +0800
categories:
  - Reading
  - Flask Web Development by Miguel Grinberg
---

我们可以使用 python 的`smtplib`库，但是使用集成的`flask-mail`更方便

## Basic

### 安装

```shell
(venv) $ pip install flask-mail
```

### 配置

Key Default Description
MAIL_HOSTNAME
localhost Hostname or IP address of the email server
MAIL_PORT
25 Port of the email server
MAIL_USE_TLS False
Enable Transport Layer Security (TLS) security
MAIL_USE_SSL False
Enable Secure Sockets Layer (SSL) security
MAIL_USERNAME None
Mail account username
MAIL_PASSWORD None
Mail account password

```python
import os
# ...
app.config['MAIL_SERVER'] = 'smtp.googlemail.com'
app.config['MAIL_PORT'] = 587
app.config['MAIL_USE_TLS'] = True
app.config['MAIL_USERNAME'] = os.environ.get('MAIL_USERNAME')
app.config['MAIL_PASSWORD'] = os.environ.get('MAIL_PASSWORD')
```

邮箱的用户名和密码属于敏感信息，最好不要直接写在代码中，可以定义环境变量  
(?如何定义环境变量?)(主流邮箱的配置)

```shell
# Linux or Mac OS
(venv) $ export MAIL_USERNAME=<Gmail username>
(venv) $ export MAIL_PASSWORD=<Gmail password>
# Windows
(venv) $ set MAIL_USERNAME=<Gmail username>
(venv) $ set MAIL_PASSWORD=<Gmail password>
```

## Sending Email from the Python Shell

```shell
(venv) $ python hello.py shell
>>> from flask.ext.mail import Message
>>> from hello import mail
>>> msg = Message('test subject', sender='you@example.com',
... recipients=['you@example.com'])
>>> msg.body = 'text body'
>>> msg.html = '<b>HTML</b> body'
>>> with app.app_context():
... mail.send(msg)
```

### Integrating Emails with the Application

定义`send_email`函数

```python
from flask.ext.mail import Message

app.config['FLASKY_MAIL_SUBJECT_PREFIX'] = '[Flasky]'
app.config['FLASKY_MAIL_SENDER'] = 'Flasky Admin <flasky@example.com>'

def send_email(to, subject, template, **kwargs):
    msg = Message(app.config['FLASKY_MAIL_SUBJECT_PREFIX'] + subject,
    sender=app.config['FLASKY_MAIL_SENDER'], recipients=[to])
    msg.body = render_template(template + '.txt', **kwargs)
    msg.html = render_template(template + '.html', **kwargs)
    mail.send(msg)
```

使用

```python
# ...
app.config['FLASKY_ADMIN'] = os.environ.get('FLASKY_ADMIN')
# ...
@app.route('/', methods=['GET', 'POST'])
def index():
    form = NameForm()
    if form.validate_on_submit():
        user = User.query.filter_by(username=form.name.data).first()
        if user is None:
            user = User(username=form.name.data)
            db.session.add(user)
            session['known'] = False
            if app.config['FLASKY_ADMIN']:
                send_email(app.config['FLASKY_ADMIN'], 'New User',
                           'mail/new_user', user=user)
        else:
            session['known'] = True
        session['name'] = form.name.data
        return redirect(url_for('index'))
    return render_template('index.html', form=form, name=session.get('name'),
                           known=session.get('known', False))
```

## 异步发送邮件

发送邮件可能会耗费一些时间，为了用户界面的流畅性，可以在后台另起线程发送

```python
from threading import Thread

def send_async_email(app, msg):
    with app.app_context():
        mail.send(msg)

def send_email(to, subject, template, **kwargs):
    msg = Message(app.config['FLASKY_MAIL_SUBJECT_PREFIX'] + ' ' + subject,
                  sender=app.config['FLASKY_MAIL_SENDER'], recipients=[to])
    msg.body = render_template(template + '.txt', **kwargs)
    msg.html = render_template(template + '.html', **kwargs)
    thr = Thread(target=send_async_email, args=[app, msg])
    thr.start()
    return thr
```

`flask-mail`的`sned()`函数`current_app`，所以需要计划`app_context`

```python
with app.app_context():
    mail.send(msg)
```
