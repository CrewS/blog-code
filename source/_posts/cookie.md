---
title: cookie与session实现持久化登陆
date: 2019-04-23 15:40:13
tags:
	- cookie
---
## 简单了解session cookie
常用的会话跟踪技术是cookie与session
cookie是存储在客户端，session是存储在server端
可以这么说，cookie是一种补足http协议无状态缺陷的机制
一个cookie的设置分为4步
- 客户端发送http请求
- 服务器响应http请求 set-cookies response
- 客户端发送http请求 包含cookie头部 发送到服务器端
- 服务器返回一个http response 

session 是用服务器来保持状态的。专门为用户开辟存储空间，session被创建后会保存在服务器中，其中session ID则发送给客户端，客户端下次发送请求携带session ID，服务器则会查询该ID 找到对应的session读取信息。
在使用session机制保持持久化登陆的实现上可以将sessionID保存在 cookie、header、URL中，客户端带着sessionID过来，服务器根据这个sessionID判断是否存在当前会话。

## 结合例子说明
session一般结合cookie实现持久化登陆的，其中nodejs中更多的会使用到passport这个库来进行鉴权和持久化登陆。
笔者最近在学习eggjs,则使用egg-passport来说明一下。
egg-passport verify 认证后，会将user信息存储在session中，通过序列化（serializeUser）和反序列化（deserializeUser）对user信息进行加工处理。

```js
app.passport.serializeUser(async (ctx, user) => {
	// do some things
	// 去除敏感信息
	return user;
});
app.passport.deserializeUser(async (ctx, user) => {
	// do some things
	// 通过user部分信息反查数据库得到完整信息
	return user;
});
```

## 深度解读egg/koa中 session的存储原理
根据概念知道，session是存储在服务器中的，但是egg中的实现比较“神奇”，因为和概念似乎有点出入，导致笔者找了许久都找不到一个存储session的地方。最后通过cnode中提问，得到线索与cookie相关，然后重新阅读源码发现整个流程原来是这样的。服务器存储session到centext中，同时设置cookie为session的值，然后再次请求的时候，解析cookie，将值初始化为session
我们在《passport源码阅读》那篇博客中已经知道，在verify验证完成后，会执行ctx.req.session = user。将用户信息保存在session中，然后这个session其实是koa-session中间件来的。通过阅读koa-session的代码。知道session通过加密方式存储在cookie中，最后用户再次请求的时候，会将携带的cookie解析为ctx.session内容。这就是egg session的工作原理。

请看关键代码
```js
//egg-passport/lib/framework.js
function initialize(passport) {
  // https://github.com/jaredhanson/passport/blob/master/lib/middleware/initialize.js
  return function passportInitialize(ctx, next) {
    const req = ctx.req;
    req._passport = {
      instance: passport,
    };
    // ref to ctx
		req.ctx = ctx;
		// 将ctx的session挂载到req.session中
    req.session = ctx.session;
    req.query = ctx.query;
    req.body = ctx.request.body;

    if (req.session && req.session[passport._key]) {
      // load data from existing session
      req._passport.session = req.session[passport._key];
    }

    return next();
  };
}
```
koa-session 通过defineProperties，对session的读取操作进行监听执行事件处理
```js
// koa-session/index.js
function extendContext(context, opts) {
  if (context.hasOwnProperty(CONTEXT_SESSION)) {
    return;
  }
  Object.defineProperties(context, {
    [CONTEXT_SESSION]: {
      get() {
        if (this[_CONTEXT_SESSION]) return this[_CONTEXT_SESSION];
        this[_CONTEXT_SESSION] = new ContextSession(this, opts);
        return this[_CONTEXT_SESSION];
      },
    },
    session: {
      get() {
        return this[CONTEXT_SESSION].get();
      },
      set(val) {
        this[CONTEXT_SESSION].set(val);
      },
      configurable: true,
    },
    sessionOptions: {
      get() {
        return this[CONTEXT_SESSION].opts;
      },
    },
  });
}
```
这个是对session保存在cookie的save方法，是commit调用的时候触发save方法
```js
// koa-session/lib/context.js
async save(changed) {
    const opts = this.opts;
    const key = opts.key;
    const externalKey = this.externalKey;
    let json = this.session.toJSON();
    // set expire for check
    let maxAge = opts.maxAge ? opts.maxAge : ONE_DAY;
    if (maxAge === 'session') {
      // do not set _expire in json if maxAge is set to 'session'
      // also delete maxAge from options
      opts.maxAge = undefined;
      json._session = true;
    } else {
      // set expire for check
      json._expire = maxAge + Date.now();
      json._maxAge = maxAge;
    }

    // save to external store
    if (externalKey) {
      debug('save %j to external key %s', json, externalKey);
      if (typeof maxAge === 'number') {
        // ensure store expired after cookie
        maxAge += 10000;
      }
      await this.store.set(externalKey, json, maxAge, {
        changed,
        rolling: opts.rolling,
      });
      if (opts.externalKey) {
        opts.externalKey.set(this.ctx, externalKey);
      } else {
        this.ctx.cookies.set(key, externalKey, opts);
      }
      return;
    }

    // save to cookie
    debug('save %j to cookie', json);
    json = opts.encode(json);
    debug('save %s', json);

    this.ctx.cookies.set(key, json, opts);
  }
```
调用commit的代码
```js
module.exports = function(opts, app) {
  // session(app[, opts])
  if (opts && typeof opts.use === 'function') {
    [ app, opts ] = [ opts, app ];
  }
  // app required
  if (!app || typeof app.use !== 'function') {
    throw new TypeError('app instance required: `session(opts, app)`');
  }

  opts = formatOpts(opts);
  extendContext(app.context, opts);

  return async function session(ctx, next) {
    const sess = ctx[CONTEXT_SESSION];
    if (sess.store) await sess.initFromExternal();
    try {
    // 这里需要理解 koa的洋葱圈模型中间件执行顺序
      await next();
    } catch (err) {
      throw err;
    } finally {
      if (opts.autoCommit) {
        await sess.commit();
      }
    }
  };
};
```
每个请求走过中间件最后会commit一次，就是更新cookie为最新的session内容。

## 后记
深入了解node的session存储机制的过程中又阅读了几个插件的源码内容，收获还是比较丰富的。
记得一位博主说过，带着问题阅读源码是读源码的最好方式。通过几个问题，可能就可以由点到面的理解源码的代码和思想。
比较深入的理解一些代码带来的成就感还是挺棒的，小伙伴们也可以带着问题慢慢去看源码，保证有收获。


## 参考
- [passport](https://github.com/jaredhanson/passport)
- [聊一聊session和cookie](https://juejin.im/post/5aede266f265da0ba266e0ef)
- [koa-session](https://github.com/koajs/session)
- [egg](https://github.com/eggjs/egg)