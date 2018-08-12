---
title: nginx 学习笔记（一）
date: 2018-08-12 17:33:33
tags:
    - nginx
    - server
---
### 前言
>Nginx (engine x) 是一个高性能的HTTP和反向代理服务器，也是一个IMAP/POP3/SMTP服务器。
在前端的开发过程中，经常会遇到一些跨域的问题，这时候可能就需要到配置nginx，做一个反向代理
既然nginx这么有用，让我们开始学习如何使用
### 开始
如何快速开始nginx的使用呢
在mac电脑上，可以通过brew快速安装 `brew install nginx`  
在终端命令行中 nginx 就可以启动了，一个默认的nginx服务器

### 指令
```bash
nginx -t        // 检查配置是否正确
nginx -s reload // 重启nginx服务器
nginx -s stop   // 关闭nginx服务器
```

### 配置
默认配置文件的目录  `/usr/local/etc/nginx`  
默认服务器目录 `/usr/local/var/www`


### 反向代理
日常开发过程中，会有遇到在`localhost:8080`域名下，跨域请求`xxx.xxx.com`下的接口
如果后端不做配置的话，  
前端也可以通过反向代理，请求成功  
打开 `/usr/local/etc/nginx/nginx.conf`文件  
在http{}中增加 server{}模块  
具体设置：
```
server{
    listen       80;
    server_name  servername.com;
    location / {
        proxy_pass localhost:8080
    }
    location /api {
        proxy_pass http://xxx.xxx.com
    }
}
```
这样访问 `servername.com`的时候，直接获取到`localhost:8080`的资源，当访问`servername.com/api` 下的接口的时候，会被代理转发到 `htto://xxx.xxx.com`，这样就解决了跨域问题

### 奇怪的“BUG”
server_name 配置问题
场景：

配置了nginx server ，监听了端口，设置好server_name 后，第一次访问这个server_name是生效的

再次修改这个server_name后，第二次设置的server_name不生效，且第一次设置的生效

假定第一次设置 test.com 第二次设置abc.com

修改配置后 test.com还生效，abc.com不生效

最后查了许多资料，发现是server_name的问题

因为修改location的配置的时候，这个配置是生效的，只是域名不匹配

最后得出解决办法是

配置一个新的server_name的时候

需要修改hosts文件

`127.0.0.1  server_name`

配置好了以后，发现访问server_name 有效

