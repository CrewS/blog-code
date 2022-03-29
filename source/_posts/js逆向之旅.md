---
title: js逆向之旅（一）
date: 2021-06-01 23:12:48
tags:
	- js逆向

banner_img: /img/bg/banner.webp
index_img: /img/bg/banner.webp
---


## 前言


之前有尝试过破解极验的`w`值，其中有涉及到反混淆、加解密等知识，一步一步的得到最后的结果非常的有意思。最近有幸了解到《猿人学-爬虫刷题平台》网站，有许多有趣的js逆向题目。以下就是js逆向之旅（刷题之旅）。


## js 混淆 - 源码乱码
环境
操作系统: mac
语言: node
编辑器：vscode


题目地址：[http://match.yuanrenxue.com/match/1](http://match.yuanrenxue.com/match/1)
概述：看了下题目大概就是请求5个接口然后拿返回的结果做下统计求平均值
#### 第一步 反调试
打开浏览器控制台看下请求，就开始报错，找到对应的js：`uyt.js` `uzt.js`
右键添加断点条件设置为false后面就不会弹窗报错了。
![image.png](https://crews-note-1253247308.cos.ap-guangzhou.myqcloud.com/note/20220307002723.png)
清空Netwrok，看到对应的请求接口 
![image.png](https://crews-note-1253247308.cos.ap-guangzhou.myqcloud.com/note/20220307002757.png)
可以看到每次请求都会携带一个m值的传参，这个就是我们要解密的m值。

#### 第二步 格式化代码
通常我会将html的源代码放在vscode编辑器里，然后格式化一下，得到一个比较好看的结果。
然后把最底下的script标签全部拷贝到新的js文件中，去掉标签再格式化一次，如下图所示。
![image.png](https://crews-note-1253247308.cos.ap-guangzhou.myqcloud.com/note/20220307002832.png)
发起的请求一般是**$.ajax**，搜索关键字 **$.ajax**
找到参数m，发现是由**oo0O0**函数生成的
![image.png](https://crews-note-1253247308.cos.ap-guangzhou.myqcloud.com/note/20220307002852.png)
#### 第三步 解密
接下来研究**oo0O0**函数即可。
阅读了一下代码发现需要window、和document对象 重新看html源码找到
把需要的变量都收集起来
![image.png](https://crews-note-1253247308.cos.ap-guangzhou.myqcloud.com/note/20220307002914.png)
发下c需要计算，其实可以直接在控制台输出c然后拿到值赋值就好。
![image.png](https://crews-note-1253247308.cos.ap-guangzhou.myqcloud.com/note/20220307002936.png)
重新整理顺序，然后执行：这里是用node的环境 `node run.js`
发现报错了！提示 ReferenceError: atob is not defined
![image.png](https://crews-note-1253247308.cos.ap-guangzhou.myqcloud.com/note/20220307002956.png)
瞬间想到这个是node的环境并不是浏览器环境，没有atob这个函数。so，直接实现一个node base64解码
```javascript
const atob = (str) => {
  return (new Buffer.from(str, 'base64')).toString()
}
```
重新执行一次 输出结果！ m值GET
![image.png](https://crews-note-1253247308.cos.ap-guangzhou.myqcloud.com/note/20220307003121.png)
#### 第四步 构造请求
接下来就是构造5个请求把获取回来的值算下平均值，然后得到答案！




## 总结


1、学习到反调试技巧
2、学习到格式化代码与信息量收集
3、了解node环境与浏览器环境差异，一些api的缺少后续逆向还是会遇到。