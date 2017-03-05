---
layout: post
title:  "FlaskScript"
date:   2016-11-16 16:11:47 +0800
categories:
  - Reading
  - Flask Web Development by Miguel Grinberg
---

# Flask Script

Flask 的开发 Web 服务器支持很多启动设置选项，但只能在脚本中作为参数传给`app.run()`函数。
这种方式并不十分方便，传递设置选项的理想方式是使用命令行参数。
Flask-Script 是一个 Flask 扩展，为 Flask 程序添加了一个命令行解析器。Flask-Script 自带
了一组常用选项，而且还支持自定义命令。

## 安装

使用`pip`安装

```
(venv) { flasky } HEAD » pip install flask-script
Collecting flask-script
  Downloading Flask-Script-2.0.5.tar.gz (42kB)
    100% |████████████████████████████████| 51kB 76kB/s
Requirement already satisfied: Flask in ./venv/lib/python2.7/site-packages (from flask-script)
Requirement already satisfied: itsdangerous>=0.21 in ./venv/lib/python2.7/site-packages (from Flask->flask-script)
Requirement already satisfied: click>=2.0 in ./venv/lib/python2.7/site-packages (from Flask->flask-script)
Requirement already satisfied: Jinja2>=2.4 in ./venv/lib/python2.7/site-packages (from Flask->flask-script)
Requirement already satisfied: Werkzeug>=0.7 in ./venv/lib/python2.7/site-packages (from Flask->flask-script)
Requirement already satisfied: MarkupSafe in ./venv/lib/python2.7/site-packages (from Jinja2>=2.4->Flask->flask-script)
Building wheels for collected packages: flask-script
  Running setup.py bdist_wheel for flask-script ... done
  Stored in directory: /home/Spike/.cache/pip/wheels/e2/ea/d8/8d114e46cef819f7d9879504a7f9cb2a88a479af2858223d9f
Successfully built flask-script
Installing collected packages: flask-script
Successfully installed flask-script-2.0.5
```

## 代码修改

```python
from flask import Flask
from flask_script import Manager

app = Flask(__name__)
manager = Manager(app)

@app.route('/')
def index():
    return '<h1>Hello World!</h1>'

@app.route('/user/<name>')
def user(name):
    return '<h1>Hello, %s!</h1>' % name

if __name__ == '__main__':
    manager.run()
```
服务改用`manager.run()`启动，而不是`app.run()`

## 运行

```
(venv) { flasky } HEAD » python hello.py
usage: hello.py [-?] {shell,runserver} ...

positional arguments:
  {shell,runserver}
    shell            Runs a Python shell inside Flask application context.
    runserver        Runs the Flask development server i.e. app.run()

optional arguments:
  -?, --help         show this help message and exit
```

可以看出`runserver`参数用来启动服务

```
(venv) { flasky } HEAD » python hello.py runserver --help
usage: hello.py runserver [-?] [-h HOST] [-p PORT] [--threaded]
                          [--processes PROCESSES] [--passthrough-errors] [-d]
                          [-D] [-r] [-R]

Runs the Flask development server i.e. app.run()

optional arguments:
  -?, --help            show this help message and exit
  -h HOST, --host HOST
  -p PORT, --port PORT
  --threaded
  --processes PROCESSES
  --passthrough-errors
  -d, --debug           enable the Werkzeug debugger (DO NOT use in production
                        code)
  -D, --no-debug        disable the Werkzeug debugger
  -r, --reload          monitor Python files for changes (not 100{'const':
                        True, 'help': 'monitor Python files for changes (not
                        100% safe for production use)', 'option_strings':
                        ['-r', '--reload'], 'dest': 'use_reloader',
                        'required': False, 'nargs': 0, 'choices': None,
                        'default': None, 'prog': 'hello.py runserver',
                        'container': <argparse._ArgumentGroup object at
                        0xffac7dcc>, 'type': None, 'metavar': None}afe for
                        production use)
  -R, --no-reload       do not monitor Python files for changes
```

`--host`参数告诉 Web 服务器在哪个网络接口上监听来自客户端的连接
默认情况下，Flask 开发 Web 服务器监听 localhost 上的连接

默认只可以本地访问
```
(venv) { flasky } HEAD » python hello.py runserver
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
```

其他电脑通过运行服务的电脑的IP访问
```
(venv) { flasky } HEAD » python hello.py runserver --host 0.0.0.0
 * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
```
