---
title: docker初步学习
date: 2018-09-03 11:14:23
tags:
    - docker
---
### 什么是docker？
Docker 项目的目标是实现轻量级的操作系统虚拟化解决方案。 Docker 的基础是 Linux 容器（LXC）等技术。在 LXC 的基础上 Docker 进行了进一步的封装，让用户不需要去关心容器的管理，使得操作更为简便。用户操作 Docker 的容器就像操作一个快速轻量级的虚拟机一样简单。
### 为什么用docker?  
* <span data-type="color" style="color:#1890FF">更快速的交付和部署</span>
    * Docker在整个开发周期都可以完美的辅助你实现快速交付。Docker允许开发者在装有应用和服务本地容器做开发。可以直接集成到可持续开发流程中。
* <span data-type="color" style="color:#1890FF">高效的部署和扩容</span>
    * Docker 容器几乎可以在任意的平台上运行，包括物理机、虚拟机、公有云、私有云、个人电脑、服务器等。 这种兼容性可以让用户把一个应用程序从一个平台直接迁移到另外一个。
    * Docker的兼容性和轻量特性可以很轻松的实现负载的动态管理。你可以快速扩容或方便的下线的你的应用和服务，这种速度趋近实时。
* <span data-type="color" style="color:#1890FF">更高的资源利用率</span>
    * Docker 对系统资源的利用率很高，一台主机上可以同时运行数千个 Docker 容器。容器除了运行其中应用外，基本不消耗额外的系统资源，使得应用的性能很高，同时系统的开销尽量小。传统虚拟机方式运行 10 个不同的应用就要起 10 个虚拟机，而Docker 只需要启动 10 个隔离的应用即可。
* <span data-type="color" style="color:#1890FF">更简单的管理</span>
    * 使用 Docker，只需要小小的修改，就可以替代以往大量的更新工作。所有的修改都以增量的方式被分发和更新，从而实现自动化并且高效的管理。
        
### Docker引擎


![image.png | left | 492x385](https://cdn.nlark.com/yuque/0/2018/png/151680/1535682119390-7a7db477-db85-46f4-ba51-3e9ef2844d87.png "")

* Server是一个常驻进程
* REST API实现了client和server间的交互协议
* CLI 实现了容器和镜像的管理，为用户提供统一的操作界面

### docker核心概念
* 镜像（images）只读模版，用来创建Docker容器
* 仓库（repository）集中存放镜像文件的场所，仓库分公有仓库和私有仓库。最大的仓库是Docker Hub
* 容器（container）

### docker常用命令
* docker pull <images:tag>
* docker images
* docker ps [-a|-l]
    * 列出正在运行的docker容器
    * -a 列出所有已构建的docker容器
    * -l  <span data-type="color" style="color:rgb(51, 51, 51)"><span data-type="background" style="background-color:rgb(255, 255, 255)">显示最近创建的容器</span></span>
* docker rm <container\_id>
### 第一个docker应用
可以发现构建一个docker应用的指令非常的简单
```bash
docker pull hello-world
docker images #列出所有images，发现hello-world 镜像被拉到本地了
docker run hello-world #查看输出了logs信息
```
### Dockerfile构建镜像
镜像可以通过pull从仓库中拉取别人已经做好的镜像，也可以自己制作镜像
通过Dockerfile可以构建属于自己的docker镜像
#### 构建镜像的相关知识
* Dockerfile文件
* docker build指令
    * docker build -t crews/example . 
    * -t 是tag， . 是dockerfile的目录
* docker push【push到dokcer仓库】
#### Dockerfile指令
* CMD： CMD指令用于指定一个容器启动时要运行的命令。这有点儿类似于RUN指令，只是RUN指令是指定镜像被构建时要运行的命令，而CMD是指定容器被启动时要运行的命令。
* ENTRYPOINT：
* WORKDIR：WORKDIR指令用来在从镜像创建一个新容器时，在容器内部设置一个工作目录，ENTRYPOINT和/或CMD指定的程序会在这个目录下执行。
* RUN： 在工作目录下执行命令
* ENV：在dockerfile文件中设置环境变量，在后续任意的RUN指令中可以使用
* VOLUME：设置卷（类似共享目录）【】
* ADD：会将指定文件 添加到镜像中目录下
#### Example
```plain
FROM ubuntu:14.04
MAINTAINER crews "179455570@qq.com"
ENV REFRESHED_AT 2018-08-30
RUN apt-get -yqq update && apt-get -yqq install nginx
RUN mkdir -p /var/www/html/website
ADD nginx/global.conf /etc/nginx/conf.d/
ADD nginx/nginx.conf /etc/nginx/nginx.conf
EXPOSE 80
```
构建镜像：`sudo docker build -t crews/test .`
推送镜像到仓库：`docker push crews/test`
通过镜像创建容器：
`sudo docker run -d -p 80 --name website -v $PWD/website:/home/www crews/test nginx`
创建了容器后下次启动，只需要 docker start <container\_id | name>
__简单理解docker run__
* docker create (利用镜像创建容器)
* docker start (启动该容器)

