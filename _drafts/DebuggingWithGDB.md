---
layout: post
title:  Debugging with GDB
date:   2018-10-10 19:30:00 +0800
categories: tools
tags: gdb
---

`GDB(GNU Debugger)`是`Linux`上通用的命令行调试工具，`DDD(Data Display Debugger)`则是`GDB`的`GUI`版本。  
可以使用他们在`Linux`上实现`c/c++`的调试工作

```shell
gdb --version
#=> GNU gdb (GDB) Fedora 8.1.1-3.fc28
ddd --version
#=> GNU DDD 3.3.12 (x86_64-redhat-linux-gnu)
```

## GDB基本命令

### 设置断点

* `break function`
* `break line_number`
* `break filename:line_number`
* `break filename:function`

### 删除断点

* `delete breakpoint-list-number`  
    根据断点编号来删除断点
* `delete`  
    删除所有断点
* `clear`  
    删除即将遇到的最近的一个断点
* `clear function`
* `clear line_number`
* `clear filename:line_number`
* `clear filename:function`

### enable/disable断点

* `disable breakpoint-list-number`
* `disable`  disable所有断点
* `enable breakpoint-list-number`
* `enable`  enable所有断点
* `enable once breakpoint-list-number`

### 继续执行

* 单步执行  
    * `next`: step over
    * `next n`: step over n times
    * `step`: step into
* `continue`  
    继续执行到下一断点或程序结束
* `continue n`  
    继续执行，并忽略即将遇到的`n`个断点
* `finish`: step out
    跳出当前的栈，即执行完当前所在的函数。如果其中遇到断点会暂停
* `until`  
    执行到下一个行号大于当前行号的语句，通常用来跳出循环  
    `until`还可以后接参数，其参数与`break`的参数同等含义  
    `until 17`执行到第17行。`until swap`执行到`swap`函数的入口

### 条件断点

* `break break-args if (condition)`  
    其中的`condition`是一个返回`Boolean`的表达式，括号是可选的。表达时中可以使用你程序中的函数。
* `cond breakpoint-list-number condition`  
    为已经存在的断点添加条件，使其变成条件断点
* `cond breakpoint-list-number`  
    将条件断点变成普通断点

### 断点执行命令

可以为每个断点配置一个命令脚本，每当断点被触发时，相应的命令就会被执行

```shell
commands breakpoint-list-number
...
your commands
...
end
```

通过这种方式可以在触发断点时自动输出相关变量的值

示例

```shell
(gdb) command 1
Type commands for when breakpoint 1 is hit, one per line.
End with a line saying just "end".
>silent
>printf "fibonacci was passed %d.\n", n
>continue
>end
```

`silent`参数使`gdb`在触发断点时不会打印断点本身属性，有利于我们自己的输出内容更连贯。
`continue`使其在断点触发后自动继续运行，避免手动

这就相当于在程序中添加了一行输出变量的语句，但是却没有改变代码

#### 宏定义

```shell
define macro_name
...
your commands
...
end
```

示例

```shell
(gdb) define print_and_go
Redefine command "print_and_go"? (y or n) y
Type commands for definition of "print_and_go".
End with a line saying just "end".
>printf $arg0, $arg1
>continue
>end
```

这样上面的断点命令可以写成

```shell
(gdb) commands 1
Type commands for when breakpoint 1 is hit, one per line.
End with a line saying just "end".
>silent
>print_and_go "fibonacci() was passed %d\n" n
>end
```

可以把这段宏定义写入`.gdbinit`文件

### Watchpoints

```shell
watch expression
```

当监视的表达式的值改变时将自动触发断点。  
被监视表达式中的变量必须在当前作用域中，当超出作用域时自动删除`watchpoint`。  
`watchpoint`不支持多线程，它只能监视变量在单个线程中的变化

### 命令缩写

* `b == break`
* `n == next`
* `s == step`
* `c == continue`
* `fin == finish`
* `u = until`

* `tbreak line`  
    设置临时断点，即该断点在被出发过一次后就会自动失效
* `run`  
    启动程序，后面可接参数
* `print variablename`  
    查看制定变量的当前值
* `watch [variablename | condition]`  
    监视`variablename`值,但其值变化时会自动出发断点。也可以使用条件语句，如`wactch (var > 30)`
* stack frame  
    相关命令: `frame`, `up`, `down`, `backtrace`
* `-tui`  
    进入`TUI`模式(可以显示源码)，在普通模式下也可以通过按`ctrl+x,ctrl+a`进入`TUI`模式
* startup files  
    `.gdbinit`文件会在`gdb`每次启动时被载入，所以可以在其中放入一些基础设定，避免每次打开时要重新配置断点等信息。  
    在`$HOME`目录和项目目录中都可以有`.gdbinit`文件，其中`$HOME`目录的`.gdbinit`文件会最先载入，然后载入可执行文件，最后载入项目文件中的`.gdbinit`文件。也可以使用`gdb -command=startup_files executable`指定其他启动文件
