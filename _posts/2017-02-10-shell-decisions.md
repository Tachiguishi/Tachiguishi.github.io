---
layout: post
title:  Chapter 7 Condition statement
date:   2017-02-10 01:52:00 +0800
categories:
  - Reading
  - Shell
---

条件语句`if`的基本格式

```shell
if command; then
  command
  command
  ...
fi
```

如果`command;`的`exit status`为0，则执行`then`和`fi`之间的代码

## Exit Status

任何程序运行结束后都会向shell返回一个数值(exit status)来表示程序的执行情况。  
一般返回0表示执行成功，其它值则表示不同类型的错误

变量`$?`始终保存shell中最后一次执行的命令的`exit status`

一个检测某用户是否登录的程序

```shell
user="$1"

if who | grep "^$user " > /dev/null
then
  echo "$user is logged on"
fi
```

因为`grep`会将匹配的行输出，所以使用`/dev/null`将输出结果指向系统的`garbage can`。
任何用户都可以读写此文件，但读取不到任何内容，写入的内容也立即消失。
从而时`grep`语句在`shell`中不输出任何内容。  
`echo`语句的锁进不是必须的，只是为了增加可读性

## `test`命令

```shell
test expression
```

`test`首先执行`expression`语句，如果结果为`true`则返回的`exit status`为0，否则为非0值

### 字符串操作

判定两个字符串是否相同

```shell
test "$name" = luna
```

注意两个运算量(`$name`和`luna`)和运算符(`=`)都是作为参数传给`test`命令的，也就是说`=`两边都要加空格

由于变量值替换发生在传参数之前，所以使用双引号是很好的习惯。如果变量`$name`的值为空，无双引号时则会报错

```shell
$ name=
$ test $name = luna
# sh: test: argument expected
```

这是因为传给`test`的只有`=`和`luna`两个参数，所以报错  
而如果使用双引号则不会有问题，传给`test`仍是三个参数`null`, `=`, `luna`

| Operator | Returns TRUE(exit status of 0) if |
| :------- | :-------------------------------- |
| string1 = string2 | string1 is identical to string2 |
| string1 != string2 | string1 is not identical to string2 |
| string | string is not null |
| -n string | string is not null(and string must be seen by `test`)(nonzero length) |
| -z string | string is null(and string must be seen by `test`)(zero length) |

```shell
$ echo $symbol
=
$ test -z "$symbol"
sh: test: argument expected
```

这是因为`=`的优先级高于`-z`。可以使用以下方式解决

```shell
test X"$symbol" = X
```

如果`$symbol`是空则返回`true`，否则返回`false`

### `test`的另一种格式

```shell
test expression
[ expression ]
```

以上两条语句等价。`[`实际上是`test`命令的另一个名字(命令名可以不包含字母)，但是你必须在结尾使用`]`结束。
所以在`expression`和`[`, `]`之间必须加空格

### 整数操作

| Operator | Returns TRUE(exit status of 0) if |
| :------- | :-------------------------------- |
| int1 -eq int2 | int1 is equal to int2 |
| int1 -ge int2 | int1 is greater than or equal to int2 |
| int1 -gt int2 | int1 is greater than int2 |
| int1 -le int2 | int1 is less than or equal to int2 |
| int1 -lt int2 | int1 is less than int2 |
| int1 -ne int2 | int1 is not equal to int2 |

### 文件操作

`shell`有多种关于文件的`test`操作，以下为常用的几种

| Operator | Returns TRUE(exit status of 0) if |
| :------- | :------- |
| -d file | file is a directory |
| -e file | file exits |
| -f file | file is an ordinary file |
| -r file | file is readable by the process |
| -s file | file has nonzero length(not empty file) |
| -w file | file is writable by the process |
| -x file | file is executable |
| -L file | file is a symbolic link |

### 逻辑取反操作符`!`

对任何`test`的运算结果取反

```shell
[ ! -f "$file" ]
```

### 逻辑与操作符`-a`

```shell
[ "$count" -ge 0 -a "$count" - lt 10 ]
```

使用最短路径原则，如果前面的值结果为`false`，则立即返回结果不进行后续计算

### 括号

```shell
[ \( "$count" -ge 0 \) -a \( "$count" - lt 10 \) ]
```

注意，括号左右必须都加空格

### 逻辑或操作符`-o`

优先级比`-a`低

```shell
[ -n "$file" -o -r $HOME/file ]
```

## `else`结构

```shell
if command
then
  command
else
  command
fi
```

```shell
if [ "$#" -ne 1 ]
then
  echo "Incorrect number of argument"
  echo "Usage: on user"
else
  user="$1"
  if who | grep "^$user " > /dev/null
  then
    echo "$user is logged on"
  else
    echo "$user is not logged on"
  fi
fi
```

## `exit`命令

```shell
exit n
```

`exit`可以立即结束结束你的程序，并将`n`最为`exit status`的值返回。如果没有知名`n`的值，则相当于`exit $?`

```shell
if [ "$#" -ne 1 ]
then
  echo "Incorrect number of argument"
  echo "Usage: rem name"
  exit 1
fi

grep -v "$1" phonebook > /temp/phonebook
mv /temp/phonebook phonebook
```

## `elif`结构

```shell
if [ condition ]; then
  command
elif [ condition ]; then
  command
else
  command
fi
```

```shell
if [ "$#" -ne 1 ]
then
  echo "Incorrect number of argument"
  echo "Usage: rem name"
  exit 1
fi

name=$1
matches=$(grep "$name" phonebook | wc -1)

if [ "$matches" -gt 1 ]
then
  echo "More than one match; please qualify further"
elif [ "$matches" -eq 1 ]; then
  grep -v "$name" phonebook > /temp/phonebook
  mv /temp/phonebook phonebook
else
  echo "can not find $name in the phone book"
fi
```

根据`Unix`习惯，程序成功运行不输出任何结构，出错时输出提示信息

## `case`命令

```shell
case value in
pattern1) command
          command
          ;;
pattern2) command
          command;;
esac
```

类似于`c++`中的`switch`，`;;`功能和`break`相同。  
`*`可以匹配任何值，所以可以理解为`switch`中的`default`

```shell
if [ $# -ne 1 ]; then
  echo Usage: ctype char
  exit 1
fi

char=$1
case "$char" in
  [0-9])  echo digit;;
  [a-z])  echo lowercase letter;;
  [A-Z])  echo uppercase letter;;
  ?    )  echo special letter;;
  *    )  echo please type a single character;;
esac
```

`pattern`可以使用`|`运算符进行或运算。`pattern1 | pattern2`即表示只要符合两个模式中的任何一个即可

```shell
hour=$(date +%H)

case "$hour" in
  0? | 1[01] )  echo "good morning";;
  1[2-7]     )  echo "good afternoon";;
  *          )  echo "good evening";;
esac
```

### 使用`-x`进行调试

```shell
sh -x command
```

这种模式下，被执行了的命令都会直接被打印在`shell`中，变量也会被其值所替代显示出来。
所以可以方便地看出哪些命令被执行了，以及各变量的值是多少

## 空命令`:`

语法上需要使用命令的地方实际砂锅却不需要执行任何命令时可以使用空命令`:`来解决

## `&&`和`||`结构

```shell
command1 && command2
```

如果`command1`的`exit status`为0，则`command2`才会被执行，否则不执行

```shell
command1 || command2
```

如果`command1`的`exit status`不为0，则`command2`才会被执行，否则不执行

`&&`和`||`结构都可以使用`if`结构改写
