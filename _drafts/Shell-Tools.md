---
layout: post
title:  "Tools of The Trade"
date:   2016-12-08 05:32:00 +0800
categories:
  - Reading
  - Shell Programming in Unix, Linux and OS X by Stephen G. Kochan
---

## 正则表达式

`s/old/new/g` 替换命令

## cut

* `cut -cchars file`  
显示指定文件中每一行的指定位置的字符(从1开始计数)  

```shell
cut -c1 file          # 显示每行的第一个字符
cut -c1-5 file        # 显示每行的前5个字符
cut -c1-5,7-11 file   # 显示每行的前5个字符和第7-11个字符，注意逗号之后不能有空格
cut -c11- file        # 显示从第11个到行尾的字符
```

如果文件名省略，`cut`可以从标准输入内容

```shell
cat file | cut -c1-4
```

* `cut -ddchar –ffields file`  
`dchar`指定每行的分隔符，将每行分割成多个区域(fields)。默认以`tab`作为分隔符  
`fields`指定显示的区域(从1开始计数)

假设有文件`password`

```
root:*:O:O:The Super User:/:/usr/bin/ksh
cron:*:1:1:Cron Daemon for periodic tasks:/:
bin:*:3:3:The owner of system files:/:
uucp:*:5:5::/usr/spool/uucp:/usr/lib/uucp/uucico
asg:*:6:6:The Owner of Assignable Devices:/:
steve:*.:203:100::/users/steve:/usr/bin/ksh
other:*:4:4:Needed by secure program:/:
```

```shell
cut -d: -f1 password
cut -d: -f1,6 password
```

```shell
cut -f1 file  # 以tab作为分割符的第一个区域
```

## paste

* `paste files`  
将一个文件的一行的内容添加到另一个文件中一行的结尾，并在标准输出中输出  
不同行之间默认以`tab`分隔

```shell
paste file1 file2
paste file1 file2 file3
```

* `-d`  
指定不同行连接时的分割符

```shell
paste -d'+' file1 file2
```

* `-s`  
表明是将同一文件的不同行的内容合并为一行，可以和`-d`联合使用

```shell
paste -s file
ls | paste -d' ' -s  # 效果等同于 echo *
```

## sed

Stream Editor. 用于编辑在`pipe`和`command`中的数据

### 语法

```shell
sed command file
```
