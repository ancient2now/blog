---
title: Scala.01. Install 搭建gitbucket环境
date: 2020-07-31 19:38:57
tags: [Scala, help]
categories: Java
---


gitbucket是提供了一个类似github的功能的一个开源项目，自己的开源项目打算用到这样的功能，先下载研究一下。

gitbukcet使用scala开发的，以前也接触过，相对还比较熟悉些，之前就觉得搭建scala环境还蛮复杂的，主要原因还是网络不太行。

<!--more-->

### scala和sbt 安装

首先要安装或下载的一共有2个：`sbt` 和 `Scala SDK`；[sbt参考文档地址](https://www.scala-sbt.org/1.x/docs/Installing-sbt-on-Linux.html)，[scala参考地址](https://docs.scala-lang.org/)

`Scala SDK`中的sdk是Scala的开发编译包，类似与Java的JDK

#### 先安装sbt吧

`sdk install sbt` 这条指令中的sdk,是管理工具sdkman，可以点击[详细了解](https://sdkman.io/),要想使用这个的话，需要自己安装

对照文档去看，一开始使用`sdk instal sbt`去下载sbt，发现有点慢，还会失败，就换成使用apt去下载，分别执行以下4步
```shell
echo "deb https://dl.bintray.com/sbt/debian /" | sudo tee -a /etc/apt/sources.list.d/sbt.list

curl -sL "https://keyserver.ubuntu.com/pks/lookup?op=get&search=0x2EE0EA64E40A89B84B2DF73499E82A75642AC823" | sudo apt-key add

sudo apt-get update
sudo apt-get install sbt
```

#### sbt安装完成后

这个小节的步骤可以跳过，如果

安装完成sbt后，运行下sbt，可能一开始会下载一些包之类的比较慢，可以考虑换镜像，更换源，比如[华为镜像](https://mirrors.huaweicloud.com/)这个里面能找到

如果想想要下载jar包快点的话，在`～/.sbt`目录下新建一个文件`repositories`,文件的全路径`~/.sbt/repositories`,内容如下，亲测可用
```
[repositories]
#本地源
local
#兼容 Ivy 路径布局
apache-ivy: https://repo1.maven.apache.org/maven2/, [organization]/[module]/(scala_[scalaVersion]/)(sbt_[sbtVersion]/)[revision]/[type]s/[artifact](-[classifier]).[ext]
#apache maven
apache-mvn: https://repo1.maven.apache.org/maven2/
#国内源，aliyun
aliyun: https://maven.aliyun.com/nexus/content/groups/public/
#添加国外源备用
typesafe: https://repo.typesafe.com/typesafe/ivy-releases/, [organization]/[module]/(scala_[scalaVersion]/)(sbt_[sbtVersion]/)[revision]/[type]s/[artifact](-[classifier]).[ext], bootOnly
sonatype-oss-releases
maven-central
sonatype-oss-snapshots
```
如果在项目里面运行的话，idea添加 VM 参数：`-Dsbt.override.build.repos=true`

命令行执行：`sbt -Dsbt.override.build.repos=true clean compile`

#### sbt初使用

可以运行直接运行sbt，进sbt的终端：
```shell
aki@mint:~$ sbt 
```

> [info] [launcher] getting org.scala-sbt sbt 1.3.13  (this may take some time)...  

可能会遇到这个，要耐心等待，就跟maven一样，下载一些包，如果想快点，看上面一小节。  
使用`tail -f $HOME/.sbt/boot/update.log`能看到详细日志。

在sbt终端里，可以使用一下命令查看它的指令：
```shell
sbt:gitbucket> help
```
更详细的介绍在[官方文档](https://www.scala-sbt.org/1.x/docs/sbt-by-example.html)里有哟


#### install scala

1. 下载scala的话，可以使用`sdk install scala`或者去上面的参考地址最下方有下载包的地方以及源码

2. `Scala SDK`可以在【Project Structure】=>【Global Libraries】=>【小加号 +】加上，在【SDKs】里是加不了的，虽然它是叫这个名字

3. 源码可以自己另外下载，这个[地址](https://www.scala-lang.org/download/)的最下面部分，然后在【Project Structure】也就是步骤2完成后，选中Scala SDK的时候，右边部分有个【Standard library】然后点击【加号】加上源码。