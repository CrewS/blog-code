---
title: 前端开发之接口代理
date: 2019-11-16 11:03:58
tags:
	- nginx
	- api
---


## 前言

现在前端开发的方式基本都是前后端分离的开发方式，那就会遇到一些请求接口的问题，比如跨域，比如代理接口地址等等。处理这些问题的方法多种多样，常见的有
devServer proxy代理，nginx的反向代理方式。其原理都是接口代理转发到制定的服务端。

## 权限认证

通过devServer或nginx的proxy可以轻松实现代理，但是权限认证的问题则需根据后端认证方式重新配置。
1、jwt的认证方式
如果采用的是jwt认证方式，改动不大，只需要在接口请求的配置中，将token设置在header中即可，代理的时候会一同携带。

2、cookie session认证方式
如果采用的是cookie、session方式，则需要配置域名，完成登陆逻辑后，cookie会存在该域名下。请求会携带cookie信息，随着转发到指定服务器完成认证。

简单说以下如何配置本地域名
1、hosts文件配置 127.0.0.1 yourhost.com。达到访问yourhost.com:port 相当于 127.0.0.1:prot访问的效果
2、如果需要使用80端口，则增加nginx配置通过启动一个server 将该域名下的服务器转发到本地devServer服务器的端口实现80端口访问。

通过上述的方法，实现了开发阶段单一环境的接口代理。

## 多环境、多域名情况

在实际项目中，会分多套环境。比如开发、测试、生产环境。这样就会遇到一个问题，在测试或者生产环境中发现有问题，但是开发环境的数据并没有问题。这就需要代理调试其他环境的数据了。

在权限认证方式为jwt的情况下，只需要更改代理的IP即可达到环境的切换。但是cookie session的方式，可能需要多思考一步。
目前我们是这样处理的
假设开发域名是A，测试是B，生产是C
则会增加一套B的配置
hosts 127.0.0.1 B
nginx server B
proxy B服务器的ip
全部修改后，重新启动服务器，则可以实现proxy代理转发的测试环境或者生产环境。
但是由于改动的地方比较多，切换环境麻烦所以一般不会切换，只有必要时候才会去切换环境调试。


## 进阶

之前对不同环境的proxy修改，是修改IP地址然后重启实现的。通过阅读devServer [proxy的文档](https://github.com/chimurai/http-proxy-middleware#context-matching)发现*custom matching*，可以自定匹配判断是否转发。
```js
proxy: [
	{
		target: 'http://A ip',
		context: function(pathname, req) {
			if (req.hostname === 'A.cn') return true;
			return false;
		},
	},
	{
		target: 'http://B ip',
		context: function(pathname, req) {
				if (req.hostname === 'B.cn') return true;
				return false;
		}
	},
	{
		target: 'http://C ip',
		context: function(pathname, req) {
			if (req.hostname === 'C.cn') return true;
			return false;
		}
	},
]
```
这样启动服务器的时候就可以针对不同的域名下进行对不同环境的转发效果。这样单一修改host就可以了。


## 最后

通过学习nginx、proxy、Charles等知识内容，可以帮助到前端开发、调试、debug工作。以后工作中遇到一些难解的问题，都能够帮助自己拓宽思路。

## 参考
- [http-proxy-middleware](https://github.com/chimurai/http-proxy-middleware)
- [nginx 语法 ](https://www.cnblogs.com/qinyujie/p/8979464.html)

