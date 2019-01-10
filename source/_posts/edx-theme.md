---
title: edx-theme
date: 2018-10-16 10:51:54
tags:
    - edx
---
## edx主题相关教程

### 第一步当然是阅读官方文档 [地址](https://edx.readthedocs.io/projects/edx-installing-configuring-and-running/en/latest/configuration/changing_appearance/theming/index.html)

### 对edx原有主题的更换

查看目前edx原来有的主题目录是 /themes 
该目录下有以下内容
```
    ├── conf
    ├── dark-theme
    ├── edge.edx.org
    ├── edx.org
    ├── mytheme
    ├── open-edx
    ├── red-theme
    └── stanford-style
```
了解了这些内容后开始进行edx主题的切换[以修改studio为例子]
#### 第一步：修改环境变量 （）
```bash
cd devstack # 进入devstack目录
make dev.up # 启动服务器
docker ps # 查看  edx.devstack.studio 的容器ID
docker exec -it containerid  /bin/bash # 进入容器
cd /edx/app/edxapp # 进入文件目录
vim cms.env.json
# 修改环境变量文件 ENABLE_COMPREHENSIVE_THEMING: true
# "COMPREHENSIVE_THEME_DIRS": ["/edx/app/edxapp/edx-platform/themes"],
#  "THEME_NAME": "red-theme"
```
环境变量至此配置完毕

### 第二步：重启服务并更新静态资源
```bash
cd devstack 
docker-compose restart
make studio-shell # 进入studio的shell 以便执行paver 指令
paver update_assets cms --settings=devstack_docker # 编译静态资源
```
如果出现访问不成功的情况可以 ```make studio-logs``` 查看相应服务器的日志信息

### 第三步：登录后台配置
访问 `http://0.0.0.0:18010/admin` 账号密码 edx/edx 登录
找到 Site themes 选项 点击进入
点击 ADD SITE THEME 按钮添加网站主题
选择目标域名，如果没有需要手动添加，点击+号填写表单 域名0.0.0.0:18010，名字studio
填写主题目录 red-theme
点击save 保存成功
访问配置主题的目标网站这里是（http://0.0.0.0:18010），刷新页面发现主题配置成功！


## 回顾难点
- 需要了解docker 相关知识，edx相关配置基础
- 文档不够详尽，需要有一定知识才能够配置
- 出错信息没有明显提醒，需要了解相关知识才能自发通过日志系统查阅