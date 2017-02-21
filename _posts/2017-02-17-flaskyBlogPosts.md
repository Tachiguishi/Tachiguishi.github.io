---
layout: post
title:  Chapter 11 Blog Posts
date:   2017-02-17 10:18:00 +0800
categories:
  - Reading
  - Flask Web Development by Miguel Grinberg
---

## Blog Post Submission and Display

`app/models.py`中添加`Post`表

```python
class Post(db.Model):
  __tablename__ = 'posts'
  id = db.Column(db.Integer, primary_key=True)
  body = db.Column(db.Text)
  timestamp = db.Column(db.DateTime, index=True, default=datetime.utcnow)
  author_id = db.Column(db.Integer, db.ForeignKey('users.id'))

class User(UserMixin, db.Model):
  # ...
  posts = db.relationship('Post', backref='author', lazy='dynamic')
```

`app/main/forms.py`中添加表单

```python
class PostForm(Form):
  body = TextAreaField("what's on you mind?", validators=[Required()])
  submit = SubmitField('Submit')
```

`app/main/views.py`

```python
@main.route('/', methods=['GET', 'POST'])
def index():
  form = PostForm()
  if current_user.can(Permission.WRITE_ARTICLES) and \
          form.validate_on_submit():
    post = Post(body=form.body.data,
                author=current_user._get_current_object())
    db.session.add(post)
    return redirect(url_for('.index'))
  posts = Post.query.order_by(Post.timestamp.desc()).all()
  return render_template('index.html', form=form, posts=posts)
```

`app/templates/index.html`

```liquid
{% raw %}
{% extends "base.hmtl" %}
{% import "bootstrap/wtf.hmtl" as wtf %}

<div>
  {% if current_user.can(Permission.WRITE_ARTICLES) %}
    {{ wtf.quick_form(form) }}
  {% endif %}
</div>
<ul class="posts">
  {% for post in posts %}
  <li class="post">
    <div class="profile-thumbnail">
      <a href="{{ url_for('.user', username=post.author.username) }}">
        <img class="img-rounded profile-thumbnail"
             src="{{ post.author.gravatar(size=40) }}">
      </a>
    </div>
    <div class="post-date">{{ moment(post.timestamp).fromNow() }}</div>
    <div class="post-author">
      <a href="{{ url_for('.user',  username=post.author.username) }}">
        {{ post.author.username }}
      </a>
    </div>
    <div class="post-body">{{ post.body }}</div>
  </li>
  {% endfor %}
</ul>
{% endraw %}
```

`app/main/__init__.py`中添加语句

```python
from ..models import Permission

@main.app_context_processor
def inject_permissions():
  return dict(Permission=Permission)
```

在`app/manage.py`中添加语句

```python
from app.models import Permission, Post

def make_shell_context():
  return dict(app=app, db=db, User=User, Role=Role,
              Permission=Permission, Post=Post)
```

更新数据库

```shell
python manage.py db migrate -m "postBlog"
python manage.py db upgrade
```

## Blog Posts on Profile Pages

`app/main/views.py`

```python
@main.route('/user/<username>')
def user(username):
  user = User.query.filter_by(username=username).first_or_404()
  posts = user.posts.order_by(Post.timestamp.desc()).all()
  return render_template('user.html', user=user, posts=posts)
```

然后在`app/templates/user.html`中做响应修改。由于其中需要添加的语句和`app/templates/index.html`
中的完全相同，所以可以提取出来放入单独的一个文件如`_posts.html`中，然后使用`include()`语句引用

`app/templates/_posts.html`

```liquid
{% raw %}
<ul class="posts">
  {% for post in posts %}
  <li class="post">
    <div class="profile-thumbnail">
      <a href="{{ url_for('.user', username=post.author.username) }}">
        <img class="img-rounded profile-thumbnail"
             src="{{ post.author.gravatar(size=40) }}">
      </a>
    </div>
    <div class="post-date">{{ moment(post.timestamp).fromNow() }}</div>
    <div class="post-author">
      <a href="{{ url_for('.user',  username=post.author.username) }}">
        {{ post.author.username }}
      </a>
    </div>
    <div class="post-body">{{ post.body }}</div>
  </li>
  {% endfor %}
</ul>
{% endraw %}
```

`app/templates/user.html`

```liquid
{% raw %}
<h3>Posts by {{ user.username }}</h3>
{% include '_posts.html' %}
{% endraw %}
```

`app/templates/index.html`中也可以做类似的修改

## Paginating Long Blog Post Lists

如果博客数量过多则列表显示则很长，则可以考虑分页显示

### Creating Fake Blog Post data

为了测试大量数据时的情况，可是使用`forgerypy`来生成所需的数据

```shell
pip install forgerypy
```

