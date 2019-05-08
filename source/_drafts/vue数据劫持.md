---
title: vue数据劫持
date: 2019-02-20 12:15:14
tags:
    - vuejs
    - javascript
---

## 前言

## defineProperty
defineProperty是vue数据劫持的核心原理，那么来看一下defineProperty的特性。
`Object.defineProperty(obj, prop, descriptor)`
- configurable  定义属性是否能被删除，默认为false
- enumerable    定义属性是否能被枚举，默认为false
- value         定义属性的值，默认是undefined
- writable      属性是否能被赋值符改变，默认是false
- get           给属性提供一个getter方法，属性访问时候会触发
- set           给属性提供一个setter方法，属性设置时候会触发

通过其中get、set定义两个方法可以监听属性的变化可以达到数据劫持的效果。
vue源码中实现数据劫持主要是通过几个方面实现的。
1、数据的监听器
2、依赖收集器
3、watcher观察者




