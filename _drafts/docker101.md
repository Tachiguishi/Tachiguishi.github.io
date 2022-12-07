---
layout: post
title:  Docker 101
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

## Image的导入与导出

```shell
docker save imageID > out.tar	# 会删除image的name和tag信息
## 导出多个的image
docker save -o out.tar <repo1>:<tag1> <repo2>:<tag2>

docker load < out.tar
```

```shell
docker export containerID > out.tar
docker import - newImageName < out.tar
```

### 区别

docker import 可以为镜像指定新名称
docker load 不能对载入的镜像重命名

export 导出（import 导入）是根据容器拿到的镜像，再导入时会丢失镜像所有的历史记录和元数据信息（即仅保存容器当时的快照状态），所以无法进行回滚操作。
而 save 保存（load 加载）的镜像，没有丢失镜像的历史，可以回滚到之前的层（layer）。

docker export 的应用场景：主要用来制作基础镜像，比如我们从一个 ubuntu 镜像启动一个容器，然后安装一些软件和进行一些设置后，使用 docker export 保存为一个基础镜像。然后，把这个镜像分发给其他人使用，比如作为基础的开发环境。
docker save 的应用场景：如果我们的应用是使用 docker-compose.yml 编排的多个镜像组合，但我们要部署的客户服务器并不能连外网。这时就可以使用 docker save 将用到的镜像打个包，然后拷贝到客户服务器上使用 docker load 载入。

## 参考资料

* [官方文档](https://docs.docker.com/)
* [Docker — 从入门到实践](https://yeasy.gitbooks.io/docker_practice/content/)


## 使用指定用户进入Container

```shell
docker run -it -u user_name --name container_name -d image_name /bin/bash

docker exec -it -u user_name containerID /bin/bash
```