---
title: passport源码阅读
date: 2019-04-11 19:28:23
tags:
	- nodejs
---

## passport是如何将用户信息注入到context中

## 前言

今天在阅读egg-conde中的本地验证的代码时候，发现有些地方不是特别理解。就尝试着阅读源码去寻找问题的根源。具体问题是: 在通过passport验证后，egg-cnode是直接可以在controller中使用ctx.user获取用户的身份信息，但是过程中是看不到这一字段产生的过程的，文档中好像也没有说明。

## 具体解析

涉及到几个npm 包

- passport
- passport-local
- egg-passport

先给出代码

```js
// app.ts
import { Application } from 'egg';
const LocalStrategy = require('passport-local').Strategy;

module.exports = (app: Application) => {
  const config = app.config.passportLocal;
  config.passReqToCallback = true;
  app.passport.verify(async (ctx, user) => {
    ctx.logger.debug('passport.verify', user);
        // 查数据库
    const result = await ctx.service.user.find({username: user.username, password: user.password});
    if (result.length > 0) {
      return result[0];
    }
    return false;
  });
  app.passport.use(
    new LocalStrategy(config, (req, username, password, done) => {
      // 把 Passport 插件返回的数据进行清洗处理，返回 User 对象
      const user = {
        provider: 'local',
        username,
        password,
      };
      // 这里不处理应用层逻辑，传给 app.passport.verify 统一处理
      app.passport.doVerify(req, user, done);
    }),
  );
};

```

```js
// controller/admin.ts
import { Controller } from 'egg';

export default class AdminController extends Controller {
  public async index() {
    const { ctx } = this;
    if (ctx.isAuthenticated()) {
      // show user info
      ctx.body = ctx.user;
      // await ctx.render('admin.tpl');
    } else {
      // redirect to origin url by ctx.session.returnTo
      ctx.session.returnTo = ctx.path;
      await ctx.render('login.tpl');
    }
  }
}
```

```js
// router.ts
export default (app: Application) => {
  const { controller, router } = app;
//...
  router.post('/auth', app.passport.authenticate('local', { successRedirect: '/admin',failureRedirect: '/login', }));
};

```

当通过验证后，控制器中可以直接获取到数据库查出的信息。那么明显是从`app.passport.verify`函数中返回来的但是具体过程是如何的呢？我们并不清楚。此时需要关注到下面的代码
egg-passport是继承passort的，所以app.passport拥有egg-passport的方法和passport的方法

```js
// node_modules/egg-passport/lib/passport.js
const debug = require('debug')('egg-passport:passport');
const Passport = require('passport').Passport;
const SessionStrategy = require('passport').strategies.SessionStrategy;
const framework = require('./framework');

class EggPassport extends Passport {
// ....
	doVerify(req, user, done) {
    const hooks = this._verifyHooks;
    if (hooks.length === 0) return done(null, user);
    (async () => {
      const ctx = req.ctx;
      for (const handler of hooks) {
        user = await handler(ctx, user);
        if (!user) {
          break;
        }
      }
      done(null, user);
    })().catch(done);
  }
	verify(handler) {
    this._verifyHooks.push(this.app.toAsyncFunction(handler));
  }
// authenticate(){...}
}
```

可以看到verify函数只是doverify主体函数，成功后回调执行`done(null,user)`
done方法是从哪里传递过来的呢？查看代码app.ts 43行
`new LocalStrategy(config, (req, username, password, done) => {`
执行new Strategy的时候的回调函数中传递的，此时需要查看Strategy的源码。

```js
// node_modules/passport-local/lib/strategy.js
function Strategy(options, verify) {
  if (typeof options == 'function') {
    verify = options;
    options = {};
  }
  if (!verify) { throw new TypeError('LocalStrategy requires a verify callback'); }
  
  this._usernameField = options.usernameField || 'username';
  this._passwordField = options.passwordField || 'password';
  
  passport.Strategy.call(this);
  this.name = 'local';
  this._verify = verify;
  this._passReqToCallback = options.passReqToCallback;
}
Strategy.prototype.authenticate = function(req, options) {
	//...
	function verified(err, user, info) {
    if (err) { return self.error(err); }
    if (!user) { return self.fail(info); }
    self.success(user, info);
  }
	try {
    if (self._passReqToCallback) {
      this._verify(req, username, password, verified);
    } else {
      this._verify(username, password, verified);
    }
  } catch (ex) {
    return self.error(ex);
  }
}
```

