---
title: 工欲善其事必先利其器——js逆向基础知识
date: 2022-02-28 21:36:36
tags: [js逆向, 爬虫]

index_img: /img/post/svelte.png
banner_img: /img/bg/banner.webp
---

## 前言
这几年随着爬虫与反爬的对抗强提升，web端的js被加强了防护。想对js解密也没有以前那么简单，控制台打开，参数一看就可以开始请求接口。
所以对爬虫也有一定的技术要求。这里将整理js逆向的一些方法论和技术。

## 总览
自己的一些观点： 对于web端的反爬只要抓住请求是最后的“出口“就能顺着执行思路往回逆向。接口请求是客户端与server的交互，通过一定的参数让服务器“鉴权”通过，认可这个是一个“合法”客户端发出的请求，同时返回信息。反爬的侧重点就到了对接口请求过程的防护了。
- 对参数加密
- 对协议的特殊处理
- 对返回的加密
其中加密的手段又通过“混淆”，“重新编码”，“jsvmp”等方式隐藏真正的加密方式
payload里也会附加 设备指纹、行为轨迹、等等的信息
对于“爬虫”来说，就需要抽丝剥茧，把这些信息获取到进而伪装成正常的用户发起请求
那对围绕这个“抽丝剥茧”的事情，就有很多工具方法，通过学习这些基础的知识，可以有效提升逆向效率

## JS hook

JS hook，js里经常用到的技术之一，通过js hook我们可以监听到一个数据的变化和使用的过程。
JS hook主要利用了浏览的API
1、[Object.defineProperty()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty) 不兼容IE8以下
2、[Proxy](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy) IE 不兼容
本质上都是对变量进行监听，可以对对象添加2个方法，get、set方法，get方法在对象使用的时候就会触发，set 是在对对象继续赋值的时候会触发。
使用场景之一：document.cookie的监听
有一些网站就是加密参数写入cookie中，然后随着请求一起发送到服务器进行校验
对于cookie的变更，js hook可以很快找到赋值的位置，再从堆栈中找到参数加密的位置
```js
(function () {
  'use strict';
  var cookieTemp = '';
  Object.defineProperty(window.byted_acrawler, 'sign', {
    set: function (val) {
      // 通过判断含有特定的cookie 再断点
      // if (val.indexOf('name') != -1) {
			// }
			debugger;
      console.log('Hook捕获到cookie设置->', val);
      cookieTemp = val;
      return val;
    },
    get: function () {
      return cookieTemp;
    },
  });
})();
```
利用Proxy监听数据的改变也一样
下面是一个基础的proxy函数

```js
const proxy = function(obj) {
	return new Proxy(obj, {
		set: (target, prop, val) => {
			console.log("SET>>>>", prop, prop, val)
			return Reflect.set(...arguments);
		},
		get: (target, prop, r) => {
			console.log("GET>>>>", prop, prop, target[prop])
			return target[prop]
		},
	})
} 
// 假设要对window进行监听
window = proxy(window)
```

## 利用Chrome开发者工具与vscode（IDE）
利用浏览器开发者工具或者vscode调试界面对js代码调试，能够方便我们理解加密逻辑
这里介绍几个方法和技巧

#### Chrome浏览器断点

先配置Chrome浏览器的开发者工具的语言环境

F12打开控制台，最上面的工具栏右侧有个齿轮，点击可以找到preference=> language => 设置中文

![截屏2022-03-01 下午10.31.24](https://crews-note-1253247308.cos.ap-guangzhou.myqcloud.com/note/%E6%88%AA%E5%B1%8F2022-03-01%20%E4%B8%8B%E5%8D%8810.31.24.png)

这时候就变成中文的控制面板，对于不熟悉Chrome 开发中工具且英文不好的同学来说是比较好的帮助

![image-20220301223526150](https://crews-note-1253247308.cos.ap-guangzhou.myqcloud.com/note/image-20220301223526150.png)

##### Chrome浏览打断点的几个技巧

**添加断点** 

顾名思义就是加断点，这个没有什么好讲的



**添加条件断点** 

就是可以根据上下文环境变量加入一些判断，达到条件就会触发断，可以简单的理解为下面的代码插入该行

```js
if(condition) {
	debugger
}
```

**添加日志点**

添加日志输出点则是每运行到该行进行输出信息，用的场景比较多是研究加密算法运算过程某些值的变化规律



**一律不在此处断点**

这个功能则是对一些简单的反爬debugger进行忽略

![截屏2022-03-01 下午10.36.12](https://crews-note-1253247308.cos.ap-guangzhou.myqcloud.com/note/%E6%88%AA%E5%B1%8F2022-03-01%20%E4%B8%8B%E5%8D%8810.36.12.png)



##### Chrome浏览器的network

可以观察到的几个点

- 请求的顺序
- 请求的协议以及请求体、响应体
- 调用的堆栈信息

![image-20220301224342082](https://crews-note-1253247308.cos.ap-guangzhou.myqcloud.com/note/image-20220301224342082.png)

**network**给出的关键信息也能辅助我们快速定位到接口发起的地方，从而逆向参数生成的位置



##### Chrome 油猴脚本插件

油猴脚本注入的js能够页面加载之前进去，所以js逻辑生效的时机也能够特别早

对于cookie的hook，可以通过油猴脚本的方式注入，第一时间能够观察到cookie的变化


## 抓包

Win、Mac 都有抓包的工具
这里主要讲下Mac里使用的抓包工具 Charles
通过抓包工具可以看到一些跳页的请求，network里有时候难以看到的请求，还能通过Charles 代理某些地址到本地文件
比如首页替换、js替换等等

**Tool -> Map local Settings -> add -> Edit Mapping**

![Map local设置](https://crews-note-1253247308.cos.ap-guangzhou.myqcloud.com/note/image-20220302224039568.png)



## 后续



2022年03月02日 暂时先补充这么多方法和技巧，后续学习中还有别的了解会继续添加在上面



