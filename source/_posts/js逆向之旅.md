---
title: js逆向之旅（一）
date: 2021-06-01 23:12:48
tags:
	- js逆向

banner_img: /img/bg/banner.webp
index_img: /img/bg/banner.webp
excerpt: 这是摘要
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
![image.png](https://cdn.nlark.com/yuque/0/2021/png/151680/1622562064467-35f5a77c-27af-4f4c-b84b-07ed75b87a0a.png#height=373&id=rzlkY&margin=%5Bobject%20Object%5D&name=image.png&originHeight=746&originWidth=1230&originalType=binary&size=119500&status=done&style=none&width=615)
清空Netwrok，看到对应的请求接口 
![image.png](https://cdn.nlark.com/yuque/0/2021/png/151680/1622562187800-876e4fe3-9913-4958-9a3f-da6934596b90.png#height=91&id=VKqXW&margin=%5Bobject%20Object%5D&name=image.png&originHeight=182&originWidth=1704&originalType=binary&size=41755&status=done&style=none&width=852)
可以看到每次请求都会携带一个m值的传参，这个就是我们要解密的m值。

#### 第二步 格式化代码
通常我会将html的源代码放在vscode编辑器里，然后格式化一下，得到一个比较好看的结果。
然后把最底下的script标签全部拷贝到新的js文件中，去掉标签再格式化一次，如下图所示。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/151680/1622562603808-1aa3f000-bbb2-4359-8ffe-7097abe041fc.png#height=431&id=Cw4wk&margin=%5Bobject%20Object%5D&name=image.png&originHeight=862&originWidth=908&originalType=binary&size=92346&status=done&style=none&width=454)
发起的请求一般是**$.ajax**，搜索关键字 **$.ajax**
找到参数m，发现是由**oo0O0**函数生成的
![image.png](https://cdn.nlark.com/yuque/0/2021/png/151680/1622562874665-bedd3de8-b79c-424a-984f-dd26fbbafeff.png#height=95&id=TxLRw&margin=%5Bobject%20Object%5D&name=image.png&originHeight=190&originWidth=1142&originalType=binary&size=43356&status=done&style=none&width=571)
#### 第三步 解密
接下来研究**oo0O0**函数即可。
阅读了一下代码发现需要window、和document对象 重新看html源码找到
把需要的变量都收集起来
![image.png](https://cdn.nlark.com/yuque/0/2021/png/151680/1622564194490-44c74078-140d-4068-9abe-985da6644164.png#height=127&id=olfgr&margin=%5Bobject%20Object%5D&name=image.png&originHeight=254&originWidth=1736&originalType=binary&size=66569&status=done&style=none&width=868)
发下c需要计算，其实可以直接在控制台输出c然后拿到值赋值就好。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/151680/1622564439686-8d6af3f2-c00e-4bc4-b62e-e68e23c09225.png#height=287&id=N6B6s&margin=%5Bobject%20Object%5D&name=image.png&originHeight=574&originWidth=1262&originalType=binary&size=102406&status=done&style=none&width=631)
重新整理顺序，然后执行：这里是用node的环境 `node run.js`
发现报错了！提示 ReferenceError: atob is not defined
![image.png](https://cdn.nlark.com/yuque/0/2021/png/151680/1622564582018-04eff6fc-5188-4a8c-b9db-399ec38c214f.png#height=57&id=BU0vp&margin=%5Bobject%20Object%5D&name=image.png&originHeight=114&originWidth=1244&originalType=binary&size=16218&status=done&style=none&width=622)
瞬间想到这个是node的环境并不是浏览器环境，没有atob这个函数。so，直接实现一个node base64解码
```javascript
const atob = (str) => {
  return (new Buffer.from(str, 'base64')).toString()
}
```
重新执行一次 输出结果！ m值GET
![image.png](https://cdn.nlark.com/yuque/0/2021/png/151680/1622565241837-2a29c26a-f984-4d29-a119-4b2fa21a5dfd.png#height=58&id=xT3hH&margin=%5Bobject%20Object%5D&name=image.png&originHeight=116&originWidth=804&originalType=binary&size=11896&status=done&style=none&width=402)
#### 第四步 构造请求
接下来就是构造5个请求把获取回来的值算下平均值，然后得到答案！




## 总结


1、学习到反调试技巧
2、学习到格式化代码与信息量收集
3、了解node环境与浏览器环境差异，一些api的缺少后续逆向还是会遇到。