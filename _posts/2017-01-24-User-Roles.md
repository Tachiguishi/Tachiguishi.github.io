---
layout: post
title:  Chapter 9 User Roles
date:   2017-01-24 14:39:00 +0800
categories:
  - Reading
  - Flask
---

## Database Representation of Roles

修改`app/models.py`中的角色表格

```python
class Role(db.Model):
    __tablename__ = 'roles'
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(64), unique=True)
    default = db.Column(db.Boolean, default=False, index=True)
    permissions = db.Column(db.Integer)
    users = db.relationship('User', backref='role', lazy='dynamic')

    def __repr__(self):
        return '<Role %r>' % self.name
```

`default`表示用户注册时默认分配给他的角色。所以应该只有一种角色为`True`，其余均为`False`  
`permissions`字段使用一个整数的二进制位表示其角色所用于的权限，
不同应用所代表的权限不同，本应用的权限含义如下：

| Task name | Bit value | Description |
| :-------- | :-------- | :---------- |
| Follow users | 0b00000001(0x01) | Follow other users |
| Comment on posts made by others | 0b00000010(0x02) | Comment on articles written by others |
| Write articles | 0b00000100(0x04) | Write original articles |
| Moderate comments made by others | 0b00001000(0x08) | Suppress offensive comments made by others |
| Administration access | 0b10000000(0x80) | Administrative access to the site |

`app/models.py`中添加权限定义

```python
class Permission:
    FOLLOW = 0x01
    COMMENT = 0x02
    WRITE_ARTICLES = 0x04
    MODERATE_COMMENTS = 0x08
    ADMINISTER = 0x80
```

角色权限定义

| User role | Permissions | Description |
| :-------- | :---------- | :---------- |
| Anonymous | 0b00000000(0x00) | User who is not logged in. Read-only access to the application. |
| User | 0b00000111(0x07) | Basic permissions to write articles and comments and to follow other users. This is the default for new users. |
| Moderator | 0b00001111(0x0f) | Adds permission to suppress comments deemed offensive or inappropriate. |
| Administrator | 0b11111111(0xff) | Full access, which includes permission to change the roles of other users. |

`app/models.py`

```python
class Role(db.Model):
    # ...
    @staticmethod
    def insert_roles():
        roles = {
            'User': (Permission.FOLLOW |
                     Permission.COMMENT |
                     Permission.WRITE_ARTICLES, True),
            'Moderator': (Permission.FOLLOW |
                          Permission.COMMENT |
                          Permission.WRITE_ARTICLES |
                          Permission.MODERATE_COMMENTS, False),
            'Administrator': (0xff, False)
        }
        for r in roles:
            role = Role.query.filter_by(name=r).first()
            if role is None:
                role = Role(name=r)
            role.permissions = roles[r][0]
            role.default = roles[r][1]
            db.session.add(role)
        db.session.commit()
```

更新数据库模型

```shell
python manage.py db migrate -m "User_roles"
python manage.py db upgrade
```

添加用户角色

```shell
(venv) $ python manage.py shell
>>> Role.insert_roles()
>>> Role.query.all()
[<Role u'Administrator'>, <Role u'User'>, <Role u'Moderator'>]
```

## Role Assignment

注册时为用户分配default角色。但`Administrator`角色比较特殊，需要提前定义。
将管理员的邮箱提前写在配置中，但其注册时为其自动分配管理员权限

`app/models.py`

```python
class User(UserMixin, db.Model):
    # ...
    def __init__(self, **kwargs):
        super(User, self).__init__(**kwargs)
        if self.role is None:
            if self.email == current_app.config['FLASKY_ADMIN']:
                self.role = Role.query.filter_by(permissions=0xff).first()
            if self.role is None:
                self.role = Role.query.filter_by(default=True).first()
    # ...
```

## Role Verification

确认某用户是否拥有某个权限  
`app/models.py`

```python
from flask.ext.login import UserMixin, AnonymousUserMixin

class User(UserMixin, db.Model):
    # ...
    def can(self, permissions):
        return self.role is not None and \
            (self.role.permissions & permissions) == permissions

    def is_administrator(self):
        return self.can(Permission.ADMINISTER)


class AnonymousUser(AnonymousUserMixin):
    def can(self, permissions):
        return False
    def is_administrator(self):
        return False

login_manager.anonymous_user = AnonymousUser
```

添加`app/decorators.py`

```python
from functools import wraps
from flask import abort
from flask_login import current_user
from .modles import Permission


def permission_required(permission):
    def decorator(f):
        @wraps(f)
        def decorated_function(*args, **kwargs):
            if not current_user.can(permission):
                abort(403)
            return f(*args, **kwargs)
        return decorated_function
    return decorator


def admin_required(f):
    return permission_required(Permission.ADMINISTER)(f)
```

`app/main/errors.py`中添加

```python
@main.app_errorhandler(403)
def forbidden(e):
    return render_template('403.html'), 403
```

使用

```python
from decorators import admin_required, permission_required

@main.route('/admin')
@login_required
@admin_required
def for_admins_only():
    return "For administrators!"

@main.route('/moderator')
@login_required
@permission_required(Permission.MODERATE_COMMENTS)
def for_moderators_only():
    return "For comment moderators!"
```
