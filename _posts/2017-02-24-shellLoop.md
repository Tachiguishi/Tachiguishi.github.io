---
layout: post
title:  Chapter 8 Loop
date:   2017-02-24 21:52:00 +0800
categories:
  - Reading
  - Shell Programming in Unix, Linux and OS X by Stephen G. Kochan
---

有三种内置循环语句命令`for`, `while`, `until`

## for

```shell
for var in word1 word2 ... wordn
do
  command
  command
  ...
done
```

`for`循环中，`word[1-n]`泪飙会依次赋值给变量`var`，每次赋值后执行一次`do`与`done`之间的语句。  
示例：

```shell
for i in 1 2 3
do
  echo $1
done
```

输出结果

```shell
1
2
3
```

`for`中的参数列表支持文件名替代, 所以可以有如下语句

```shell
for file in memo[1-4]   # 等价于 for file in memo1 memo2 memo3 memo4
do
done
for file in *
do
done
for file in $(cat filelist) # 文件filelist中保存有一些文件名
do
done
```

在命令文件中接受参数时，`$*`代表所有参数，所以可以使用`for var in $*`遍历所有参数

### `$@`

现有如下程序`args`

```shell
for arg in $*
do
  echo $arg
done
```

则在命令行运行`args 'a b' c`得到的结果为：

```shell
a
b
c
```

这是因为在`$*`替换为参数时`'a b'`被拆分成了`a b`两个参数。这时如果使用`"$@"`代替`$*`即可。
如果不加`""`，则`$@`和`$*`没有区别  
`args`

```shell
for arg in "$@"
do
  echo $arg
done
```

### `for` without the list

```shell
for var
do
  command
done
```

上面的语句等效于

```shell
for var in "$@"
do
  command
done
```

## while

```shell
while [[ condition ]]; do
  #statements
done
```

首先判定`condition`的`exit status`，如果为`0(true)`则执行一次`do`和`done`之间的语句。  
执行完之后再次判定，直到`condition`判定为`false`为止  
示例

```shell
while [[ "$#" -ne 0 ]]; do
  echo "$1"
  shift
done
```

## until

```shell
until [[ condition ]]; do
  #statements
done
```

与`while`不同的是，只有当`condition`返回非0值(false)时才会执行循环体  
和`while`一用，循环体中的内容也可能一次都不会被执行

```shell
sleep n  # 使程序暂停n秒
```

## more on loops

### breaking out of a loop

```shell
break
```

示例

```shell
while true; do
  cmd=$(getcmd)

  if [ "$cmd" = quit]; then
    break
  else
    processcmd "$cmd"
  fi
done
```

如果循环里嵌套着循环，则可以使用`break n`语句同时跳除`n`层循环

### skipping the remaining commands in a loop

```shell
continue
continue n
```

### executing a loop in the background

在`done`后面添加`&`即可

### I/O redirection on a loop

在`done`后面使用`>`运算符可以将循环中的所有结果都导入文件中  
如果循环体中某跳语句也使用了`>`进行输出转换，则其自身的转换会覆盖循环的转换

* `> /dev/tty`可以将结果输出到终端显示
* `> /dev/null`不输出结果
* `2> file`将`standard error`输出到文件中
* `1>&2`将结果输出到终端，即使它被导向其它文件或通道

### piping data into and out of a loop

将循环当成一个整体然后使用`|`

### typing a loop on one line

```shell
for i in 1 2 3; do echo $i; done
```

注意，在`do`后面时没有`;`的  
同样也可以对`if`使用`;`将其写成一行，注意`then`和`else`后面不加`;`

## getopts

```shell
getopts options variable
```

该命令常被用与检测在命令行输入的参数。
`getopts ab:c variable`则表示只接受`-a`, `-b`和`-c`三个配置参数，且`-b`后需要有一个参数表示其值

```shell
while [ getopts mt: option ]; do
  case "$option" in
    m) command;;
    t) var=$OPTARG;;
    \?) echo "Usage: waitfor [-m] [-t n] user"
        exit 1;;
  esac
done
```

命令行中的输入第一个参数首先进行`getopts`运算，如果参数以`-`开头且后跟的字母在`options`中(这里即为`mt`)，
则将后跟的字母保存在变量`option`中(变量名随意)，并将`exit status`以`0`返回。下一次循环时则判定第二个参数  
如果`-`后跟的字母不在`options`之中，则将`?`保存在变量`option`中，并将`exit status`以`0`返回  
如果参数不是以`-`开头，则`getopts`直接返回一个非0的`exit status`  
由于`mt:`中`t`后跟了一个`:`，所以如果`getopts`检测到参数为`-t`则其会继续检测该参数后面一个参数，
如果没有或者以`-`开头，则将`?`赋值给变量`option`，并想`standard error`中输出一条错误；否则将后面跟着的参数
赋值给变量`OPTARG`  

还有一个特殊变量`OPTIND`，其初始值为`1`，每次迭代完则自动加1。
所以`$OPTIND`的值比以`-`开头的参数个数大1(不包括赋值给`OPTARG`的参数)。
可以以此来判定除以`-`开头的参数外还有没有其它参数
