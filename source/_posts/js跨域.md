---
title: js跨域
date: 2018-11-02 10:22:46
tags:
    - javascript
---
## 前言
跨域是前端开发经常会遇到的问题，过去的项目中也处理不少。
对这类问题系统的总结尤为必要，无论是以后处理该问题还是对项目架构的思考也有帮助
![](https://cdn.nlark.com/yuque/0/2018/png/151680/1541407751449-2a79a3f1-30bd-445d-bb57-2aeed1f71e12.png)
## 跨域原理
什么是跨域？为什么会出现跨域呢
要了解跨域首先要知道同源策略，同源策略的定义
>如果两个页面的协议，端口（如果有指定）和域名都相同，则两个页面具有相同的源。
>同源策略限制了从同一个源加载的文档或脚本如何与来自另一个源的资源进行交互。这是一个用于隔离潜在恶意文件的重要安全机制。

造成跨域的两种分类
1、dom同源策略：禁止对不同源页面DOM进行操作。这里主要场景是iframe跨域的情况，不同域名的iframe是限制互相访问的。
2、XmlHttpRequest同源策略：禁止使用XHR对象向不同源的服务器地址发起HTTP请求。

## 跨域解决方案
网络请求跨域解决方案
- CORS方法解决跨域
- JSONP跨域请求
- 服务器代理方式解决

dom跨域操作解决方案
- 修改document.domain实现子域不同的页面进行跨域交互
- 在同一窗体下，不同页面中修改window.name的值，能够互相读取

#### 一、CORS
简介：
>MDN:跨域资源共享(CORS) 是一种机制，它使用额外的 HTTP 头来告诉浏览器  让运行在一个 origin (domain) 上的Web应用被准许访问来自不同源服务器上的指定的资源。当一个资源从与该资源本身所在的服务器不同的域或端口请求一个资源时，资源会发起一个跨域 HTTP 请求。

兼容性[查询](https://caniuse.com/#search=cors)
注意【ie(8、9)解决方案】
CORS两种请求：
- 简单请求（simple request）
- 非简单请求（not-so-simple request）


满足以下2大条件的就是简单请求
1、【HEAD,GET,POST】其中一个请求方法
2、HTTP的头部信息不超过以下几种
- Accept
- Accept-Language
- Content-Language
- Last-Event-ID
- Content-Type：只限于三个值application/x-www-form-urlencoded、multipart/form-data、text/plain

基本流程[参考](http://www.ruanyifeng.com/blog/2016/04/cors.html)
简单请求，浏览器会直接发出CORS请求，在请求头信息中增加一个origin字段，服务器根据origin源是否在允许范围内返回不同值，浏览器根据是否返回Access-Control-Allow-Origin字段来做出不同反应
如果没有返回Access-Control-Allow-Origin字段，浏览器会抛出一个错误被XMLHttpRequest的onerror回调函数捕获
非简单请求，会在正式通信之前，增加一次HTTP查询请求，称为“预检”（preflight）
"预检"请求用的请求方法是OPTIONS，表示这个请求是用来询问的。头信息里面，关键字段是Origin，表示请求来自哪个源。通过“预检”，则发出请求，没通过则报错
面试可能会被问到跨域报错的时候发出了请求吗？
具体实践操作
1、服务器设置 Access-Control-相关字段允许跨域，浏览器正常发出xmlhttprequest请求
2、如果需要携带cookies跨域请求，需要服务器设置Credentials, 浏览器请求设置withCredentials为true

#### 二、JSONP的方式
jsonp跨域请求的方式，是利用了同源策略中通常允许跨域资源嵌入（Cross-origin embedding)[点击查看](https://developer.mozilla.org/zh-CN/docs/Web/Security/Same-origin_policy)
简单来说就是动态增加来一个script标签，请求来服务器一段js代码文件
加载完成后执行该代码，在请求中可以增加参数
例子
```html
<script type="text/javascript">
    var functionHandler = function(data){
        console.log(data);
    }
    var url = 'http://xxx.com/xxxx?prams=xxx&callback=functionHandler';
    var script = document.createElement('script');
    script.setAttribute('src', url);
    document.getElementsByTagName('head')[0].appendChild(script); 
</script>
```
服务端代码（简略）
```bash
# 通过参数查询data
functionHandler(data)
```
jsonp限制性：只能get请求

#### 三、服务器代理的方式
在服务器端配置好代理，浏览器端就不会出现跨域的问题
在开发阶段比较常实现
devsever的proxy就是用来该原理
在devsever中配置代理，原指向devserver的请求被代理到目标地址，在服务器中http请求没有跨域限制，所以解决了浏览器js跨域的问题


#### 四、document.domain
document.domain解决了子域名不同页面互相交互的问题，但是也只限制于子域名不一样，端口和协议必须一样
具体操作就是限制document.domain = 顶级域名

#### 五、window.name
window.name则利用同一窗体下加载不同的页面，window.name的值不会清除，达到传递数据的效果 数据大小支持到2MB
具体操作需要3个页面
a 域名下的 origin page
a 域名下的 proxy page
b 域名下的 data page 
a 域名下origin page 通过动态的iframe 加载 data page, data page中设置了window.name = data数据
可是此时 origin page的域名与data page域名不一致，浏览器限制交互，所以需要将iframe跳转到proxy page （即iframe的scr值设置为proxy page）。此时iframe与 origin page同源，可以操作获取到iframe 的window.name中的数据，获取完毕后销毁iframe
这样origin page就可以获取到非同源下的 data page数据
a域名下的 origin page
```html
<script type="text/javascript">
    var a=document.getElementsByTagName("button")[0];
    a.onclick=function(){                               //button添加click事件
        var inf=document.createElement("iframe");       //创建iframe
        inf.src="http://www.b.com/data.html"+"?h=5"  //加载数据页www.b.com/data.html同时携带参数h=5
        var body=document.getElementsByTagName("body")[0];
        body.appendChild(inf);                          //引入a页面

        inf.onload=function(){
            inf.src='http://www.a.com/proxy.html'       //iframe加载完成，加载www.a.com域下边的空白页proxy.html
            console.log(inf.contentWindow.name)        //输出window.name中的数据
            body.removeChild(inf)                      //清除iframe
        }
    }
</script>
```
b域名下 data page
```html
<script>
    // var str=window.location.href.substr(-1,1);      //获取url中携带的参数值
    // 因为已经是b域名下的页面了，可以通过请求各种b域名下的数据再设置window.name的值
    window.name = 'some data'
</script>
```
