---
layout: post
title:  Chapter 4 Tools of The Trade
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

`sed`会对文件中的内容执行`command`所指定的操作，
然后将操作后的整个文件的内容通过标准输出输出  
并不会改变原文件中的内容

```shell
sed command file > temp
mv temp file
```
可以通过这种方式改变原文件

### `-n`

`sed -n`表示不将执行命令后的整个文件的内容输出，结合`p`使用可以指定输出内容  

```shell
# 输出`intro`文件中的前两行
sed -n '1,2p' intro
# 将`intro`文件中所有包含`Unix`的行输出
sed -n '/Unix/p' intro
# 将`intro`文件所有行输出，使用转义字符表示非打印字符 如`tab`用`\t`表示
sed -n l intro
```

### 删除行

在`command`后加`d`则可将符合`command`的行删除，并输出文件中剩余的内容

```shell
sed '1,2d' intro  # 删除 intro 中的第1,2行
$ sed '/Unix/d' intro # 删除 intro 中所有包含 Unix 的行
```

## tr

## 语法

```shell
tr from-chars to-chars
```

`tr`只接受标准输入，将`from-chars`替换成`to-chars`，并标准输出  
只作用于单个字符

```shell
# 将 intro 中内内容大写转小写，小写转大写
tr '[a-zA-Z]' '[A-Za-z]' < intro
```

对于非打印字符，可以使用`\nnn`的方式，其中`nnn`为字符的八进制数值

### `-s`

`s` short for `squeeze`  
如果`from-chars`连续出现，则仅适用一个`to-chars`代替

```shell
# 将 intro 中连续的多个空格替换为一个空格(删除多余空格)
$ tr –s ' ' ' ' < intro
```

### `-d`

`d` short for `delete`

```shell
# 删除 输入参数中的 from-chars 匹配的字符
tr -d from-chars
```

## grep

Global Regular Expression Print

### 语法

```shell
grep pattern files
```

匹配指定文件中符合`pattern`的行，并将匹配的行标准输出  
`grep`可以同时接受多个文件名进行匹配，接受多个文件名时，
输出结果的每行会前缀`filename:`进行区分。  
`pattern`可以使用正则表达式  

可以使用`*`代表当前目录的下的所有文件

```shell
grep Unix *
```

```shell
grep Unix intro
sed -n '/UNIX/p' intro
```
以上两行命令的输出结果是一样的

为了避免歧义，可以在`pattern`模式外使用`''`包围进行保护  

```shell
grep * intro
```

假如当前目录下有文件`intro, pilot, shell`，则执行上述命令实际上是执行

```shell
grep intro pilot shell intro
```

`grep`会在`pilot shell intro`三个文件中匹配`intro`  
这是只要使用`''`就可以了

```shell
grep '*' intro
```

### `-i`

`i` short for `insensitive`  
匹配`pattern`是忽略大小写

### `-v`

`v` short for `reverse`
输出所有不匹配`pattern`的行

### `-l`

只输出含有匹配的文件名，而不输出具体的匹配行

### `-n`

在输出匹配行的时候同时输出行号作为前缀`lineNO:`

## sort

```shell
sort file
```

将文件中的行按升序排列后输出(包括空行)

### `-u`

`u` short for `unique`  
排序时删除重复的行。`sort -u`等效于`sort | uniq`

### `-r`

`r` short for `reverse`  
以降序方式排列

### `-o`

`o` short for `output`  
指定输出文件，将排序后的结果输出到指定文件中  

```shell
sort intro -o sorted-intro
sort intro > sorted-intro
```
上述两条命令是等效的，但是如果想把排序后的文件写入原文件，就只能使用`-o`命令了

```shell
sort intro -o intro  # 正常执行，原文件内容被排序
sort intro > intro   # Error 会清空 intro 中的内容
```

### `-n`

`n` short for `number`  
将文件中每行第一个区域(默认以空格或tab分隔)作为数字进行排序而不是字符串  
`5 < 12`(数字)，`5 > 12`(字符串)

### `-k`

`k` short for `skip`  
默认以空格或tab将文件中的每行分隔为多个区域(从1开始计数)，
`k`后跟一个数字表明从该数字起开始比较计算，而忽略之前的区域

```shell
sort -k2 intro  # 忽略第一个单词
```

### `-t`

前面说了默认以空格或tab分隔，而`-t`后紧跟字符则可以自定义分隔符

```shell
sort -k2 -t: intro  # 以 : 分隔
```

## uniq

```shell
uniq in_file out_file
```
删除`in_file`中的重复行并将结果输出到`out_file`中，
如果没有指明`out_file`则输出到标准输出

需要说明的是，`uniq`定义的重复行是指行号相邻连续的重复行。
如果两行的内容相同，但是行号不连续，则不会被认定为重复行。
所以通常和`sort`一起使用

### `-d`

`d` short for `duplicate`  
`-d`让`uniq`只输出重复的行(只输出一次)

### `-c`

`c` short for `count`  
该参数使`uniq`可以统计相同行出现的次数，并以每行的前缀输出


## Summary

可以使用`man`命令后跟一个命令名来查看一个命令的手册  
`man` short for `manual`
