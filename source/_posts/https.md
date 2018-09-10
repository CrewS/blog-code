---
title: 阿里云服务器网站配置https
date: 2018-09-07 15:09:31
tags:
    - https
    - server
    - 阿里云
---

### 前言
https的普及，还有ca证书有免费的申请，让笔者兴起了给服务器增加https协议
学习与实现 配置https
### 开始
Let's Encrypt作为一个公共且免费SSL的项目逐渐被广大用户传播和使用，是由Mozilla、Cisco、Akamai、IdenTrust、EFF等组织人员发起，主要的目的也是为了推进网站从HTTP向HTTPS过度的进程，目前已经有越来越多的商家加入和赞助支持。
目前，默认的免费证书是90天，不过官方有相对应的证书生成客户端。只需要相应配置，即可做到证书一直有效
通过cerbot脚本自动生产安装证书，这里的例子是 阿里云上centos7.4 nginx服务器配置的https

#### 安装cerbot与服务器配置 ([参考地址](https://www.yuzhi100.com/article/centos-7-install-lets-encrypt-certbot))
这里用virtualenv的方式安装
- 安装虚拟环境软件包（针对于Python2.7） `sudo yum install python-virtualenv`
- 创建虚拟环境 `sudo virtualenv /usr/local/python-certbot`
- 进入虚拟环境 `source /usr/local/python-certbot/bin/activate`
- 更新pip `pip install --upgrade pip`
- 安装cerbot `pip install certbot`
- 安装 cerbot nginx 插件（如果使用其他server可安装对应插件）`pip install certbot-nginx`
- 已有nginx服务器下，只需要一键 `sudo certbot --nginx`, 脚本会自动生成证书与修改nginx 配置

至此，正常流程下，对应的 https://域名 可以访问到服务了

#### 问题流程
**安装问题**
参考教程中给出两个常见的安装问题
- [python-urllib3安装失败](https://www.yuzhi100.com/article/centos-7-install-certbot-python-urllib3-failure)
- [pyOpenSSL错误](https://www.yuzhi100.com/article/centos-7-certbot-pyopenssl-missing-required-functionality)
笔者在实践的过程中，也遇到了pyOpenSSL版本问题与其他依赖版本问题，网上参考了一些资料都说，跟着提示走，有什么更新就更新就可以了
`pip update xxxxx`
其中可能会遇到无法更新的问题，我是通过yum erase pyOpenSSL 重新下载才完成（环境依赖的问题错误可能会比较多，不同服务器的问题也不太一样）

**端口问题**
cerbot --nginx配置好了后，访问地址，发现无法访问,可能以下指令排除下问题
- 查看nginx是否监听了443端口服务: `netstat -ntl `, 如果有可以看到443端口对应有nginx服务器
- 查看防火墙是否开放443端口：`firewall-cmd --list-ports`, 查看443/tcp 是否存在
- （重要）阿里云服务器安全策略组是否开放443端口：需要登录到控制台里，安全策略组，添加对应规则443/443端口


可能需要用到的相关指令
```bash
    nginx -t # 测试nginx配置问题
    nginx -s reload # 重启nginx服务器
    nginx -s stop # 关闭nginx服务器
```


