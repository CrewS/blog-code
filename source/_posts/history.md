---
title: react-router路由 haschange探讨
date: 2019-11-29 15:08:56
tags:
	- react
	- history
---



## 场景

针对pushState操作完，浏览器后退，以及页面返回时历史页面的处理。

需要了解的知识点

- [PopStateEvent](https://developer.mozilla.org/zh-CN/docs/Web/API/PopStateEvent) 浏览器后退事件
- [History.pushState()](https://developer.mozilla.org/zh-CN/docs/Web/API/History/pushState)
- [HashChangeEvent](https://developer.mozilla.org/en-US/docs/Web/API/HashChangeEvent)
- [History](https://github.com/ReactTraining/history) 前端路由hash history

##### 处理方法

Page A 在跳转Page B前需要保存信息到url中，处理完后返回A，需要将其信息保留下来。

我们处理的方式是，在跳转B前会通过[`History.pushState()`](https://developer.mozilla.org/zh-CN/docs/Web/API/History/pushState),将状态保留在hash值中，又不会触发hashChange导致页面更新。然后从B返回A或者浏览器后退返回A的时候，能够获取到hash中的状态，从而根据状态完成初始化。



##### 问题与现象

页面A /#/a

通过pushState() 修改hash /#/a?v=1，然后浏览器回退按钮点击，页面并无变化。期待是会根据状态重新初始化。

通过pushState()修改hash /#/a?v=1, /#/a?v=2,然后点击浏览器回退按钮，页面会刷新了。

浏览器回退是触发了hashchange事件的，React-router中的hashHistory调用了[History](https://github.com/ReactTraining/history)，通过阅读代码发现能够解释以上问题的原因。

1、history中使用自身的self history记录路由堆栈。但是pushState中并没有触发hashchange方法，也就没有记录该值。所以通过pushState改变的路径后，self history永远记录了/#/a，浏览器后退的时候触发了hashChange进入到逻辑层中

```js
// https://github.com/ReactTraining/history/blob/v4.7.0/modules/createHashHistory.js
const handleHashChange = () => {
    const path = getHashPath()
    const encodedPath = encodePath(path)

    if (path !== encodedPath) {
      // Ensure we always have a properly-encoded hash.
      replaceHashPath(encodedPath)
    } else {
      const location = getDOMLocation()
      const prevLocation = history.location // 堆栈中的loaction,由于pushState改变，所以堆栈中的还是 /#/a

      if (!forceNextPop && locationsAreEqual(prevLocation, location))// 浏览器后退 /#/a 与/#/a对比，一样，所以返回
        return // A hashchange doesn't always == location change.

      if (ignorePath === createPath(location))
        return // Ignore this change; we already setState in push/replace.

      ignorePath = null

      handlePop(location)
    }
  }
```



2、pustate2次的时候反而正常的原因，a =>a1 => a2， a2返回a1的时候，是a1与a对比，不一致所以会调用handlePop 触发组件更新

##### 如何处理这一问题

在pushState前使用[`History.replaceState()`](https://developer.mozilla.org/zh-CN/docs/Web/API/History/replaceState)改变当前的值例如增加_t?new Date()，然后再pushState。这样无论是第一次还是第二次后退的时候永远对比的都是不一样的，从事解决后退无发触发更新的问题。



#### 后记

通过研究history还有pushState，更加了解了前端路由的工作原理。以后遇到浏览器前进后退的问题，也能从容处理。



A => B 前通过pushState更改了hash /#/a?v=1，再跳转到B