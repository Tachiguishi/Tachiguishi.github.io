---
layout: post
title:  Can I Quote You on That
date:   2016-12-22 15:41:00 +0800
categories:
  - Reading
  - Shell Programming in Unix, Linux and OS X by Stephen G. Kochan
---

总共有四种引用符号: `'`, `"`, `\`, `

## The Single Quote

由于Shell使用空白符来分割参数。
如果某个参数里有空白符，则直接传入会被识别为两个参数，从而出错。
这时只需将该参数包含在单引号中即可

```shell
grep 'Thomas Anderson' Matrix
```

不仅是空白符，单引号中所有的特殊符号都不会被Shell转义

## The Double Quote

双引号和单引号功能相似，只是在双引号内以下三个特殊符号仍然会被转义:  
* 美元符号`$`，即可以表示变量的值
* 反斜杠`\`
* back quotes `

```shell
filelist=*
echo $filelist
# addresses intro lotsaspaces names nu numbers phonebook stat
echo '$filelist'
# $filelist
echo "$filelist"
# *
```

## The Backslash

格式

```shell
\c
```

直接输出紧跟在`\`后面的字符`c`，不进行任何转义

### 作为续行符使用

```shell
$ lines=one'
> 'two                # Single quotes tell shell to ignore newline
$ echo "$lines"
one
two
$ lines=one\          # Try it with a \ instead
> two
$ echo "$lines"
onetwo
```

### 在双引号中使用

由于一些符号在双引号中仍然会被转移，可以使用`\c`去除转义

## Command Substitution

命令替换是指命令行中使用命令的输出结果替换命令所在位置。  
有两种实现方法：  
* \`
* `$(...)`

需要知道的是，现在更倾向于使用`$(...)`，但该结构在较老版本的Unix中并不支持。  
但\`支持老版本

### 反引号\`

反引号和上面所述的三种不同的地方在于，
它不是保护其中的内容不被转义，而是相反：使用命令结果替换命令

格式：

```shell
`command`
```

```shell
echo The date and time is: `date`
# The date and time is: Wed Aug 28 14:28:43 EDT 2002
```

### `$(...)`结构

格式：

```shell
$(command)
```

```shell
echo The date and time is: $(date)
# The date and time is: Wed Aug 28 14:28:43 EDT 2002
```

可以因此将一个命令的运行结果存入在一个变量中

```shell
filelist=$(ls)
echo $filelist
# addresses intro lotsaspaces names nu numbers phonebook stat
echo "$filelist"
# addresses
# intro
# lotsaspaces
# names
# nu
# numbers
# phonebook
# stat
```

注意输出`$filelist`时加引号与不加引号的区别

相比于使用反引号，`$()`结构更便利与进行嵌套使用

### expr

一些老版本的Unix并没有内建的整数计算功能，但是可以使用`expr`命令  
支持`+`, `-`, `*`, `/`, `%`

```shell
expr 1 + 2   # 有空格
# 3
expr 1+2     # 没有空格
# 1+2
```

注意，乘法运算符`*`有特殊的含义，直接输入会被转义，可以使用上述引用符忽略转义

```shell
expr "2 * 3"
```
