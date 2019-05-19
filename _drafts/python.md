---
layout: post
title:  Python 101
date:   2019-05-06 19:30:00 +0800
categories: lang
tags: python
---

## Tutorial

* [google_tutorial](https://developers.google.com/edu/python/introduction)

## Basics

### 基本计算

`+`, `-`, `*`, `/`, `//`(整除), `%`(取余)  
`x % y`等价于`x - ((x // y) * y)`  

> `//`和`%`都可以应用与浮点数

`**`(指数)

```python
-3 ** 2
# >> -9
(-3) ** 2
# >> 9
```

### 变量

### import

```python
import cmath
from math import sqrt
from turtl import *
from math import power as mypower
```

十六进制(0x), 八进制(0), 二进制(0b)

### script

```python
#!/usr/bin/env python3
```

### 字符串

`'`与`"`无差别，`\`转义字符

拼接: 字面型字符串可以直接自动拼接，变量可以使用`+`拼接

转换: `str`与`repr`

长字符串： 使用`'''`包裹

raw string: 不进行任何转移(末尾的`\`除外)，前缀`r`: `r'Lte\'s party'`

Unicode: 前缀`\u`或`\U`,或使用`\N{name}`格式  
[unicode-table.com](https://unicode-table.com/)

### 编码

设定文件编码

```python
# -*- coding: encoding name -*-
```

## Sequence

### List

使用`,`分隔，被`[]`包裹。可以嵌套

```python
edward = ['Edward Gumby', 42]
```

#### Index

通过下标(index)获取元素，从`0`开始; 或者从`-1`开始反向访问

```python
edward[0]
```

#### Slicing

通过指定两个下标获取一段元素，`[m:n]`包含`m`但不包含`n`

```python
tag = '<a href="http://www.python.org">Python web site</a>'
tag[9:30]
# 'http://www.python.org'
tag[32:-4]
# 'Python web site'
```

可以省略`m`或`n`表示第一个或最后一个元素: `[m:]`, `[:n]`, `[:]`

使用第三个参数指定步长: `[m:n:step]`, `step`可以时负数，表示反向

#### Add

使用`+`可以拼接

```python
[1, 2, 3] + [4, 5, 6]
# [1, 2, 3, 4, 5, 6]
```

#### Multiplication

`Sequence`与数字`n`相承会将`Sequence`重复`n`次

#### 成员

`in`判断某个元素是否包含在`sequence`内，返回`True`或`False`

```python
users = ['mlh', 'foo', 'bar']
'foo' in users
# True
```

#### len, min, max

#### funcion

* 修改 `names[1] = 'Alice'`
* 删除 `del names[2]`
* Assigning to Slices 可以通过此方法添加删除元素
* append 在末尾添加元素
* clear 清空列表
* copy 深复制， 直接使用赋值语句`=`是浅复制
* count 计算某个元素出现的次数
* extend 在末尾添加一个列表
* index 返回某个元素第一次出现的下标，如果不存在则报错
* insert 在指定下标位置插入元素 `names.insert(2, 'ppt')`
* pop 删除最后一个元素并将其返回
* remove 删除指定元素(一次只能删除一个), 无返回值。如果不存在则报错
* reverse 取反
* sort 排序， 特殊排序`x.sort(key=len)` `x.sort(reverse=True)`

### Tuple

使用`,`分隔，被`[]`包裹。可以嵌套

## Map

### Dictionary

## Control

### Boolean

`False`, `None`, `0`, `""`, `()`, `[]`, `{}`

## Function

### document

使用`'`,`'''`在函数第一行写帮助文档，当使用`help()`或`funName.__doc__`可以查看

### 参数

1. 使用`key=value`格式可以忽视参数顺序
2. 参数中`*param`可以以`tuple`格式接收任意数量的参数
3. 参数中的`**param`可以以`dict`格式接收任意数量的`key=value`格式参数
4. 传参时使用`*`,`**`可以将`tuple`, `dict`格式的参数解构传递给普通参数列表的函数

## Class

## 开发环境

[reference](https://vimiix.com/post/2018/03/11/manage-your-virtualenv-with-pipenv/)

```shell
pip3 install --user pipenv
mkdir project_dir
cd project_dir
pipenv install --three
```
