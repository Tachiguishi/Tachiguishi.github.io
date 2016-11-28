## Jinja2模板引擎

### 模板文件

`index.html`
```html
<h1>Hello World!</h1>
```

`user.html`
```html
<h1>Hello, {{ name }}!</h1>
```

### 渲染

默认情况下，Flask在程序文件夹中的`templates`子文件夹中寻找模板。
所以要把模板文件`index.html`，`user.html`放到改文件夹中

```python
from flask import Flask, render_template
from flask_script import Manager

app = Flask(__name__)

manager = Manager(app)

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/user/<name>')
def user(name):
    return render_template('user.html', name=name)

if __name__ == '__main__':
    manager.run()
```

`render_template`函数的第一个参数是模板的文件名。
随后的参数都是键值对，表示模板中变量对应的真实值。

#### 变量

模板中使用的`{{ name }}`结构表示一个变量，
它是一种特殊的占位符，告诉模板引擎这个位置的值从渲染模板时使用的数据中获取。
`Jinja2` 能识别所有类型的变量，甚至是一些复杂的类型，例如列表、字典和对象  
```html
<p>A value from a dictionary: {{ mydict['key'] }}.</p>
<p>A value from a list: {{ mylist[3] }}.</p>
<p>A value from a list, with a variable index: {{ mylist[myintvar] }}.</p>
<p>A value from an object's method: {{ myobj.somemethod() }}.</p>
```

* **过滤器(Filter)**  
格式
```html
Hello, {{ name|capitalize }}
```

过滤器名 | 说明
------- | ------
safe | 渲染值时不转义
capitalize | 把值的首字母转换成大写，其他字母转换成小写
lower | 把值转换成小写形式
upper | 把值转换成大写形式
title | 把值中每个单词的首字母都转换成大写
trim | 把值的首尾空格去掉
striptags | 渲染之前把值中所有的 HTML 标签都删掉

> 千万别在不可信的值上使用 safe 过滤器，例如用户在表单中输入的文本。  

