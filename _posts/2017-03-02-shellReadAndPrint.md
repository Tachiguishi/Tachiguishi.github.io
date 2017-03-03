---
layout: post
title:  Chapter 9 Reading and Printing Data
date:   2017-03-02 22:20:00 +0800
categories:
  - Reading
  - Shell Programming in Unix, Linux and OS X by Stephen G. Kochan
---

## The `read` Command

```shell
read variables
```

`shell`从`standard input`中读取一行，然后将其依次赋值给变量列表。
如果读取的单词个数大于变量个数，则将剩余的单词全部赋值给最后一个变量

### a program to copy file

```shell
if [ "$#" -ne 2]; then
  echo "Usage: mycp from to"
  exit 1
fi

from="$1"
to="$2"

if [ -e "$to" ]; then
  echo "$to already exists; overwrite [yes/no]?"
  read answer

  if [ "$answer" != yes ]; then
    echo "Copy not performed"
    exit 0
  fi
fi

cp $from $to
```

### special echo escape characters

`echo`命令运行后会自动在结尾添加换行符，所以如果想让用户在同一行输入可以添加`\c`符号

```shell
echo "$to already exists; overwrite [yes/no]? \c"
```

注意，`\c`是被`echo`命令进行转义的，而不是`shell`，所以必须添加在引号类。  
其它转义符

| Character | Prints |
| :-------- | :----- |
| \b | backspace |
| \c | the line without a terminating newline |
| \f | formfeed |
| \n | newline |
| \r | carriage return |
| \t | tab character |
| \\ | backslash |
| \0nnn | character whose ASCII value is nnn, where nnn is a one- to three-digit octal number |

### an improved version of mycp

当目标路径只是文件夹时，默认将原文件以相同文件名复制到目标文件夹中

```shell
if [ "$#" -ne 2 ]; then
  echo "Usage: mycp from to"
  exit 1
fi

from="$1"
to="$2"

if [[ -d "$to" ]]; then
  to="$to/$(basename $from)"
fi

if [[ -e "$to" ]]; then
  echo "$to already exists; overwrite [yes/no]? \c"
  read answer

  if [[ "$answer" != yes ]]; then
    echo "Copy not performed"
    exit 0
  fi
fi

cp $from $to
```

`basename`命令可以将包含路径的字符串中提取除单纯的文件名

### a final version of mycp

将多个文件复制到同一目录下

```shell
numargs=$#
filelist=
copylist=

while [[ "$#" -gt 1 ]]; do
  filelist="$filelist $1"
  shift
done

to="$1"

if [[ "$numargs" -lt 2 -o "$numargs" -gt 2 -a ! -d "$to" ]]; then
  echo "Usage: mycp file1 file2"
  echo "       mycp file(s) dir"
  exit 1
fi

for from in $filelist ; do
  if [[ -d "$to" ]]; then
    tofile="$to/$(basename $from)"
  else
    tofile="$to"
  fi

  if [[ -e "$tofile" ]]; then
    echo "$tofile already exists; overwrite [yes/no]? \c"
    read answer

    if [[ "$answer" = yes ]]; then
      copylist="$copylist $from"
    fi
  else
    copylist="$copylist $from"
  fi
done

if [[ -n "$copylist" ]]; then
  cp $copylist $to
fi
```

### a menu-driven phone program

```shell
if [[ "$#" -ne 0 ]]; then
  lu "$@"
  exit
fi

validchoice=""

until [[ -n "$validchoice" ]]; do
  echo '
  would you like to :
    1. look someone up
    2. add someone to the phone book
    3. remove someone from the phone book
  please select one of the above (1-3): \c'

  read choice
  echo

  case "$choice" in
    1) echo "Enter name to look up: \c"
      read name
      lu "$name"
      validchoice=TRUE;;
    2) echo "Enter name to be added: \c"
      read name
      echo "Enter number: \c"
      read number
      add "$name" "$number"
      validchoice=TRUE;;
    3) echo "Enter name to be removed: \c"
      read name
      rem "$name"
      validchoice=TRUE
    *) echo "Bad choice";;
  esac
done
```

### $$

`$$`表示当前登录`shell`的进程ID

### the exit status from `read`

通常`read`的`exit status`都为0，除非遇到了`end-of-file`标识。
如果数据来源于控制台，则表示用户按了`ctrl + d`；如果数据源为文件，则表示到达了文件尾部。  
利用这个特性，可以很方便地按行遍历文件

```shell
lineno=1

cat $* |
while [[ read -r line ]]; do
  echo "$lineno: $line"
  lineno=$((lineno + 1))
done
```

`-r`参数选项表示忽略"\\"的转义功能

## the `printf` command

```shell
printf "format" arg1 arg2 ...
```

功能和用法类似于`c/c++`中的`printf`

| character | Use for Printing |
| :------ | :--------------- |
| %d | integer |
| %u | unsigned integer |
| %o | octal integer |
| %x | hexadecimal integer, using a-f |
| %X | hexadecimal integer, using A-F |
| %c | single character |
| %s | literal string |
| %b | string containing backslash escape character |
| %% | percent sign |

其中`format`的一般形式

```shell
%[flags][width][.precision]type
```

其中`type`就是上表中所列出的字符(`d,u,o,x,X,c,s,b`)

| modifier | meaning |
| :------- | :------ |
| [flags] | |
| - | left justify value |
| + | precede integer with `+` or `-` |
| (space) | precede positive integer with space character |
| # | precede octal integer with `0`, hexadecimal integer with `0x` or `0X` |
| [width] | minimum width of field; `*` means use next argument as width |
| [.precision] | minimum number of digits for integer; maximum number of characters to display for strings; `*` means use next argument as precision |
