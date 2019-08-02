---
title: 微信h5分享自定义设置（一）
date: 2018-11-25 17:09:17
tags:
    - 微信分享
    - h5
    - 移动开发调试
---
# 前言
做过手机端h5页面的前端同学都知道，离不开宣传页面在微信中的传播与分享
今天就这个问题，我们来总结下前端在做h5微信分享的时候应该处理的问题
因为整个微信分享的流程比较长，在“一”中，我们只关注前端将要面临的问题以及怎么处理


# 微信h5分享
前端涉及到的流程处理
 - 判断浏览器ua
 - 动态加载wx-sdk
 - 异步请求后端接口获取signature签名信息（需要将当前url传递给服务器做签名，常见错误附录1提醒）
 - 利用签名配置wx sdk
 - 设置个性化分享信息，图标，标题，描述，链接[logo,title,desc,link]
 - 绑定wx sdk事件
 - debug调试
 - 上线

代码部分
```js
// 1.1判断ua
var ua = navigator.userAgent;
if(/micromessenger/i.test(ua)){
    // 1.2动态加载wx-sdk
    var head = document.getElementsByTagName('head')[0];
    var script = document.createElement('script');
    script.src = '//res2.wx.qq.com/open/js/jweixin-1.4.0.js ';
    head.appendChild(script);
    var url = encodeURIComponent(location.href.split('#')[0]); // 需要将url 编码
    script.onload = function(){
        function bindWxShare(obj) {
            wx.onMenuShareTimeline(obj)
            wx.onMenuShareAppMessage(obj)
            wx.onMenuShareQQ(obj)
            wx.onMenuShareQZone(obj)
        }
        // 1.3 异步获取签名信息
        $.ajax({
            url: '/api?url='+url,
            success: function (response) {
                console.log(response)
                !response.appId && (response = JSON.parse(response));
                // 1.4配置wx-sdk config
                wx.config({
                    // debug: true, // 开启调试模式,调用的所有api的返回值会在客户端alert出来，若要查看传入的参数，可以在pc端打开，参数信息会通过log打出，仅在pc端时才会打印。
                    appId: response.appId,
                    timestamp: response.timestamp, // 必填，生成签名的时间戳
                    nonceStr: response.nonceStr, // 必填，生成签名的随机串
                    signature: response.signature,// 必填，签名，见附录1
                    jsApiList: ['onMenuShareTimeline', 'onMenuShareAppMessage', 'onMenuShareQQ', 'onMenuShareQZone'] // 必填，需要使用的JS接口列表，所有JS接口列表见附录2
                });
                // 1.5 配置个性化设置
                // 确保分享的图片都是设置的图片，而不是微信默认抓取的
                wx.ready(function(){
                    var wxShareImg = 'logo address';
                    bindWxShare({
                        title: '自定义标题',
                        link: 'your link',
                        imgUrl: wxShareImg,
                        desc: '自定义描述'
                    })
                })
            }
        })
    }
    
}
```
通过以上配置基本可以完成对微信分享的功能的实现，介入调试的过程可能是比较麻烦的
因为公众号那边需要有安全域名的设置，只有该域名的及子域名才能通过签名认证
前端这边调试有两种方式
#### 本地调试方式 (适用于在后端提供好接口，前端沙箱开发测试)
前提知识预备
- wx-jssdk安全域名相关知识（只有该域名或者子域名下的页面才能应用wxjssdk）
- devserver porxy代理方式
- charles本地代理
- wx-jssdk 代码调试（错误代码附录一）

**第一步**：创建一个精简的webpack server 修改`hosts`配置好域名`test.example.com`(假定安全域名为example.com) 
```bash
sudo vim /etc/hosts
127.0.0.1 test.example.com
```
**第二步**：根据后端接口的开放程度，如果是允许跨域的接口，无需处理直接异步请求获取即可。如果有跨域限制需要在`webpack.config.js`中配置`devServer`字段，代理转接后端接口
此时本地能够访问 `test.example.com:port`（目标分享页面），同时能够访问目标域名下的后端接口信息
```js
{
    ...
    proxy: {
        '/api': {
            target: 'target.com',
            changeOrigin: true
        }
    }
    ...
}
```
**第三步**：配置Charles代理抓包工具：因为微信分享的测试是在手机端进行的（模拟真实环境，也可以采取微信开发者工具），手机并不能直接访问 `test.example.com:port`。此时需要到charles，在手机上配置好了相关代理信息后，即可手机访问`test.example.com:port`进行真机调试与测试[参考地址](https://www.jianshu.com/p/68684780c1b0)
局限性：通过本机代理的设备才可以访问该页面

#### ngrok开内网穿透技术（进阶方式，方便广泛调试测试，调研功能）
前提知识
- ngrok 本地开映射内网穿透
- 微信公众号测试号配置
- node server 编码签名算法、ACCESS_TOKEN、jsticket获取

思路：就是完全实现一个本地的微信jssdk的调用过程。生成签名，页面异步获取签名，调用wxjssdk
ngrok只是开了一个公网可以访问的域名方便测试，以及符合微信配置里的安全域名设置
优点：前端可以完全控制，能够方便调用wxjssdk各种功能
缺点：需要熟悉node知识，最好有完善的一套工具生成接口，方便开发
ps:第二种方式将在 微信分享自定义设置（二）中体现

# 后续（to be continue）
- 新版本jssdk旧的接口将被遗弃，需要适配低版本微信客户端则要做版本配置
- 补充（二）
- 封装函数，改成一键配置方式

# 参考资料
- [微信公众号测试号地址](https://mp.weixin.qq.com/debug/cgi-bin/sandbox?t=sandbox/login)
- [微信jssdk说明文档](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421141115)
- [webpack devserver 文档](https://webpack.js.org/configuration/dev-server/#devserver-proxy)
- [charles配置](https://www.jianshu.com/p/68684780c1b0)