---
title: 《循序渐进学Docker》读书笔记
date: 2017-06-07 18:05:49
tags: [docker, help]
categories: help
description: 看完《循序渐进学Docker》这本书，将书中关键的docker指令记录下来，方便以后查阅
---

## install
* sudo apt-get update
* sudo apt-get intall docker.io

* sudo service docker start
* 检查Docker安装是否成功`sudo docker run hello-world`

## sudo权限
* sudo usermod -aG docker username

## 搭建WordPress个人博客
* docker run --name db --env MYSQL_ROOT_PASSWORD=example -d mariadb
* docker run --name MyWordPress --link db:mysql -p 8080:80 -d wordpress

## 搭建gitlab服务
#### https://github.com/sameersbn/docker-gitlab
1.  启动postgresql
```
docker run --name gitlab-postgresql -d \
    --env 'DB_NAME=gitlabhq_production' \
    --env 'DB_USER=gitlab' --env 'DB_PASS=password' \
    sameersbn/postgresql:9.4-12
```
2. 启动redis`docker run --name gitlab-redis -d sameersbn/redis:latest`
3. 启动gitlab
```
docker run --name gitlab -d \
    --link gitlab-postgresql:postgresql --link gitlab-redis:redis.io \
    --publish 10022:22 --publish 10080:80 \
    --env 'GITLAB_PORT=10080' --env 'GITLAB_SSH_PORT=10022' \
    --env 'GITLAB_SECRETS_DB_KEY_BASE=long-and-random-alpha-numeric-string' \
        sameersbn/gitlab:8.4.4
```

## docker基础
1. **查询镜像** `docker search <string>`
2. **下载镜像** `docker pull learn/turorial` 记得使用全名
3. **创建并启动容器** `docker run learn/turorial echo "hello world"`
4. **修改容器** `docker run learn/tutorial apt-get install -y ping` 在非交互模式下安装软件包，要使用 -y
5. **创建新镜像**,使用 `docker ps -l` 找到安装过ping包的容器的ID号，然后把这个容器提交为新镜像。  
  使用 `docker commit <ID> learn/ping` 把容器提交为新镜像
6. **使用新镜像** `docker run learn/ping ping www.docker.com`
7. **查询容器信息**，使用 `docker ps` 能够查询本机上所有正在运行的容器，使用 `docker inspect <ID>` 可以看到单个容器详细信息，使用容器ID的前3~4个字符来指定
8. **把新镜像上传仓库**，先执行 `docker images` 查看本机的镜像列表，执行 `docker push learn/ping` 把镜像推送到Docker官方仓库

## Docker容器管理

#### 单一容器管理
* 查询完整的CONTAINER ID `docker ps --no-trunc`
* 通过CONTAINER ID查询容器的状态 `docker ps -a | grep 0ee2433coi`
* 停止容器 `docker stop 009oljo902`
* 启动容器 `docker start 0u0ljou0j`
* 创建容器时，使用 --name 参数给容器起一个别名
* 使用 ```docker inspect -f &#123;{.State.Startus}&#125; MyWordPress``` 提取信息
* 使用 `docker logs` 查询日志 
* Docker提供了原生的方式支持登入容器 **docker exec**，使用形式 `docker exec+容器名 + 容器执行的命令`  
  例子：`docker exec MyWordPress ps aux`  
  如果希望执行多个命令，使用 `docker exec -it MyWordPress /bin/bash` 相当于以root身份登入容器
* 删除容器 `docker rm MyWordPress db`

#### 多容器管理
* Docker提供一个容器编排工具——Docker Compose,根据配置模板中的“--link”等参数，对启动的优先级自动排序，执行 `docker-compose up` 就可以把同一个服务中的多个容器依次创建和启动
```
wordpress: 
    image: wordpass
    links: 
        - db:mysql
    ports: 
        - 8080:80
db:
    image: mariadb
    enviroment: 
        MYSQL_ROOT_PASSWORD: example
```
* 启动 `docker-compose start`
* 停止 `docker-compose stop`

## Docker镜像管理


## Dockerfile
* `docker build -t image_redis:v1.0` -t用来取名字

## Docker仓库管理
* `docker login`
* `docker push ubuntu-baseimage:1.0`
* `docker pull centos`