从代码中 verify赋值给_verify 与 `this._verify(req, username, password, verified)`,可以看出
`new LocalStrategy`的回调函数就是这里执行的，然后done方法就是这里的verified方法。
具体查看verified方法，可以看到`self.success(user, info)`，将user的信息传递。但是突然多出来的success方法是从哪里出现呢。
这里笔者迷惑来比较久，并没有在strategy文件中发现该方法的定义。然后观察代码，发现app.passport.use这个方法，然后阅读passport的源码，最终发现success方法是passport的中间件里给strategy添加的一个方法。（不太理解为什么作者要这么处理）

```js
// node_modules/passport/lib/middleware/authenticate.js
//...
strategy.success = function(user, info) {
// ...
	req.logIn(user, options, function(err) {
		if (err) { return next(err); }
		
		function complete() {
			if (options.successReturnToOrRedirect) {
				var url = options.successReturnToOrRedirect;
				if (req.session && req.session.returnTo) {
					url = req.session.returnTo;
					delete req.session.returnTo;
				}
				return res.redirect(url);
			}
			if (options.successRedirect) {
				return res.redirect(options.successRedirect);
			}
			next();
		}
		
		if (options.authInfo !== false) {
			passport.transformAuthInfo(info, req, function(err, tinfo) {
				if (err) { return next(err); }
				req.authInfo = tinfo;
				complete();
			});
		} else {
			complete();
		}
	});
};
//...
```

可以看到success方法中，是通过req.logIn方法继续将user信息传递下去的，
而req.logIn方法则是在以下代码中增加的方法。

```js
// node_modules/passport/lib/http/request.js
req.login =
req.logIn = function(user, options, done) {
  if (typeof options == 'function') {
    done = options;
    options = {};
  }
  options = options || {};
  
  var property = 'user';
  if (this._passport && this._passport.instance) {
    property = this._passport.instance._userProperty || 'user';
  }
  var session = (options.session === undefined) ? true : options.session;
  
  this[property] = user;
  if (session) {
    if (!this._passport) { throw new Error('passport.initialize() middleware not in use'); }
    if (typeof done != 'function') { throw new Error('req#login requires a callback function'); }
    var self = this;
    this._passport.instance.serializeUser(user, this, function(err, obj) {
      if (err) { self[property] = null; return done(err); }
      if (!self._passport.session) {
        self._passport.session = {};
      }
      self._passport.session.user = obj;
      if (!self.session) {
        self.session = {};
      }
      self.session[self._passport.instance._key] = self._passport.session;
      done();
    });
  } else {
    done && done();
  }
};
```

可以看到最终是req.logIn中`this[property] = user;`完成来用户信息的赋值
egg context对象是继承koa的contetxt,而koa的contetx对象是将node request和response对象封装到按个对象中去的。
所以最后ctx.user能够获取到用户信息。到此能够理解用户信息在整个过程中的流动。
以上还是忽略一些步骤，中间的启用过程

```js
// node_modules/passport/lib/authenticator.js
//...
Authenticator.prototype.init = function() {
  this.framework(require('./framework/connect')());
  this.use(new SessionStrategy());
};
//...
```

```js
// node_modules/passport/lib/framework/connect.js
var initialize = require('../middleware/initialize')
  , authenticate = require('../middleware/authenticate');
exports = module.exports = function() {
  
  // HTTP extensions.
  exports.__monkeypatchNode();
  
  return {
    initialize: initialize,
    authenticate: authenticate
  };
};
exports.__monkeypatchNode = function() {
  var http = require('http');
  var IncomingMessageExt = require('../http/request');
  http.IncomingMessage.prototype.login =
  http.IncomingMessage.prototype.logIn = IncomingMessageExt.logIn;
  http.IncomingMessage.prototype.logout =
  http.IncomingMessage.prototype.logOut = IncomingMessageExt.logOut;
  http.IncomingMessage.prototype.isAuthenticated = IncomingMessageExt.isAuthenticated;
  http.IncomingMessage.prototype.isUnauthenticated = IncomingMessageExt.isUnauthenticated;
};
```

Authenticator 初始化的时候，完成了req方法的添加，与中间件方法的搭载。
以上是对passport验证后，用户信息注入到context流程源码分析

## 参考资料

- [passport仓库仓库代码](https://github.com/jaredhanson/passport)
- [passport-local仓库代码](https://github.com/jaredhanson/passport-local)
- [egg 鉴权demo](https://eggjs.org/zh-cn/tutorials/passport.html)