---
title: docker初体验
date: 2017-04-01 18:05:49
tags: [docker]
categories: HelpDocument
---

## 一. docker的主要目标

通过对应用组件的封装，分发，部署，运行等生命周期的管理，达到应用级别的一次封装，到处运行。

## 二. docker容器虚拟化的优点

环境隔离  
更快速的交付部署  
更高效的资源利用  
更迁移扩展  
更简单的更新管理

<!-- more -->

`docker build -t="vagabondize/test"`  
`docker rmi vagabondize/test`  
`docker run --name test1 vagabondize/test`

## 三. docker镜像

镜像是一个只读的模板  

`docker inspect <imageID>`  
`docker inspect -f &#123;{.Os}&#125; <imageID>`

### 创建镜像

1. 基于原有容器创建新的镜像  

    `docker run -ti ubuntu bash`

    `docker commit -a "vagabondize" -m "message" <containerID> vagabondize/test`

2. 迁出，载入镜像

    `docker save -o test.tar <imageID>`

    `docker load --input test.tar`

    `docker tag <imageid> "vagabondize/test"`

    `docker push <域名>/<namespace>/<repo>:<tag>`

3. 删除镜像

    `docker rmi <image_id>`

## 四. docker容器

容器是从镜像创建的运行实例，每个容器都是相互隔离的  

### 1. 创建容器

1) create只创建

    `docker create --name test_create -ti ubuntu`

2) run 创建并运行（create + start）

### 2. 使用容器

1. 进入容器

   启动时使用bash `docker run -i -t ubuntu /bin/bash`

   进入已启动容器的内部 `docker exec -i -t <container_id> bash`

2. 守护台运行

    后台启动容器 `docker run -d ubuntu /bin/sh -C "while true;do echo hello world; sleep 1; done"`

    查看容器日志 `docker logs -f <container_id>`

    终止容器 `docker stop <container_id>`

3. 查看容器

    查看终止的容器：`docker ps -a`

    查看运行的容器：`docker ps`  

4. 删除容器

    `docker rm <container_id>`

    `docker rm -f <container_id>`

5. 导入和导出容器

    `docker export <container_id> > test.tar`

     `cat test.tar | docker import - vagabondize/hello:test`

## 五. docker仓库

1. 私有仓库

    registry

2. 管理私有仓库

    docker-registry-ui

## 六. 数据卷

 -v 标记创建一个数据卷，映射目录

 `docker run -d --name app1 -it -v ${PWD}/webapp:/root/webapp ubuntu bash`

 :ro 只读   默认:rw

 `docker run -d --name atest -v /root/test -ti ubuntu bash`

 `docker run -d --name btest --volumes-from atest -ti ubuntu bash`

### 备份

`docker run -d --name dbdata -v /dbdata -ti ubuntu bash`

`docker run --volumes-from dbdata -v ${PWD}:/backup --name worker ubuntu \ tar cvf /backup/backup.tar /dbdata`

`docker run -d --name dbdata2 -v /dbdata -ti ubuntu bash`

`docker run --volumes-from dbdata2 -v ${PWD}:/backup ubuntu / tar xvf /backup/bacup.tar`

## 七. docker网络

## centos install

`sudo yum install net-tools`  

`sudo usermod -aG docker username`

配置信任仓库  
/lib/systemd/system/docker.service  添加  --insecure-registry 192.168:5000  
`sudo systemctl daemon-reload`  
`service docker restart`  

添加加速器

监听 netstat -nlp | grep 8080