由于`forgerypy`只在开发测试时需要，所以可以将项目所需依赖相分成`dev.txt`和`prod.txt`两个。
而对于两者共有的依赖库可以放在`common.txt`中，然后在`dev.txt`和`prod.txt`中的列表前面添加
`-r common.txt`来解决

`app/models.py`中添加生成测试数据的函数

```python
class User(UserMixin, db.Model):
  #...
  @staticmethod
  def generate_fake(count=100):
    from sqlalchemy.exc import IntegrityError
    from random import seed
    import forgerypy

    seed()
    for i in range(count):
      u = User(email=forgery_py.internet.emial_address(),
               username=forgery_py.internet.user_name(True),
               password=forgery_py.lorem_ipsum.word(),
               confirmed=True,
               name=forgery_py.name.full_name(),
               location=forgery_py.address.city(),
               about_me=forgery_py.lorem_ipsum.sentence(),
               member_since=forgery_py.date.date(True))
      db.session.add(u)
      try:
        db.session.commit()
      except IntegrityError:
        db.session.rollback()

class Post(db.Model):
  #...
  @staticmethod
  def generate_fake(count=100):
    from random import seed, randint
    import forgery_py

    seed()
    user_count = User.query.count()
    for i in range(count):
      u = User.query.offset(randint(0, user_count - 1)).first()
      p = Post(body=forgery_py.lorem_ipsum.sentences(randint(1, 3)),
               timestamp=forgery_py.date.date(True),
               author=u)
      db.session.add(p)
      db.session.commit()
```

`forgerypy`产生信息是完全随机的，所以在生成用户时可能会生成相同的用户不符合主键唯一原则造成插入失败，
所以使用`try - except`语句

调用函数生成数据

```shell
python manage.py shell
User.generate_fake(100)
Post.generate_fake(100)
```

### Rendering Data on Pages

`app/main/views.py`

```python
@main.route('/', methods=['GET', 'POST'])
def index():
  #...
  page = request.args.get('page', 1, type=int)
  pagination = Post.query.order_by(Post.timestamp.desc()).paginate(
      page, per_page=current_app.config['FLASKY_POSTS_PER_PAGE'],
      error_out=False)
  posts = pagination.items
  return render_template('index.html', form=form,
                          posts=posts, pagination=pagination)
```

### Adding a Pagination Widget

`pagination()`函数返回的是一个`Pagination`类的实例，该类有如下属性和方法

| Attribute | Description |
| :-------- | :---------- |
| items | The records in the current page |
| query | The source query that was paginated |
| page | The current page number |
| prev_num | The previous page number |
| next_num | The next page number |
| has_prev | True if there is a previous page |
| has_next | True if there is a next page |
| pages | the total number of pages for the query |
| per_page | the number of items per page |
| total | the total number of items returned by the query |

| Method | Description |
| :----- | :---------- |
| iter_pages(left_edge=2, left_current=2, right_current=5, right_edge=2) | |
| prev() | a pagination object for the previous page |
| next() | a pagination object for the next page |

`app/templates/_macros.html`

```liquid
{% raw %}
{% macro pagination_widget(pagination, endpoint) %}
<ul class="pagination">
  <li {% if not pagination.has_prev %} class="disabled"{% endif %}>
    <a href="{% if pagination.has_prev %}{{ url_for(endpoint,
        page=pagination.page-1, **kwargs) }}{% else %}#{% endif %}">
        &laquo;
    </a>
  </li>
  {% for p in pagination.iter_pages() %}
    {% if p %}
      {% if p == pagination.page %}
        <li class="active">
          <a href="{{ url_for(endpoint, page=p, **kwargs) }}">{{ p }}</a>
        </li>
      {% else %}
        <li>
          <a href="{{ url_for(endpoint, page=p, **kwargs) }}">{{ p }}</a>
        </li>
      {% endif %}
    {% else %}
      <li class="disabled"><a href="#">&hellip;</a></li>
    {% endif %}
  {% endfor %}
  <li {% if not pagination.has_next %} class="disabled" {% endif %}>
    <a href="{% if pagination.has_next %}{{ url_for(endpoint,
        page=pagination.page+1, **kwargs) }}{% else %}#{% endif %}">
        &raquo;
    </a>
  </li>
</ul>
{% endmacro %}
{% endraw %}
```

`app/templates/index.html`

```liquid
{% raw %}
{% extends "base.html" %}
{% import "bootstrap/wtf.html" as wtf %}
{% import "_macros.html" as macros %}
...
{% include '_posts.html' %}
<div class="pagination">
  {{ macros.pagination_widget(pagination, '.index') }}
</div>
{% endraw %}
```

## Rich-Text Posts with Markdown and Flask-PageDown

安装必要的package

```shell
pip install flask-pagedown markdown bleach
```

### Using Flask-PageDown

