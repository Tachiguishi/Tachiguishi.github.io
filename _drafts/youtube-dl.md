---
layout: post
title:  CMake 101
date:   2018-11-14 19:30:00 +0800
categories: tools
tags: youtube-dl
---

## 安装

```shell
dnf install youtube-dl
```

## 使用

### 查看可下载格式列表

```shell
youtube-dl -F [url]
# or
youtube-dl --list-formats [url]
```

## ffmpeg

```shell
sudo dnf install https://download1.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm
sudo dnf install https://download1.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm

sudo dnf install ffmpeg ffmpeg-devel
```

[How to Install FFmpeg on Fedora](https://tecadmin.net/install-ffmpeg-on-fedora/)
