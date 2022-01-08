---
title: docker统一构建
date: 2019-12-09 20:14:10
tags:
	- docker
	- 构建
---



### 前言

最近工作中会遇到这样的一个问题，当有个依赖包更新，或者某些配置需要更新的时候，需要在目前处理的所有系统中一起更新，这样的工作量其实是挺繁琐的。（多个系统配置几乎一致）。所以就想着能够有方法统一这个构建环境，包依赖和webpack配置能够统一处理。



### 实践过程

- 第一步：#抽取公共配置作为配置仓库 docker-front
- 第二步：编写Dockerfile，以node镜像为基础，拉取配置仓库为内容的镜像
- 第三步：编写docker-compose.yml。定义port，共享文件夹卷，定义启动后的指令操作
- 第四步：整理业务系统代码。项目目录剩下 src目录（处理业务），config目录 （处理配置）
- 第五步：docker-compose up启动项目



#### 抽取公共配置作为仓库docker-front

一般来说公共配置如下

- webpack.base.config.js
- webpack.dev.config.js
- webpack.pro.config.js
- package.json
- index.ejs

还有一些可能有公共的常量配置等等，可以继续往上添加

#### 编写Dokcerfile

这里docker主要是提供了一个环境，拉取配置仓库代码

```dockerfile
FROM node:8.9.3
RUN git clone http://username:password@gitlab.com/docker-front.git
WORKDIR /docker-front

```

#### 编写docker-compose.yml

yourapp 定义docker-compose名字

prots 定义暴露的端口

volumes 定义了共享的目录

command 定义启动时候执行的指令

```yaml
version: "3"

services:
  yourapp:
    build: .
    ports:
      - 5000:5000
    # stdin_open: true
    volumes:
      - "./src:/docker-front/src"
      - "./config:/docker-front/config"
    # 这段指令是每次启动都会拉取最新的代码执行npm i再启动
    command: 
      - /bin/sh
      - -c
      - |
        git reset --hard HEAD
        git fetch
        git checkout dev
        git pull
        npm i
        npm start
    tty: true
    
```

#### 整理业务系统代码

src 主要是用作映射业务系统的业务代码

config 的作用是用于配置各个业务系统中个性化的配置

比如webpack中独立的配置、index.ejs中的项目名称、配置的项目独立私有的key等等

目前还没有做独立的包的处理。不过有思路就是config中增加_package.json文件，在启动项目的时候增加指令处理package.json，将业务系统的私有配置和公共package.json配置合并。再执行包安装指令



#### Docker-compose up

通过这个指令可以构建镜像，创建容器和启动容器。如果想要重新构建镜像可以`docker-compose build`指令处理

更多的操作可以找相关的教程。



#### Docker统一构建的优势

- 解决了公共配置在多个系统里重复配置的问题
- 解决了环境不统一的问题，node版本npm 版本
- 解决了项目window和Mac 差异性问题
- 提供给非前端使用的可能性，无需配置简单执行指令即可启动项目



#### 带来的问题也是有的

- docker占用资源问题，是否构建速度下降？
- 公共配置管理的问题，需要更严格的处理，不然影响范围比较大





