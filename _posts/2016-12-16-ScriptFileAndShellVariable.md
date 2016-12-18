---
layout: post
title:  Script file And Shell variable
date:   2016-12-16 09:57:00 +0800
categories:
  - Reading
  - Shell Programming in Unix, Linux and OS X by Stephen G. Kochan
---

## Command Files

可以将一串命令写入文件，然后再执行该文件就可以了。  
为了执行文件，首先要确定文件是可执行的  
可以使用`ls -l file`命令查看文件的权限

```shell
ls -l XMusicSetup.exe
# -rwxr-xr-x 1 Jason None 15474112 Oct 25  2015 XMusicSetup.exe
```

`-`没有相应权限，`r`可读，`w`可写(编辑)，`x`可执行

如果文件不可执行，需要通过`chmod +x file(s)`命令添加执行权限

#### 注释

任何跟在`#`字符后面的内容都会被忽略

## 变量

#### 变量名

必须符合`[A-Za-z_][a-zA-Z0-9_]*`

#### 赋值

```shell
variable=value
```

注意：
* 赋值操作`=`两边不能有空格  
* shell 没有数据类型的概念，所有值都被当作字符来处理

#### 输出

```shell
count=1
echo $count
```

`$`字符会转换其后跟的变量为其所代表的值。这种转变会发生在任何地方  

```shell
sortCommand=sort
$sortCommand file   # 等效于 sort file
```

将一个变量的值赋给另一个变量

```shell
var1=3
var2=$var1
```

#### 空值

如果一个变量没有被赋值或没有被定义则为空值，即没有任何字符

手动定义空值变量  

```shell
nullVar=
nullVar=""
nullVar=''
```

#### 文件名替换

考虑以下情况   

```shell
var1=*
echo $var1  # 输出 pwd 下的文件名
```

注意，这里变量`var1`被赋予的值仅仅是字符`*`，此时并没有发生替换  
在执行`echo $var1`命令时，首先发生变量替换将命令变为`echo *`，  
然后发生文件名替换`*`并替换为当前目录下的文件名

#### `${variable}`

假如变量`filename`中存储了一个文件名，现在你想将该文件重命令，在其原名后面添加一个`X`  
如果直接执行

```shell
mv $filename $filenameX
```

则得不到想要的结果  
因为`$filenameX`并识别为名为`filenameX`的变量。正确方式应该为  

```shell
mv $filename ${filename}X
```

## 内置数值计算

```shell
$((expression))
```

```shell
i=1
$((i++))  # 这里的变量 i 无需 $ 前缀
echo $i   # 2
```