[Jinja2 文档](http://jinja.pocoo.org/docs/templates/#builtin-filters)

### Control Structures(控制结构)

#### if

```html
{% if user %}
  Hello, {{ user }}!
{% else %}
  Hello, Stranger!
{% endif %}
```

#### for

```html
<ul>
  {% for comment in comments %}
    <li>{{ comment }}</li>
  {% endfor %}
</ul>
```

#### macro

```html
{% macro render_comment(comment) %}
  <li>{{ comment }}</li>
{% endmacro %}

<ul>
  {% for comment in comments %}
    {{ render_comment(comment) }}
  {% endfor %}
</ul>
```

* 可以将宏定义在单独的文件中  
```html
{% import 'macros.html' as macros %}
<ul>
  {% for comment in comments %}
    {{ macros.render_comment(comment) }}
  {% endfor %}
</ul>
```
```html
{% include 'common.html' %}
```

#### 继承

基模板`base.html`
```html
<html>
<head>
  {% block head %}
    <title>{% block title %}{% endblock %} - My Application</title>
  {% endblock %}
</head>
<body>
  {% block body %}
  {% endblock %}
</body>
</html>
```

`block` 标签定义的元素可在衍生模板中修改

衍生模板
```html
{% extends "base.html" %}

{% block title %}Index{% endblock %}

{% block head %}
  {{ super() }}
  <style>
  </style>
{% endblock %}

{% block body %}
  <h1>Hello, World!</h1>
{% endblock %}
```

## Bootstrap

### 安装

```
(venv) { flasky } HEAD » pip install flask-bootstrap
Collecting flask-bootstrap
  Downloading Flask-Bootstrap-3.3.7.0.tar.gz (456kB)
    100% |████████████████████████████████| 460kB 43kB/s
Requirement already satisfied: Flask>=0.8 in ./venv/lib/python2.7/site-packages (from flask-bootstrap)
Collecting dominate (from flask-bootstrap)
  Downloading dominate-2.3.0.tar.gz (76kB)
    100% |████████████████████████████████| 81kB 55kB/s
Collecting visitor (from flask-bootstrap)
  Downloading visitor-0.1.3.tar.gz
Requirement already satisfied: itsdangerous>=0.21 in ./venv/lib/python2.7/site-packages (from Flask>=0.8->flask-bootstrap)
Requirement already satisfied: click>=2.0 in ./venv/lib/python2.7/site-packages (from Flask>=0.8->flask-bootstrap)
Requirement already satisfied: Jinja2>=2.4 in ./venv/lib/python2.7/site-packages (from Flask>=0.8->flask-bootstrap)
Requirement already satisfied: Werkzeug>=0.7 in ./venv/lib/python2.7/site-packages (from Flask>=0.8->flask-bootstrap)
Requirement already satisfied: MarkupSafe in ./venv/lib/python2.7/site-packages (from Jinja2>=2.4->Flask>=0.8->flask-bootstrap)
Building wheels for collected packages: flask-bootstrap, dominate, visitor
  Running setup.py bdist_wheel for flask-bootstrap ... done
  Stored in directory: /home/Spike/.cache/pip/wheels/e3/c7/a9/6727fb8f3c88e633efa6d9d5f1575897bde03558a08fdaa78d
  Running setup.py bdist_wheel for dominate ... done
  Stored in directory: /home/Spike/.cache/pip/wheels/df/b8/4d/24d0a69da8c127f2ff7a9f60602bef6e185ab2c7b7886ef1e4
  Running setup.py bdist_wheel for visitor ... done
  Stored in directory: /home/Spike/.cache/pip/wheels/8e/5e/41/c0f515d39b38d492db6d7c12fc4d027385a76fb01aebc9c2b3
Successfully built flask-bootstrap dominate visitor
Installing collected packages: dominate, visitor, flask-bootstrap
Successfully installed dominate-2.3.0 flask-bootstrap-3.3.7.0 visitor-0.1.3
```

### 重定义模板文件

`templates/base.html`
```html
{% extends "bootstrap/base.html" %}

{% block title %}Flasky{% endblock %}

{% block navbar %}
<div class="navbar navbar-inverse" role="navigation">
    <div class="container">
        <div class="navbar-header">
            <button type="button" class="navbar-toggle" data-toggle="collapse" data-target=".navbar-collapse">
                <span class="sr-only">Toggle navigation</span>
                <span class="icon-bar"></span>
                <span class="icon-bar"></span>
                <span class="icon-bar"></span>
            </button>
            <a class="navbar-brand" href="/">Flasky</a>
        </div>
        <div class="navbar-collapse collapse">
            <ul class="nav navbar-nav">
                <li><a href="/">Home</a></li>
            </ul>
        </div>
    </div>
</div>
{% endblock %}

{% block content %}
<div class="container">
    {% block page_content %}{% endblock %}
</div>
{% endblock %}
```

> `bootstrap/base.html`中还定义了许多可以继承的`block`，直接重载可能会引起一些问题，
> 所以在重载是调用`super()`函数是很好的选择

```html
{% block scripts %}
  {{ super() }}
  <script type="text/javascript" src="my-script.js"></script>
{% endblock %}
```

### 代码

```python
from flask_bootstrap import Bootstrap

app = Flask(__name__)
bootstrap = Bootstrap(app)
```

## 自定义错误页面

```python
@app.errorhandler(404)
def page_not_found(e):
  return render_template('404.html'), 404

@app.errorhandler(500)
def internal_server_error(e):
  return render_template('500.html'), 500
```

`templates/404.html`
```html
{% extends "base.html" %}

{% block title %}Flasky - Page Not Found{% endblock %}

{% block page_content %}
<div class="page-header">
    <h1>Not Found</h1>
</div>
{% endblock %}
```

## 连接

可以使用`url_for()`生成连接，如：  
```python
url_for('index')    # /
url_for('index', _external=True)   #  http://localhost:5000/
url_for('user', name='john', _external=True)  # http://localhost:5000/user/john
url_for('index', page=2)    #  /?page=2
```

## 静态文件

静态文件的引用被当成一个特殊的`route`，即`/static/<filename>`
```python
url_for('static', filename='css/styles.css', _external=True)
# 结果http://localhost:5000/static/css/styles.css
```
默认设置下，`Flask` 在程序根目录中名为 `static` 的子目录中寻找静态文件  

可以为`base.html`添加`favicon`
```python
{% block head %}
{{ super() }}
<link rel="shortcut icon" href="{{ url_for('static', filename='favicon.ico') }}" type="image/x-icon">
<link rel="icon" href="{{ url_for('static', filename='favicon.ico') }}" type="image/x-icon">
{% endblock %}
```

## `Flask-Monent`处理时间

一般通过在前端通过浏览器处理时间的本地化，可以利用[`moment.js`](http://momentjs.com)  
Flask通过`Flask-Monent`整合了`moment.js`

### 安装

```
(venv) { flasky } HEAD » pip install flask-moment        /cygdrive/d/GitHub/flasky
Collecting flask-moment
  Downloading Flask-Moment-0.5.1.tar.gz
Requirement already satisfied: Flask in ./venv/lib/python2.7/site-packages (from flask-moment)
Requirement already satisfied: itsdangerous>=0.21 in ./venv/lib/python2.7/site-packages (from Flask->flask-moment)
Requirement already satisfied: click>=2.0 in ./venv/lib/python2.7/site-packages (from Flask->flask-moment)
Requirement already satisfied: Jinja2>=2.4 in ./venv/lib/python2.7/site-packages (from Flask->flask-moment)
Requirement already satisfied: Werkzeug>=0.7 in ./venv/lib/python2.7/site-packages (from Flask->flask-moment)
Requirement already satisfied: MarkupSafe in ./venv/lib/python2.7/site-packages (from Jinja2>=2.4->Flask->flask-moment)
Building wheels for collected packages: flask-moment
  Running setup.py bdist_wheel for flask-moment ... done
  Stored in directory: /home/Spike/.cache/pip/wheels/24/f7/ad/3b7aeb286c374c414dcc350b892cf8998e3294c88472526cb3
Successfully built flask-moment
Installing collected packages: flask-moment
Successfully installed flask-moment-0.5.1
```

### base模板中引入`moment.js`

在模板中添加
`templates/base.html`
```html
{% block scripts %}
{{ super() }}
{{ moment.include_moment() }}
{% endblock %}
```

### 代码

```python
from flask_moment import Moment

app = Flask(__name__)

moment = Moment(app)

@app.route('/')
def index():
    return render_template('index.html',
                           current_time=datetime.utcnow())
```

### index模板中使用

```html
<p>The local date and time is {{ moment(current_time).format('LLL') }}.</p>
<p>That was {{ moment(current_time).fromNow(refresh=True) }}.</p>
```

### 补充

* Flask-Moment 实现了 `moment.js` 中的 `format()` `fromNow()` `fromTime()` `calendar()` `valueOf()`
和 `unix()` 方法。你可[查阅文档](http://momentjs.com/docs/#/displaying/)学习 `moment.js` 的全部格式化结构
* 可以通过`lang()`函数指定语言
```html
{% block scripts %}
{{ super() }}
{{ moment.include_moment() }}
{{ moment.lang('cv') }}
{% endblock %}
```
