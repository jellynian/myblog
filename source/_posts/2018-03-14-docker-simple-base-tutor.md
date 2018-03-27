---
layout: post
title: "docker 精简基础教程"
tagline: "docker" 
description: ""
category: sysadmin
key:
tags: [docker,sysadmin]
published: true
last_updated: 2018年 3月14日 星期三 17时28分01秒 CST
---
## 安装docker

> 官方文档的参考链接 [https://docs.docker.com/install/linux/docker-ce/ubuntu/#install-docker-ce-1](https://docs.docker.com/install/linux/docker-ce/ubuntu/#install-docker-ce-1)

 卸载旧版本docker

    sudo apt-get remove docker docker-engine docker.io
    sudo apt install software-properties-common #安装工具

 信任密钥:

     curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
    
     
添加源:
    
     sudo add-apt-repository "deb [arch=amd64] https://mirrors.aliyun.com/docker-ce/linux/ubuntu  $(lsb_release -cs) stable"


安装docker

    sudo apt-get install docker-ce -y




默认情况下，`docker`	命令会使用Unix	socket	与	Docker	引擎通讯。而只有`root`用户和`docker	`	组的用户才可以访问	Docker	引擎的	Unix	socket。出于 安全考虑，一般	Linux	系统上不会直接使用`root	`	用户。因此，更好地做法是将 需要使用	docker		的用户加入	`docker	`	用户组。

如果不存在`docker`组则建立		docker		组：

    $	sudo groupadd docker
    
将当前用户加入docker组：

    $	sudo usermod -aG docker $USER

之后logout当前用户，再login 就可以使用普通用户操作docker了

    
## 拉取镜像：

    docker pull ubuntu

运行docker容器

    docker run -it ubuntu /bin/bash 
    
Docker	包括三个基本概念
镜像（Image） 容器（Container） 仓库（Repository）


## 获取镜像

从	Docker	Registry	获取镜像的命令是		docker	pull	。其命令格式为

    docker	pull	[选项]	[Docker	Registry地址]<仓库名>:<标签>
    
拉取ubuntu 16.04的镜像

    docker	pull	ubuntu:16.04 


拉取alpine镜像
> Alpine Linux 是一个社区开发的面向安全应用的轻量级Linux发行版,只有4M左右。

    docker pull alpine

列出已经下载到的镜像

    
    jelly@MiWiFi-R3-srv:~$ docker images
    REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
    alpine              latest              3fd9065eaf02        7 weeks ago         4.15MB

基本的操作

    docker ps # 查看当前运行中的容器
    docker ps -a # 查看所有的容器
    docker top 容器名或容器id #查看容器中运行的进程
    docker attach 容器名或容器id # attach 如果要正常退出不关闭容器，请按Ctrl+P+Q进行退出容器。
    docker exec -it 容器名或容器id /bin/bash # 以交互的方式在容器中运行一个进程
    

使用`docker commit` 构建镜像

    docker run --name webserver  -d -p 80:80 nginx 
    
    docker exec -it webserver /bin/bash
    
    echo "<h1> Hello World </h1>" > /usr/share/nginx/html/index.html

`docker commit` 的帮助

    docker commit  --help
    
    Usage:	docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]
    
    Create a new image from a container's changes
    
    Options:
      -a, --author string    Author (e.g., "John Hannibal Smith <hannibal@a-team.com>")
      -c, --change list      Apply Dockerfile instruction to the created image
      -m, --message string   Commit message
      -p, --pause            Pause container during commit (default true)
  
    
打开页面查看效果

    curl http://localhost:80 
      
提交一个新镜像到本地

    docker commit -a "jelly <jelly@yunio.com>" -m "modify index.html"  webserver cagetest_helloworld:v1.0

