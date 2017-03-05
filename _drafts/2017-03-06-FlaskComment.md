---
layout: post
title:  Chapter 13 User Comment
date:   2017-03-06 06:45:47 +0800
categories:
  - Reading
  - Flask Web Development by Miguel Grinberg
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
