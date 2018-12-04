---
layout: post
title:  CMake 101
date:   2018-11-14 19:30:00 +0800
categories: tools
tags: docker
---

`Docker`是一个提供操作系统级别虚拟化(容器化)的程序，可以使每个用户程序运行在单独的容器内部，实现了独立与自动化部署。  
开源地址[Github/Docker](https://github.com/docker/docker-ce)

## 安装与配置

### 安装

* [Fedora](https://docs.docker.com/install/linux/docker-ce/fedora/)

```shell
sudo dnf config-manager --add-repo https://download.docker.com/linux/fedora/docker-ce.repo
sudo dnf install docker-ce
```

### 验证：

```shell
docker --version
#=> Docker version 18.09.0, build 4d60db4
docker info
#=> 显示更多的详细信息
```

### 添加当前用户到`docker`用户组

避免每次都需要使用`sudo`

```shell
sudo usermod -aG docker $USER
```

### 启动

```shell
systemctl start docker
# 开机自启动
systemctl enable docker
```

### 运行`Hello world`

```shell
docker run hello-world
# 查看已有的image
docker image
# 查看运行的container
docker container ls
# 查看所有container
docker container ls --all
# 查看所有未运行的container
docker container ls -aq
```

### 配置修改

可以通过修改`/usr/lib/systemd/system/docker.service`文件来修改`docker daemon`的启动配置，
但通常通过`sudo systemctl edit docker.service`命令来修改，
其修改的配置在`/etc/systemd/system/docker.service.d/override.conf`文件中，其中的配置会覆盖`docker.serice`中的配置

```shell
sudo systemctl edit docker.service
sudo systemctl daemon-reload
sudo systemctl restart docker.service
```

## 运行基本命令

```shell
docker build -t friendlyhello .  # Create image using this directory's Dockerfile
docker run -p 4000:80 friendlyhello  # Run "friendlyname" mapping port 4000 to 80
docker run -d -p 4000:80 friendlyhello         # Same thing, but in detached mode
docker container ls                                # List all running containers
docker container ls -a             # List all containers, even those not running
docker container stop <hash>           # Gracefully stop the specified container
docker container kill <hash>         # Force shutdown of the specified container
docker container rm <hash>        # Remove specified container from this machine
docker container rm $(docker container ls -a -q)         # Remove all containers
docker image ls -a                             # List all images on this machine
docker image rm <image id>            # Remove specified image from this machine
docker image rm $(docker image ls -a -q)   # Remove all images from this machine
docker login             # Log in this CLI session using your Docker credentials
docker tag <image> username/repository:tag  # Tag <image> for upload to registry
docker push username/repository:tag            # Upload tagged image to registry
docker run username/repository:tag                   # Run image from a registry
```

## 参考资料

* [官方文档](https://docs.docker.com/)
* [Docker — 从入门到实践](https://yeasy.gitbooks.io/docker_practice/content/)
