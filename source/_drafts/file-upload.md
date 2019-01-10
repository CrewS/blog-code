---
title: 文件上传的两三事
date: 2018-10-24 10:49:59
tags:
    - javascript
    - ajax
---
## 1.文件上传
web开发都会遇到的问题，文件上传
虽然现有许多成熟的框架能够实现文件上传的功能。不过内在的原理还是有必要去了解一下的。
#### 1.1 表单上传文件
普通的表单上传文件是action同步提交表单，没有办法实现页面不刷新
#### 1.2 ajax异步上传文件 formdata
>XMLHttpRequest Level 2添加了一个新的接口FormData.利用FormData对象,我们可以通过JavaScript用一些键值对来模拟一系列表单控件,我们还可以使用XMLHttpRequest的send()方法来异步的提交这个"表单".比起普通的ajax,使用FormData的最大优点就是我们可以异步上传一个二进制文件.

- 具体使用例子
```js

```
- 兼容性问题具体查看[caniuse](https://caniuse.com/#search=formData)
PC端 IE兼容性问题,ie10以下不支持
移动端 低版本的手机浏览器不支持
- 兼容性问题解决方案
js-form 是jquery的一个插件 example

#### 1.3 文件上传服务升级之：oss上传
- 查看源码分析

## 2.ajax

## 3.post请求的content-type部分
- application/x-www-form-urlencoded 表单提交（数组怎么处理？）
- multipart/form-data
- application/json
- text/xml