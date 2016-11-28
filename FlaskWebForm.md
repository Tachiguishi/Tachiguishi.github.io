# Flask Web Form

## Cross-Site Request Forgery (CSRF) Protection

为了实现 CSRF 保护，Flask-WTF 需要程序设置一个密钥
```python
app = Flask(__name__)
app.config['SECRET_KEY'] = 'hard to guess string'
```

## Web Form Class

使用 Flask-WTF 时，每个 Web 表单都由`Form`类的一个子类表示  
定义一个表单类
```python
from flask.ext.wtf import Form
from wtforms import StringField, SubmitField
from wtforms.validators import Required

class NameForm(Form):
  name = StringField('What is your name?', validators=[Required()])
  submit = SubmitField('Submit')
```

* WTForms支持的HTML标准字段  
字段类型 | 说　　明
------- | --------
StringField | 文本字段
TextAreaField | 多行文本字段
PasswordField | 密码文本字段
HiddenField | 隐藏文本字段
DateField | 文本字段，值为 datetime.date 格式
DateTimeField | 文本字段，值为 datetime.datetime 格式
IntegerField | 文本字段，值为整数
DecimalField | 文本字段，值为 decimal.Decimal
FloatField | 文本字段，值为浮点数
BooleanField | 复选框，值为 True 和 False
RadioField | 一组单选框
SelectField | 下拉列表
SelectMultipleField | 下拉列表，可选择多个值
FileField | 文件上传字段
SubmitField | 表单提交按钮
FormField | 把表单作为字段嵌入另一个表单
FieldList | 一组指定类型的字段

* WTForms验证函数  
验证函数 | 说　　明
------- | --------
Email | 验证电子邮件地址
EqualTo | 比较两个字段的值；常用于要求输入两次密码进行确认的情况
IPAddress | 验证 IPv4 网络地址
Length | 验证输入字符串的长度
NumberRange | 验证输入的值在数字范围内
Optional | 无输入值时跳过其他验证函数
Required | 确保字段中有数据
Regexp | 使用正则表达式验证输入值
URL | 验证 URL
AnyOf | 确保输入值在可选值列表中
NoneOf | 确保输入值不在可选值列表中

## 模板渲染

```python
<form method="POST">
  {{ form.hidden_tag() }}
  {{ form.name.label }} {{ form.name(id='my-text-field') }}
  {{ form.submit() }}
</form>
```

使用`Flask-Bootstrap`中的表单
```python
{% import "bootstrap/wtf.html" as wtf %}
{{ wtf.quick_form(form) }}
```

## Form Handling in View Functions

```python
@app.route('/', methods=['GET', 'POST'])
def index():
  name = None
  form = NameForm()
  if form.validate_on_submit():
    name = form.name.data
    form.name.data = ''
  return render_template('index.html', form=form, name=name)
```

## Redirects and User Sessions

```python
from flask import Flask, render_template, session, redirect, url_for

@app.route('/', methods=['GET', 'POST'])
def index():
  form = NameForm()
  if form.validate_on_submit():
    session['name'] = form.name.data
    return redirect(url_for('index'))
  return render_template('index.html', form=form, name=session.get('name'))
```

## Message Flashing

```python
from flask import Flask, render_template, session, redirect, url_for, flash

@app.route('/', methods=['GET', 'POST'])
def index():
    form = NameForm()
    if form.validate_on_submit():
        old_name = session.get('name')
        if old_name is not None and old_name != form.name.data:
            flash('Looks like you have changed your name!')
        session['name'] = form.name.data
        return redirect(url_for('index'))
    return render_template('index.html', form=form, name=session.get('name'))
```

`templates/base.html`
```python
{% block content %}
<div class="container">
    {% for message in get_flashed_messages() %}
    <div class="alert alert-warning">
        <button type="button" class="close" data-dismiss="alert">&times;</button>
        {{ message }}
    </div>
    {% endfor %}

    {% block page_content %}{% endblock %}
</div>
{% endblock %}
```
