---
layout: post
title:  Chapter 13 User Comment
date:   2017-03-06 06:45:47 +0800
categories:
  - Reading
  - Flask
---

## Database representation of comment

用户评论和博客与用户表都是一对多的关系

`app/models.py`

```python
class Comment(db.Model):
 __tablename__ = 'comments'
 id = db.Column(db.Integer, primary_key=True)
  body = db.Column(db.Text)
  body_html = db.Column(db.Text)
  timestamp = db.Column(db.DateTime, index=True, default=datetime.utcnow)
  disabled = db.Column(db.Boolean)
  author_id = db.Column(db.Integer, db.ForeignKey('users.id'))
  post_id = db.Column(db.Integer, db.ForeignKey('posts.id'))

  @staticmethod
  def on_changed_body(target, value, oldValue, initiator):
    allowed_tags = ['a', 'abbr', 'acronym', 'b', 'code', 'em', 'i', 'strong']
    target.body_html = bleach.linkify(bleach.clean(
        markdown(value, output_format='html'),
        tags=allowed_tags, strip=True))

db.event.listen(Comment.body, 'set', Comment.on_changed_body)

class User(db.Model):
  # ...
  comments = db.relationship('Comment', backref='author', lazy='dynamic')

class Post(db.Model):
  # ...
  comments = db.relationship('Comment', backref='post', lazy='dynamic')
```

更新数据库

```shell
python manage.py db migrate -m "comments"
python manage.py db upgrade
```

## Comment submission and display

评论在博客正文页面显示，并且还有一个提交评论的表单

`app/main/forms.py`评论表单

```python
class CommentForm(Form):
  body = StringField('', validators=[Required()])
  submit = SubmitField('Submit')
```

`app/main/views.py`

```python
@main.route('/post/<int:id>', methods=['GET', 'POST'])
def post(id):
  post = Post.query.get_or_404(id)
  form = CommentForm()
  if form.validate_on_submit():
    comment = Comment(body=form.body.data,
                      post=post,
                      author=current_user._get_current_object())
    db.session.add(comment)
    flash('Your comment has been published')
    return redirect(url_for('.post', id=post.id, page=-1))
  page = request.args.get('page', 1, type=int)
  if page == -1:
    page = (post.comments.count() - 1) / \
            current_app.config['FLASK_COMMENTS_PER_PAGE'] + 1
  pagination = post.comments.order_by(Comment.timestamp.asc()).paginate(
    page, per_page=current_app.config['FLASK_COMMENTS_PER_PAGE'],
    error_out=False)
  comments = pagination.items
  return render_template('post.html', posts=[post], form=form,
                          comments=comments, pagination=pagination)
```

添加`app/templates/_comments.html`模版，和`_posts.html`类似

```liquid
{% raw %}
<ul class="comments">
  {% for comment in comments %}
  <li class="comment">
    <div class="comment-thumbnail">
      <a href="{{ url_for('.user', username=comment.author.username) }}">
        <img class="img-round profile-thumbnail"
             src="{{ comment.author.gravatar(size=40) }}">
      </a>
    </div>
    <div class="comment-content">
      <div class="comment-date">
        {{ moment(comment.timestamp).fromNow() }}
      </div>
      <div class="comment-author">
        <a href="{{ url_for('.user', username=comment.author.username) }}">
          {{ comment.author.username }}
        </a>
      </div>
      <div class="comment-body">
        {% if comment.body_html %}
          {{ comment.body_html | safe }}
        {% else %}
          {{ comment.body }}
        {% endif %}
      </div>
    </div>
  </li>
  {% endfor %}
</ul>
{% endraw %}
```

在`app/templates/post.html`中添加评论

```liquid
{% raw %}
<h4 id="comments">Comments</h4>
{% if current_user.can(Permission.COMMENT) %}
<div class="comment-form">
  {{ wtf.quick_form(form) }}
</div>
{% endif %}
{% include '_comments.html' %}
{% if pagination %}
<div class="pagination">
{{ macros.pagination_widget(pagination, '.post',
    fragment='#comments', id=posts[0].id) }}
</div>
{% endif %}
{% endraw %}
```

## Comment moderation

对有`Permission.MODERATE_COMMENTS`权限的用户提供修改评论的功能，
在`comments.html`页面上列出所有的评论过，并在导航条上添加到该页面的连接

`app/templates/base.html`

```liquid
{% raw %}
...
{% if current_user.can(Permission.MODERATE_COMMENTS) %}
<li><a href="{{ url_for('main.moderate') }}">Moderate Comments</a></li>
{% endif %}
...
{% endraw %}
```

`app/main/view.py`

```python
@main.route('/moderate')
@login_required
@permission_required(Permission.MODERATE_COMMENTS)
def moderate():
  page = request.args.get('page', 1, type=int)
  pagination = Comment.query.order_by(Comment.timestamp.desc()).paginate(
    page, per_page=current_app.config['FLASK_COMMENTS_PER_PAGE'],
    error_out=False)
  comments = pagination.items
  return render_template('moderate.html', comments=comments,
                        pagination=pagination, page=page)
```

`app/templates/moderate.html`

```liquid
{% raw %}
{% extends "base.html" %}
{% import "_macros.html" as macros %}

{% block title %}Flasky - Commnet Moderation{% endblock %}

{% block page_content %}
<div class="page-header">
  <h1>Comment Moderation</h1>
</div>
{% set moderate = True %}
{% include '_comments.html' %}
{% if pagination %}
<div class="pagination">
  {{ macros.pagination_widget(pagination, '.moderate') }}
</div>
{% endif %}
{% endblock %}
{% endraw %}
```

`app/templates/_comments.html`

```liquid
{% raw %}
...
<div class="comment-body">
{% if comment.disabled %}
<p><i>This comment has been disabled by a moderator.</i></p>
{% endif %}
{% if moderate or not comment.disabled %}
  {% if comment.body_html %}
    {{ comment.body_html | safe }}
  {% else %}
    {{ comment.body }}
  {% endif %}
{% endif %}
</div>
{% if moderate %}
  <br>
  {% if comment.disabled %}
  <a class="btn btn-default btn-xs"
      href="{{ url_for('.moderate_enable', id=comment.id, page=page) }}">
      Enable</a>
  {% else %}
  <a class="btn btn-danger btn-xs"
      href="{{ url_for('.moderate_disable', id=comment.id, page=page) }}">
      Disable</a>
  {% endif %}
{% endif %}
...
{% endraw %}
```

`app/main/views.py`

```python
@main.route('/moderate/enable/<int:id>')
@login_required
@permission_required(Permission.MODERATE_COMMENTS)
def moderate_enable(id):
  comment = Comment.query.get_or_404(id)
  comment.disabled = False
  db.session.add(comment)
  return redirect(url_for('.moderate',
                          page=request.args.get('page', 1, type=int)))

@main.route('/moderate/disable/<int:id>')
@login_required
@permission_required(Permission.MODERATE_COMMENTS)
def moderate_disable(id):
  comment = Comment.query.get_or_404(id)
  comment.disabled = True
  db.session.add(comment)
  return redirect(url_for('.moderate',
                          page=request.args.get('page', 1, type=int)))
```
