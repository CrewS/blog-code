---
title: edx-devstack搭建
date: 2018-07-31 17:14:25
tags:
    - edx
---
## <a name="0wimyg"></a>前言
edx devstack搭建的过程也不是一帆风顺的，特别笔者是一名前端工程师，爬了许多坑
接下来，笔者就以一个前端开发者的身份去搭建 open edx devstack
## <a name="i0zovs"></a>起步
环境配置
* Mac OS
* [Docker for Mac](https://docs.docker.com/docker-for-mac/)
* 安装 pip、make
pip安装    `sudo easy_install pip `
#### <a name="nofihb"></a>第一步
克隆代码到本地
`git clone git@github.com:edx/devstack.git`
`cd devstack`
#### <a name="gfp0vp"></a>第二步
打开官方的[README](https://github.com/edx/devstack)
按照教程走,执行以下指令
```
make down
make pull
make dev.up
```

#### <a name="d2a7pr"></a>第三步
* 安装 <span data-type="color" style="color:rgb(36, 41, 46)"><span data-type="background" style="background-color:rgb(255, 255, 255)"> </span></span>[Python virtualenv](http://docs.python-guide.org/en/latest/dev/virtualenvs/#lower-level-virtualenv) 配置虚拟环境
* 设置docker的配置 2G cpu 6G内存，这里不设置 docker会卡死（踩坑）
    > __NOTE:__<span data-type="color" style="color:rgb(36, 41, 46)"><span data-type="background" style="background-color:rgb(255, 255, 255)"> </span></span>Since a Docker-based devstack runs many containers, you should configure Docker with a sufficient amount of resources. We find that [configuring Docker for Mac](https://docs.docker.com/docker-for-mac/#/advanced)<span data-type="color" style="color:rgb(36, 41, 46)"><span data-type="background" style="background-color:rgb(255, 255, 255)"> </span></span>with a minimum of 2 CPUs and 6GB of memory works well.
* 在目录下进入virtualenv，具体方法点击[教程](https://docs.python-guide.org/dev/virtualenvs/#lower-level-virtualenv)（踩坑）
* 执行命令 
    ```
    make requirements
    make dev.clone
    make dev.provision
    ```
命令全部完成后，devstack搭建完成
* 对应的服务列表

| Service | URL |
| :--- | :--- |
| Credentials | [http://localhost:18150/api/v2/](http://localhost:18150/api/v2/) |
| Catalog/Discovery | [http://localhost:18381/api-docs/](http://localhost:18381/api-docs/) |
| E-Commerce/Otto | [http://localhost:18130/dashboard/](http://localhost:18130/dashboard/) |
| LMS | [http://localhost:18000/](http://localhost:18000/) |
| Notes/edx-notes-api | [http://localhost:18120/api/v1/](http://localhost:18120/api/v1/) |
| Studio/CMS | [http://localhost:18010/](http://localhost:18010/) |

    打开[http://localhost:18010/](http://localhost:18010/) 即可看到cms页面
* 退出docker关闭服务  `make stop`

