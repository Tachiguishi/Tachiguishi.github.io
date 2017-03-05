---
layout: post
title:  Chapter 12 Followers
date:   2017-02-28 05:09:47 +0800
categories:
  - Reading
  - Flask Web Development by Miguel Grinberg
---

## Database relationships revisited

### many-to-many relationships

```python
registrations = db.Table('registrations',
  db.Column('student_id', db.Integer, db.ForeignKey('students.id')),
  db.Columd('class_id', db.Integer, db.ForeignKey('classes.id')))

class Student(db.Model):
  id = db.Column(db.Integer, primary_key=True)
  name = db.Column(db.String)
  classes = db.relationship('Class',
                            secondary=registrations,
                            backref=db.backref('students', lazy='dynamic'),
                            lazy='dynamic')

class Class(db.Model):
  id = db.Column(db.Integer, primary_key=True)
  name = db.Column(db.String)
```

### self-referential relationships

### advanced many-to-many relationships

`app/model/user.py`

```python
class Follow(db.Model):
  __tablename__ = 'follows'
  follower_id = db.Column(db.Integer, db.ForeignKey('users.id'),
                          primary_key=True)
  followed_id = db.Column(db.Integer, db.ForeignKey('users.id'),
                          primary_key=True)
  timestamp = db.Column(db.DateTime, default=datetime.utcnow)

class User(UserMixin, db.Model):
  #...
  followed = db.relationship('Follow',
                             foreign_keys=[Follow.follower_id],
                             backref=db.backref('follower', lazy='joined'),
                             lazy='dynamic',
                             cascade='all, delete-orphan')
  followers = db.relationship('Follow',
                              foreign_keys=[Follow.followed_id],
                              backref=db.backref('followed', lazy='joined'),
                              lazy='dynamic',
                              cascade='all, delete-orphan')
```

`app/model/user.py`中添加操作方法

```python
class User(UserMixin, db.Model):
  #...
  def follow(self, user):
    if not self.is_following(user):
      f = Follow(follower=self, followed=user)
      db.session.add(f)

  def unfollow(self, user):
    f = self.followed.filter_by(followed_id=user.id).first()
    if f:
      db.session.delete(f)

  def is_following(self, user):
    return self.followed.filter_by(followed_id=user.id).first() is not None

  def is_followed_by(self, user):
    return self.followers.filter_by(follower_id=user.id).first() is not None
```

更新数据库

```shell
python manage.py db migrate -m "follwers"
python manage.py db upgrade
```

## followers on the profile page

`app/templates/user.html`

```liquid
{% raw %}
{% if not current_user.is_following(user) %}
<a href="{{ url_for('.follow', username=user.username) }}"
  class="btn btn-primary">Follow</a>
{% else %}
<a href="{{ url_for('.unfollow', username=user.username) }}"
  class="btn btn-default">Unfollow</a>
{% endif %}
<a href="{{ url_for('.followers', username=user.username) }}">
  Followers:<span class="badge">{{ user.followers.count() }}</span>
</a>
<a href="{{ url_for('.followed_by', username=user.username) }}">
  Following:<span class="badge">{{ user.followed.count() }}</span>
{% if current_user.is_authenticated() and user != current_user and
  user.is_following(current_user) %}
  | <span class="label label-default">Follows you</span>
{% endif %}
{% endraw %}
```

`app/main/views.py`添加`route`

```python
@main.route('/follow/<username>')
@login_required
@permission_required(Permission.FOLLOW)
def follow(username):
  user = User.query.filter_by(username=username).first()
  if user is None:
    flash('Invalid user')
    return redirect(url_for('.index'))
  if current_user.is_following(user):
    flash('You are already following this user')
    return redirect(url_for('.user', username=username))
  current_user.follow(user)
  flash('You are now following %s', %username)
  return redirect(url_for('.user', username=username))
```

```python
@main.rout('/followers/<username>')
def followers(username):
  user = User.query.filter_by(username=username).first()
  if user is None:
    flash('Invalid user')
    return redirect(url_for('.index'))
  page = request.args.get('page', 1, type=int)
  pagination = user.followers.paginate(
    page, per_page=current_app.config['FLASK_FOLLOWERS_PER_PAGE'],
    error_out=False)
  followers = [{'user': item.follower, 'timestamp': item.timestamp}
               for item in pagination.item]
  return render_template('followers.html', user=user, title="Followers of",
                         endpoint='.followers', pagination=pagination,
                         follows=follows)
```

## show followed posts on the home page

`app/models.py`

```python
class User(UserMixin, db.Model):
  #...
  @property
  def followed_posts(self):
    return Post.query.join(Follow, Follow.followed_id == Post.author_id)\
            .filter_by(Follow.follower_id == self.id)
```

`app/main/views.py`

```python
@app.rout('/', methods=['GET', 'POST'])
def index():
  #...
  show_followed = False
  if current_user.is_authenticated():
    show_followed = bool(request.cookies.get('show_followed', ''))
  if show_followed:
    query = current_user.followed_posts
  else:
    query = Post.query
  pagination = query.order_by(Post.timestamp.desc()).paginate(page,
                  per_page=current_app.config['FLASKY_POSTS_PER_PAGE'],
                  error_out=False)
  posts = pagination.items
  return render_template('index.html', form=form, posts=posts,
                        show_followed=show_followed, pagination=pagination)
```

`app/main/views.py`

```python
@main.route('/all')
@login_required
def show_all():
  resp = make_response(redirect(url_for('.index')))
  resp.set_cookie('show_followed', '', max_age=30*24*60*60)
  return resp

@main.route('/followed')
@login_required
def show_followed():
  resp = make_response(redirect(url_for('.index')))
  resp.set_cookie('show_followed', '1', max_age=30*24*60*60)
  return resp
```

将自己关注自己

```python
class User(UserMixin, db.Model):
  #...
  def __init__(self, **kwargs):
    #...
    self.follow(self)

  @staticmethod
  def add_self_follows():
    for user in User.query.all():
      if not user.is_following(user):
        user.follow(user)
        db.session.add(user)
        db.session.commit()
```

然后运行

```shell
python manage.py shell
>>> User.add_self_follows()
```