`Flask-PageDown`拓展定义了`PageDownField`类，它和WTForms中的`TextAreaField`有着相同的接口  
在使用前，需要将其初始化

`app/__init__.py`

```python
from flask.ext.pagedown import PageDown
# ...

pagedown = PageDown()
# ...
def create_app(config_name):
  # ...
  pagedown.init_app(app)
  # ...
```

修改`app/main/forms.py`中的`PostForm()`

```python
from flask_pagedown.fields import PageDownField

class PostForm(Form):
  body = PageDownField("What's on your mind?", validators=[Required()])
  submit = SubmitField('Submit')
```

`app/templates/index.html`中添加脚本

```liquid
{% raw %}
{% block scripts %}
{{ super() }}
{{ pagedown.include_pagedown() }}
{% endblock %}
{% endraw %}
```

### Handling Rich Text on the Server

保存博客表单时，只保存原始markdown数据，所以再次显示时需要重新转换成html

`app/models.py`

```python
from markdown import markdown
import bleach

class Post(db.Model):
  #...
  body_html = db.Column(db.Text)
  #...

  @staticmethod
  def on_changed_body(target, value, oldvalue, initiator):
    allowed_tags = ['a', 'abbr', 'acronym', 'b', 'blockquote', 'code',
                    'em', 'i', 'li', 'ol', 'pre', 'strong', 'ul',
                    'h1', 'h2', 'h3', 'p']
    target.body_html = bleach.linkify(bleach.clean(
                          markdown(value, output_format='html'),
                          tags=allowed_tags, strip=True))

db.event.listen(Post.body, 'set', Post.on_changed_body)
```

更新数据库

```shell
python manage.py db migrate -m "markdown"
python manage.py db upgrade
```

`app/templates/_posts.html`

```liquid
{% raw %}
...
<div class="post-body">
  {% if post.body_html %}
    {{ post.body_html | safe }}
  {% else %}
    {{ post.body }}
  {% endif%}
</div>
...
{% endraw %}
```

## Permanent Links to Blog Posts

为了分享单独一片文件，需要每片博客有独立的固定URL

`app/main/views.py`

```python
@main.route('/post/<int:id>')
def post(id):
  post = Post.query.get_or_404(id)
  return render_template('post.html', posts=[post])
```

注意传给`post.html`页面中的是一个博客列表，虽然其中只有一个元素。采用这种方式是为了在这个页面中使用`_posts.html`

`app/templates/post.html`

```liquid
{% raw %}
{% extends "base.html" %}

{% block title %}Flasky - Post{% endblock %}

{% block page_content %}
{% include '_posts.html' %}
{% endblock %}
{% endraw %}
```

同时在`_posts.html`中添加到单片博客的连接

`app/templates/_posts.html`

```liquid
<ul class="posts">
  {% for post in posts %}
  <li class="post">
    ...
    <div class="post-content">
      <a href="{{ url_for('.post', id=post.id) }}">
        <span class="label label-default">Permalink</span>
      </a>
    </div>
  </li>
  {% endfor %}
</ul>
```

## Blog Post Editor

`app/templates/edit_post.html`

```liquid
{% raw %}
{% extends "base.html" %}
{% import "bootstrap/wtf.html" as wtf %}

{% block title %}{% endblock %}

{% block page_content %}
<div class="page-header">
  <h1>Edit Post</h1>
</div>
<div>
  {{ wtf.quick_form(form) }}
</div>
{% endblock %}

{% block scripts %}
{{ super() }}
{{ pagedown.include_pagedown() }}
{% endblock %}
{% endraw %}
```

`app/main/views.py`

```python
@main.route('/edit/<int:id>', methods=['GET', 'POST'])
@login_required
def edit(id):
  post = Post.query.get_or_404(id)
  if current_user != post.author and \
        not current_user.can(Permission.ADMINISTER):
    abort(403)
  form = PostForm()
  if form.validate_on_submit():
    post.body = form.body.data
    db.session.add(post)
    flash('the post has been updated.')
    return redirect(url_for('post', id=post.id))
  form.body.data = post.body
  return render_template('edit_post.html', form=form)
```

`app/templates/_posts.html`

```liquid
{% raw %}
<ul class="posts">
  {% for post in posts %}
  <li class="post">
    ...
    <div class="post-content">
      ...
      <div class="post-footer">
        ...
        {% if current_user == post.author %}
        <a href="{{ url_for('.edit', id=post.id) }}">
          <span class="label label-primary">Edit</span>
        </a>
        {% elif current_user.is_administrator() %}
        <a href="{{ url_for('.edit', id=post.id) }}">
          <span class="label label-danger">Edit[Admin]</span>
        </a>
        {% endif %}
      </div>
    </div>
  </li>
  {% endfor %}
</ul>
{% endraw %}
```
