---
layout: post
title:  Python 101
date:   2019-05-06 19:30:00 +0800
categories: lang
tags: python
---

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
