---
layout: post
title:  "Linux Basic Command"
date:   2016-11-26 16:11:47 +0800
categories: linux
---

### 101

* `date`
* `who`
* `who am i`
* `echo`

### File

* `ls`
* `cat` 后跟文件名(带后缀名)。输出文件中的内容
* `wc` 后跟文件名(带后缀名)。输出文件的行数/字数/字符数/文件名
* `wc -l filename` 只输出行数
* `wc -w filename` 只输出字数
* `wc -c filename` 只输出字符数
* `cp name name_copy` 复制文件
* `mv name new_name` 文件重命名
* `rm filename` 删除文件

### Directory

* `pwd` Print Working Directory
* `ls pathname`
* `ls -l` 详细信息
* `mkdir` 创建文件夹
* `ln` link files
* `rmdir` 删除文件夹(只能删除空文件夹)
* `rm -r directory` 删除文件夹`directory`及其包含的所有文件

### Filename subsitution

* `*` 匹配任意长度任意字符
* `?` 匹配单个字符
* `[`和`]` `[a-f]`, `[adfji]`
* `!` `[!a-f]`

### Standard I/O

* `Ctrl + d` 结束输入
* `> file` 将原本输出在`terminal`的结果输出到`file`中。
  如果该文件已存在，则其中的内容会被覆盖
* `>> file` 将输出的内容添加到文件末尾，文件原始内容保存
* `< file` 使用文件中的内容作为输入

### Pipes

* `|` 将一个命令的输出作为另一个命令的输入，如`ls | wc -l`

### Standard Error

* standard error 和 standard output 在 terminal 中显示时毫无区别，
但是却不能通过`>`将输出内容导入文件
* `2> file` 将 standard error 导入文件

### Other

* 同一行输入多个命令。不同命令之间使用`;`分开
* 将命令转入后台运行：在命令结尾添加`&`。
* `ps` processor status。显示正在运行的进程状态

### Summary

Command | Description
------- | -----------
cat file(s) | Display contents of  file(s) or standard input if not supplied
cd dir | Change working directory to  dir
cp file l file 2 | Copy  file 1 to  file 2
cp file(s) | dir Copy  file(s) into  dir
date | Display the date and time
echo args | Display  args
ln file l file 2 | Link  file 1 to  file 2
ln file(s) dir | Link  file(s) into  dir
ls file(s) | List  file(s)
ls dir(s) | List files in  dir(s) or in current directory if  dir(s) is not specified
mkdir dir(s) | Create directory  dir(s)
mv file l file 2 | Move  file 1 to  file 2 (simply rename it if both reference the same directory)
mv file(s) dir | Move  file(s) into directory  dir
ps | List information about active processes
pwd | Display current working directory path
rm file(s) | Remove  files(s)
rmdir dir(s) | Remove empty directory  dir(s)
sort file(s) | Sort lines of  file(s) or standard input if  file(s) not supplied
wc file(s) | Count the number of lines, words, and characters in  file(s) or standard input if  file(s) not supplied
who | Display who’s logged in
