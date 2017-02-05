---
layout: post
title:  Chapter 10 User Profiles
date:   2017-02-04 12:56:00 +0800
categories:
  - Reading
  - Flask Web Development by Miguel Grinberg
---

## Profile Information

为`User`表单添加相关条目

`app/models.py`

```python
from datetime import datetime

class User(UserMixin, db.Model):
    __tablename__ = 'users'
    # ...
    name = db.Column(db.String(64))
    location = db.Column(db.String(64))
    about_me = db.Column(db.Text())
    member_since = db.Column(db.DateTime(), default=datetime.utcnow)
    last_seen = db.Column(db.DateTime(), default= datetime.utcnow)
```

`db.String`和`db.Text`的区别在于`db.Text`没有长度上限  
两个时间字段的默认值为`datetime.utcnow`，注意这里没有`()`，这是因为`db.Column`接受函数作为默认值  
`member_since`字段只需要初始化一次即可，但`last_seen`字段需要在用户每次登录的时候更新  
在`app/models.py`中添加函数

```python
class User(UserMixin, db.Model):
    # ...

    def ping(self):
        self.last_seen = datetime.utcnow()
        db.session.add(self)
```

在用户每次放松请求时都需要调用`ping()`函数，所以可以将其放在`before_app_request`中调用  
`app/auth/views.py`

```python
@auth.before_app_request
def before_request():
    if current_user.is_authenticated:
        current_user.ping()
        if not current_user.confirmed \
                and request.endpoint[:5] != 'auth.' \
                and request.endpoint != 'static':
            return redirect(url_for('auth.unconfirmed'))
```

## User Profile Page

`app/main/views.py`

```python
@main.route('/user/<username>')
def user(username):
    user = User.query.filter_by(username=username).first_or_404()
    return render_template('user.html', user=user)
```

添加模版`app/templates/user.html`

```liquid
{% raw %}
{% extends "base.html" %}

{% block title %}Flasky - {{ user.username}}{% endblock %}

{% block page_content %}
<div class="page-header">
    <h1>{{ user.username }}</h1>
    {% if user.name or user.location %}
      <p>
        {% if user.name %}{{ user.name}}{% endif %}
        {% if user.location %}
          From
          <a href="http://maps.google.com/?q={{ user.location }}">
            {{ user.location }}
          </a>
        {% endif %}
      </p>
    {% endif %}

    {% if current_user.is_administrator() %}
      <p><a href="mailto:{{ user.email }}">{{ user.email }}</a></p>
    {% endif %}

    <% if user.about_me %><p>{{ user.about_me }}</p>{% endif %}

    <p>
      Member since {{ moment(user.member_since).format('L')}}.
      Last seen {{ moment(user.last_seen).fromNow() }}.
    </p>
</div>
{% endblock %}
{% endraw %}
```

在导航条中添加到用户信息页面的链接

`app/templates/base.html`

```liquid
{% raw %}
{% if current_user.is_authenticated %}
  <li>
    <a href="{{ url_for('main.user', username=current_user.username) }}">
      Profile
    </a>
  </li>
{% endif %}
{% endraw %}
```

更新数据库模型

```shell
python manage.py db migrate -m "user_info"
python manage.py db upgrade
```

## Profile Editor

修改用户信息有两种情况，一种是用户自己填写自己的信息，另一种是管理员修改所有用户信息

### User-Level Profile Editor

`app/main/forms.py`中添加Profile编辑表格

```python
class EditProfileForm(Form):
    name = StringField('Real name', validators=[Length(0, 64)])
    location = StringField('Location' validators=[Length(0, 64)])
    about_me = TextAreaFeild('About me')
    submit = SubmitField('Submit')
```

`app/main/views.py`中添加route

```python
@main.route('/edit_profile', methods=['GET', 'POST'])
@login_required
def edit_profile():
    form = EditProfileForm()
    if form.validate_on_submit():
        current_user.name = form.name.data
        current_user.location = form.location.data
        current_user.about_me = form.about_me.data
        db.session.add(current_user)
        flash('Your profile has been updated.')
        return redirect(url_for('.user', username=current_user.username))
    form.name.data = current_user.name
    form.location.data = current_user.location
    form.about_me = current_user.about_me
    return render_template('edit_profile', form=form)
```

`app/templates/user.html`中添加到`edit_profile`的链接

```liquid
{% raw %}
{% if user == current_user %}
  <a class="btn btn-default" href="{{ url_for('.edit_profile') }}">
    Edit Profile
  </a>
{% endif %}
{% endraw %}
```

添加模版`app/templates/edit_profile.html`

```liquid
{% raw %}
{% extends "base.html" %}
{% import "bootstrap/wtf.html" as wtf %}

{% block title %}Flasky - Edit Profile{% endblock %}

{% block page_content %}
<div class="page-header">
  <h1>Edit your profile</h1>
</div>
<div class="col-md-4">
  {{ wtf.quick_form(form) }}
</div>
{% endblock %}
{% endraw %}
```

### Administrator-Level Profile Editor

`app/main/forms.py`中添加管理员修改`form`

