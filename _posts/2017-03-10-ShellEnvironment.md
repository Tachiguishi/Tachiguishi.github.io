---
layout: post
title:  Chapter 10 Environment
date:   2017-03-10 21:18:00 +0800
categories:
  - Reading
  - Shell Programming in Unix, Linux and OS X by Stephen G. Kochan
---

## Local variable

### subshell

当你在`login shell`中执行一段程序文件时，会启动一个新的`shell`来执行，即`subshell`。
`subshell`有自己独立的环境，所以无法获取在`parent shell`中定义的局部变量，自然也无法改变其值

## exported variable

通过使用`export`命令，可以使`subshell`获取`parent shell`中的变量。
当变量被`export`后则`subshell`可以获取该变量的一个副本，在`subshell`中所做的任何更改只存在其自己的环境中，
不会改变`parent shell`中相应变量的值  
被`export`的变量在`subshell`中会自动被`export`，从而传递给更下一层的`subshell`

### export -p

`export -p`命令可以显示当前环境中所有被`export`的变量及他们的值

## PS1 and PS2

变量`PS1`中保存了当前环境中表示`command prompt`符号的值。你可以任意修改其值，但是修改只在当前环境有效  
同样变量`PS2`中保存了`secondary command prompt`。

## HOME

你登录时所在的目录路径会在启动`login shell`时自动被保存在`HOME`变量中。
同样你可以任意修改该变量的值，但是修改只在当前环境有效

## PATH

当你执行一个程序时，`shell`会在`PATH`中保存的所有路径中按顺序依次寻在改程序，如果寻找到则运行程序

## your current directory

当前目录也属于`shell`环境，所以在`subshell`中进行的修改并不会影响到`parent shell`。其值一直保存在变量`PWD`中

### CDPATH

`CDPATH`和`PATH`类似，只不过它时在执行`cd`命令时所查找的目录。  
不同的是`CDPATH`并不会在你启动`shell`是被自动赋值，通常需要用户手动去赋值

## more on subshell

### `.` 命令

```shell
. file
```

`.`会使其后所执行的命令使在当前`shell`中进行的，而不会启动一个独立的`subshell`。

在`shell`中启动一个新`shell`可以建立一个独立的`shell`环境，在退出返回时新`shell`中的各种局部变量都会自动消失  
而且不像`subshell`缺乏交互环境。使用`ctrl + d`可以退出新`shell`

示例：  

```shell
HOME=/usr2/data
BIN=$HOME/bin
RPTS=$HOME/rpts
DATA=$HOME/rawdata

PATH=$PATH$BIN
CDPATH=:$HOME:$RPTS

PS1="DB: "

export HOME BIN RPTS DATA PATH CDPATH PS1

# start up a new shell
/bin/sh
```

### `exec` 命令

```shell
exec program
```

TBD

### `(...)`与`{ ...; }`

`()`和`{}`可以将其中的命令作为一组命令执行。不同的是`()`中的命令是在`subshell`中执行的，
`{}`中的命令是在当前`shell`中执行的

### 向`subshell`中传递变量的其它方式

在运行程序的同一行定义的变量，会自动被`export`

```shell
x=100 foo
(x=100; export x; foo)
```

以上两条语句等价

## `.profile`文件

在启动`shell`时系统会读取两个特殊文件，`/etc/profile`和用户主目录下的`.profile`  
你可以修改`.profile`，在其中定义一些你自己独特的环境参数


## `TERM`

`TERM`变量表示你所使用的`terminal`的配置，一些命令(`vi`等)需要用到

## `TZ`

`TZ`表示Time Zone
