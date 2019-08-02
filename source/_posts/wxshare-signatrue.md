---
title: 微信h5分享自定义设置（二）
tags:
  - 微信分享
---

## 前言

一般来说，前端开发者只需要完成微信分享一的实现即可。但是总有些时候，前端开发需要做更多的事情
继微信分享（一），这次将用 egg 实现后端签名算法。

## 快速搭建环境

第一步： 配置开发环境
[参考 egg 的官方文档](https://eggjs.org/zh-cn/intro/quickstart.html)，快速搭建一个 egg 环境

```bash
$ mkdir egg-example && cd egg-example
$ npm init egg --type=ts
$ npm i
$ npm run dev
```

配置 egg-chache (缓存), egg-view-nunjucks（模版引擎）。

第二部：配置 ngrok(内网穿透，让微信服务器访问获取设置安全域名)
[搭建教程](http://www.imooc.com/article/79754)

##

验证服务器 token

```js
// router.js
router.get("/api/wx", controller.api.wx.get);
```

```js
//  controller/api/wx.ts
const getSha1 = function(str) {
  const sha1 = crypto.createHash('sha1'); // 定义加密方式:md5不可逆,此处的md5可以换成任意hash加密的方法名称；
  sha1.update(str);
  const res = sha1.digest('hex'); // 加密后的值d
  return res;
};
//
export default class WxController extends Controller {
  public async get() {
      const { ctx } = this;
      const { signature, timestamp, nonce, echostr } = ctx.query;
      const token = 'wxjsdk';
      const list = [ token, timestamp, nonce ];
      list.sort();
      const hashcode = getSha1(list.join(''));
      if (signature === hashcode) {
        ctx.body = echostr;
      } else {
        ctx.body = '';
      }
  }
}
```

具体分享页面请求获取 signature 算法

```js
//  controller/api/wx.ts

// 随机字符串
var createNonceStr = function () {
  return Math.random().toString(36).substr(2, 15);
};

// 时间戳
var createTimestamp = function () {
  return parseInt((new Date().getTime() / 1000).toString()).toString();
};
export default class WxController extends Controller {
  // 内部函数
  private async getSignature(){
    // 从缓存中获取 token和jsticket 判断是否过期
    const token = await this.app.cache.get('token');
    const jsapiTicket = await this.app.cache.get('jsapiTicket');
    if (token && jsapiTicket) {
      return jsapiTicket;
    } else {
      // 分步获取token和jsticket
      // apppid和appsecret从微信公众号后台设置里获取
      const appid = 'your appid';
      const appsecret = 'your appsecret';
      const result = await this.app.curl(
        `https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid=${appid}&secret=${appsecret}`,
        {
          dataType: 'json',
        },
      );
      if (result.data && result.data.access_token) {
        const access_token = result.data.access_token;
        await this.app.cache.set('token', access_token, 7200);
        const jsapi = await this.app.curl(
          `https://api.weixin.qq.com/cgi-bin/ticket/getticket?access_token=${
            access_token
          }&type=jsapi`,
          {
            dataType: 'json',
          },
        );
        if (jsapi.data && jsapi.data.errcode === 0) {
          const ticket =  jsapi.data.ticket
          await this.app.cache.set('jsapiTicket', ticket, 7200);
          return ticket
        }
      }
    }
    return false
  }
  // 接口请求控制
  public async sg() {
    const { ctx } = this;
    const jsticket = await this.getSignature();
    console.log(jsticket)
    if (!jsticket) {
      ctx.body = {
        errmsg: '出错了',
      };
    }
    const { url } = ctx.query;
    const nonceStr = createNonceStr();
    const timestamp = createTimestamp();
    const str = `jsapi_ticket=${jsticket}&noncestr=${nonceStr}&timestamp=${timestamp}&url=${url}`
    const signature = getSha1(str);
    ctx.body = {
      appId: 'your appid',
      timestamp,
      nonceStr,
      signature,
    }
  }
}
```

设置请求路由

```js
// router.js
router.get("/api/wxsg", controller.api.wx.sg);
```

此上完成签名算法的接口请求
配合微信分享（一）中的前端代码，能够实现分享自定义

## 注意事项

一、设置接口配置信息的时候，微信服务器会主动访问自己的服务器（所以需要具备自己的服务器或者用 ngrok 配置内网穿透）完成验证
二、token 和 jsticket 都是 7200 秒的有效期且不能频繁访问获取，所以需要缓存存起来，需要用户配置相关策略
三、调试接口和网页可以使用微信开发者工具，具备更详细的 log 信息

## 参考资料

- [微信公众号配置地址](https://mp.weixin.qq.com/debug/cgi-bin/sandbox?t=sandbox/login)
- [微信公众号签名算法](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421141115)
- [ngrok 配置地址](http://www.imooc.com/article/79754)
- [nodejs 微信 jsdk](https://www.jianshu.com/p/1a35e1dbe1ad)
