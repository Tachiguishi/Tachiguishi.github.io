---
layout: post
title:  Shell
date:   2019-08-27 19:15:00 +0800
categories: tools
tags: shell
---

## file

```shell
filename=$(basename -- "$fullfile")
extension="${filename##*.}"
filename="${filename%.*}"
```

## sed

```shell
sed '/ABC/d' # 删除包含ABC的行
sed '/ABC/a123' # 在包含ABC的行后面添加一行数值为123的行
```