```python
class EditProfileAdminForm(Form):
    email = StringField('Email',
                        validators=[Required(), Length(1, 64), Email()])
    username = StringField('Username',
                  validators=[Required(),
                              Length(1, 64),
                              Regexp('^[A-Za-z][A-Za-z0-9_.]*$',
                                    0,
                                    'Usernames must have only letters, '
                                    'numbers, dots or underscores')])
    confirmed = BooleanField('Confirmed')
    role = SelectField('Role', coerce=int)
    name = StringField('Real name', validators=[Length(0, 64)])
    location = StringField('Location', validators=[Length(0, 64)])
    about_me = TextAreaFeild('About me')
    submit = SubmitField('Submit')

    def __init__(self, user, *args, **kwargs):
        super(EditProfileAdminForm, self).__init__(*args, **kwargs)
        self.role.choices = [(role.id, role.name)
                              for role in Role.query.order_by(Role.name).all()]
        self.user = user

    def validate_email(self, field):
        if field.data != self.user.email \
                and User.query.filter_by(email=field.data).first():
            raise ValidationError('Email already registered')

    def validate_username(self, field):
        if field.data != self.user.username \
                and User.query.filter_by(username=field.data).first():
            raise ValidationError('Username already in use.')
```

`app/main/views.py`中添加`route`

```python
@main.route('/edit_profile/<int:id>', methods=['GET', 'POST'])
@login_required
@admin_required
def edit_profile_admin(id):
    user = User.query.get_or_404(id)
    form = EditProfileAdminForm(user=user)
    if form.validate_on_submit():
        user.email = form.email.data
        user.username = form.username.data
        user.confirmed = form.confirmed.data
        user.role = Role.query.get(form.role.data)
        user.name = form.name.data
        user.location = form.location.data
        user.about_me = form.about_me.data
        db.session.add(user)
        return redirect(url_for('.user', username=user.username))

    form.email.data = user.email
    form.username.data = user.username
    form.confirmed.data = user.confirmed
    form.role.data = user.role_id
    form.name.data = user.name
    form.location.data = user.location
    form.about_me = user.about_me
    return render_template('edit_profile.html', form=form, user=user)
```

`app/templates/user.html`中添加到编辑页面的链接

```liquid
{% raw %}
{% if current_user.is_administrator() %}
  <a class="btn btn-danger"
      href="{{ url_for('.edit_profile_admin', id=user.id) }}">
    Edit Profile [Admin]
  </a>
{% endif %}
{% endraw %}
```

## User Avatar

本节需要使用[Gravatar](http://gravatar.com)

Gravatar查询参数

| Argument name | Description |
| :------------ | :---------- |
| s | image size, in pixels |
| r | image rating. Options are "g", "pg", "r", and "x" |
| d | the default image generator for users who have no avatar registered with Gravatar Service. Opthions are "404" to return a 404 error, a URL that points to a default image, or one of the following image generators: "mm", "identicon", "monsterid", "wavatar", "retro" or "blank" |
| fd | force the use of default avatar |

可以将产生 Gravatar URL 的方式放在`app/models.py`中

```python
import hashlib
from flask import request

class User(UserMixin, db.Model):
    # ...
    def gravatar(self, size=100, default='identicon', rating='g'):
        if request.is_secure:
            url = 'https://secure.gravatar.com/avatar'
        else:
            url = 'http://www.gravatar.com/avatar'
        hash = hashlib.md5(self.email.encode('utf-8')).hexdigest()
        return '{url}/{hash}?s={size}&d={default}&r={rating}'.format(
                url=url, hash=hash, size=size, default=default, rating=rating)
```

添加css文件`app/static/styles.css`

```css
.profile-thumbnail{
  position: absolute;
}

.profile-header{
  min-height: 260px;
  margin-left: 280px;
}
```

`Jinja2`模版中可以直接调用`gravatar()`函数

`app/templates/user.html`

```liquid
{% raw %}
<img class="img-rounded profile-thumbnail" src="{{ user.gravatar(size=256)}}">
{% endraw %}
```

`app/templates/base.html`导航条中添加头像缩略图

```liquid
{% raw %}
<link rel="stylesheet" type="text/css"
  href="{{ url_for('static', filename='styles.css')}}">

<ul class="nav navbar-nav navbar-right">
  {% if current_user.is_authenticated %}
    <li class="dropdown">
      <a href="#" class="dropdown-toggle" data-toggle="dropdown">
        <img src="{{ current_user.gravatar(size=18) }}"
        Account <b class="caret"></b>
      </a>
      <ul class="dropdown-menu">
        <li><a href="{{ url_for('auth.logout') }}">Log out</a></li>
      </ul>
    </li>
  {% else %}
    <li><a href="{{ url_for('auth.login') }}">Log in</a></li>
  {% endif%}
</ul>
{% endraw %}
```

生成md5比较占用资源，但是可以提前生成放入数据库中缓存

`app/models.py`

```python
class User(Usermixin, db.Model):
    # ...
    avatar_hash = db.Column(db.String(32))

    def __init__(self, **kwargs):
        #...
        if self.email is not None and self.avatar_hash is None:
            self.avatar_hash = hashlib.md5(
                                  self.email.encode('utf-8')).hexdigest()

    def gravatar(self, size=100, default='identicon', rating='g'):
        if request.is_secure:
            url = 'https://secure.gravatar.com/avatar'
        else:
            url = 'http://www.gravatar.com/avatar'
        hash = self.avatar_hash or hashlib.md5(
                                      self.email.encode('utf-8')).hexdigest()
        return '{url}/{hash}?s={size}&d={default}&r={rating}'.format(
                url=url, hash=hash, size=size, default=default, rating=rating)
```
