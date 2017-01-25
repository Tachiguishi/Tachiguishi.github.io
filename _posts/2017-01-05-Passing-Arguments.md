---
layout: post
title: Chapter 6 Passing Arguments
date: 2017-01-05 21:22:00 +0800
categories:
  - Reading
  - Shell Programming in Unix, Linux and OS X by Stephen G. Kochan
---

之前已经知道，可以将一段程序写入文件然后在运行文件。  
但是这样只能运行固定的代码，其实还可以向这段程序中输入参数使其更具有灵活性

和执行其它shell程序一样，只要在命令后紧跟参数即可。
shell会自动将第一个参数存储在一个特殊的变量`1`中，第二个变量存储在变量`2`中。  
所以在程序中使用使用`$1`, `$2`代替参数应该出现的地方即可  
这被称之为`positional parameters`，因为参数是由其所在的位置决定的

## The `$#` Variable

特殊变量`$#`用于存储在命令行中输入的参数个数。通常用来校验用户数据的参数个数是否正确

```shell
$ cat args                  # Look at the program
echo $# arguments passed
echo arg 1 = :$1: arg 2 = :$2: arg 3 = :$3:
$ args a b c                # Execute it
3 arguments passed
arg 1 = :a: arg 2 = :b: arg 3 = :c:
$ args a b                  # Try it with two arguments
2 arguments passed
arg 1 = :a: arg 2 = :b: arg 3 = :: # Unassigned args are null
$ args                      # Try it with no arguments
0 arguments passed
arg 1 =:: arg 2 =:: arg 3 = ::
$ args "a b c"              # Try quotes
1 arguments passed
arg 1 = :a b c: arg 2 = :: arg 3 = ::
$ ls x*                     # See what files start with x
xact
xtra
$ args x*                   # Try filename substitution
2 arguments passed
arg 1 = :xact: arg 2 = :xtra: arg 3 = ::
$ my_bin=/users/steve/bin
$ args $my_bin              # And variable substitution
1 arguments passed
arg 1 = :/users/steve/bin: arg 2 = :: arg 3 = ::
$ args $(cat names)         # Pass the contents of names
7 arguments passed
arg 1 = :Charlie: arg 2 = :Emanuel: arg3 = :Fred:
```

## The $* Variable

特殊变量`$*`存储了所有的传递参数

```shell
$ cat args2
echo $# arguments passed
echo they are :$*:
$ args2 a b c
3 arguments passed
they are :a b c:
$ args2 one            two
2 arguments passed
they are :one two:
$ args2
0 arguments passed
they are ::
$ args2 *
8 arguments passed
they are :args args2 names nu phonebook stat xact xtra:
```

## A Program to Look Up Someone in the Phone Book

现有文件`phonebook`如下

```shell
cat phonebook
# Alice Chebba 973-555-2015
# Barbara Swingle 201-555-9257
# Liz Stachiw 212-555-2298
# Susan Goldberg 201-555-7776
# Susan Topple 212-555-4932
# Tony Iannino 973-555-1295
```

如果想查询某人的号码可以使用`grep name phonebook`的命令，
如果想输入全名则需要在需要给人名加上引号`""`  
`grep "name" phonebook`


我们可以写一个`lu`程序，接受人名作为参数来查询

```shell
cat lu
# grep $1 phonebook
lu Alice
# Alice Chebba 973-555-2015
lu "Susan T"
# grep: T: No such file or directory
# phonebook:Susan Goldberg 201-555-7776
# phonebook:Susan Topple 212-555-4932
```

可以发现在查询`Susan T`时出错  
这是因为算`Susan T`被当作一个参数传递给了`$1`，但是在执行`grep`命令时变量名被值替代

```shell
grep Susan T phonebook
```

所以应该为`$1`也加上双引号(注意不能使用单引号)

```shell
cat lu
# grep "$1" phonebook
```

## A Program to Add Someone to the Phone Book

```shell
cat add
# echo "$1  $2" phonebook
# sort -u -o phonebook phonebook
```

## A Program to Remove Someone from the Phone Book

```shell
cat rem
# grep -v "$1" phonebook > /tmp/phonebook   # 方法1
# sed "/$1/d" phonebook > /tmp/phonebook    # 方法2
# mv /tmp/phonebook phonebook
```

## ${n}

如果传递的参数超过9个，想获取地10个参数不能直接使用`$10`  
而应该使用`${10}`

## The `shift` Command

`shift`命令会使位置参数集体左移，即`$2`的值赋值给`$1`，`$3`的值赋值给`$2`，以此类推。
而`$1`中的值则会被丢弃

```shell
cat tshift
# echo $# $*
# shift
# echo $# $*
# shift
# echo $# $*
# shift
# echo $# $*
# shift
# echo $# $*
# shift
# echo $# $*
tshift a b c d e
# 5 a b c d e
# 4 b c d e
# 3 c d e
# 2 d e
# 1 e
# 0
```

如果当`$#`为0时去执行`shift`则会报错  

如果想连续多次执行`shift`可以直接在`shift`后跟数字

```shell
shift 3
```
等价于

```shell
shift
shift
shift
```
