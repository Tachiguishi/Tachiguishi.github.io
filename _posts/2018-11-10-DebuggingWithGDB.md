---
layout: post
title:  Debugging with GDB
date:   2018-11-10 19:30:00 +0800
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

### 查看变量值

* `print variablename` 显示变量当前值，可以直接显示结构体，数组等复杂数据结构。  
* `print/x var` 以十六进制输出变量的值，其它格式参数`c`: 单个字符, `s`: 字符串，`f`: 浮点型
* `display variablename` 在每个断点被触发后自动显示变量的值(如果变量在作用域内的话)  
* `info display` 列出所有`display`列表。
* `disable display display_list_number`
* `enable display display_list_number`
* `undisplay display_list_number` 删除
* `command` 使用`command`语法自定义输出变量与格式，示例：  
    ```shell
    >p tmp->val
    >if (tmp->left != 0)
        >p tmp->left->val
        >else
        >printf "%s\n", "none"
        >end
    >if (tmp->right != 0)
        >p tmp->right->val
        >else
        >printf "%s\n", "none"
        >end
    >end
    ```
* `call function()`  
    在`command`语句中，使用使用`call function()`的方式可以调用程序中的函数
* 动态数组  
    对于普通数组`int x[10]`,可以使用`print x`直接将整个数组全部输出，  
    当时对于动态分配的数组`int* y; y = (int*)malloc(10*sizeof(ing));`，
    这种方法却不行,`print y`输出的时`y`的地址，`print *y`输出的时`y[0]`的值。  
    要想输出整个数组，需要使用`*pointer@number_of_elements`的形式，即`print *y*10`。
    同时支持在输出时的类型转化: `print (int [10])*y`
* `ptype classname`  
    输出类的基本结构：`public`的变量与接口
* `info locals` 列出当前调用栈当中的所有局部变量的值

### 修改变量值

`set var = new_value`

### 调用栈(call stack)

* `backtrace` 显示当前调用栈信息，每个栈都有一个序号(从`0`开始)
* `frame n` 调转到指定栈，(`n`为`backtrace`列表中的序号)
* `up`, `down` 向上/下跳转一个栈
* `backtrace n` 显示从栈顶开始的`n`栈信息
* `backtrace -n` 显示从栈底开始的`n`栈信息

### 命令缩写

* `b == break`
* `n == next`
* `s == step`
* `c == continue`
* `fin == finish`
* `u = until`
* `p == print`
* `dis == disable`
* `disp == display`
* `undisp == undisplay`
* `bt == backtrace`

### other

* `-tui`  
    进入`TUI`模式(可以显示源码)，在普通模式下也可以通过按`ctrl+x,ctrl+a`进入`TUI`模式
* startup files  
    `.gdbinit`文件会在`gdb`每次启动时被载入，所以可以在其中放入一些基础设定，避免每次打开时要重新配置断点等信息。  
    在`$HOME`目录和项目目录中都可以有`.gdbinit`文件，其中`$HOME`目录的`.gdbinit`文件会最先载入，然后载入可执行文件，最后载入项目文件中的`.gdbinit`文件。也可以使用`gdb -command=startup_files executable`指定其他启动文件

## core dump

程序在运行中崩溃时通常会生成`core dump`文件，生成文件的目录位置与文件名可以通过`sysctl kernel.core_pattern`查看  
`core_pattern`的具体含义可以查看`man 5 core`或[kernel_doc](https://www.kernel.org/doc/Documentation/sysctl/kernel.txt)

而在`Fedora`中，默认是不会产生`core`文件的，但可以通过`coredumpctl`命令来获取

```shell
Example 1. List all the core dumps of a program named foo
    # coredumpctl list foo

Example 2. Invoke gdb on the last core dump
    # coredumpctl gdb

Example 3. Show information about a process that dumped core, matching by its PID 6654
    # coredumpctl info 6654

Example 4. Extract the last core dump of /usr/bin/bar to a file named bar.coredump
    # coredumpctl -o bar.coredump dump /usr/bin/bar
```

> [参考文档](https://ask.fedoraproject.org/en/question/98776/where-is-core-dump-located/)

## 多线程

* `info threads` 列出当前所有线程，当前线程前有一个`*`标记
* `thread n` 调转到指定线程(`n`为`info threads`命令列出的编号)
* `break args thread n` 在指定线程上设置断点

## 重定向

通常运行程序的输出与GDB在同一窗口，对于某些程序，特别是`ncurse`程序这样很不适合调试。  
这时需要将程序输出重定向到别的窗口。  

1. 先启动一个终端窗口作为运行程序的输出窗口，在该窗口中运行`tty`获取其ID,如`/dev/pts/7`
2. 启动`GDB`，运行`tty /dev/pts/7`将程序输出重定向到先前打开的终端窗口  

## 其他工具

* `strace` 打印出所有系统调用，调用参数，返回结果，错误信息
* `ltrace` 打印出所有库函数调用，调用参数，返回结果，错误信息
* `lint`等静态分析工具
* 内存检测工具， 参见[Memory Debuggers](https://elinux.org/Memory_Debuggers)

## 其它语言

`GDB`还可以用来调试其它语言，如`Java`, `Python`, `Perl`

## 参考资料

* [Official_documentation](https://www.gnu.org/software/gdb/documentation/)
