---
title: JavaScript防抖与节流（记录）
date: 2019-01-21 15:55:51
tags:
    - javascript
---

## 前言
在实际的开发过程中，其实我们会经常遇到需要用防抖与节流的场景。比如搜索框连续输入文字进行异步请求的搜索可以用到防抖函数进行优化。

## 防抖
防抖函数，简单的理解为频发触发事件在指定时间内不触发才执行。
在输入框添加了onchange监听事件，只要输入框内容变化就会调用监听事件。
但需求是连续输入只需要查询最后的输入结果，那么给这个监听事件增加防抖即可实现。
具体看代码例子
```js
function ajax(){
    console.log('boom')
}
fn = debounce(ajax, 500);
function debounce(fn,wait){
    var timer = null;
    return function(){
        var context = this;
        var args = arguments;
        clearTimeout(timer)
        timer = setTimeout(() => {
            fn.apply(context,args);
        }, wait);
    }
    
}
document.querySelector('#a').onclick = fn;
```
debounce就是防抖函数。当事件绑定了防抖函数后，触发事件，执行防抖函数，如果timer还在计时器中则被清除，只有在时间范围内不触发事件，才能真正执行该业务逻辑。
## 节流
```js
function ajax(){
    console.log('boom')
}
function throttle (fn, wait){
    var timer = null;
    var pre = 0;
    return function(){
        var context = this;
        var args = arguments;
        var now = +new Date();
        if (now - pre > wait){
            fn.apply(context,arguments);
            pre = now;
        }
    }
}
var fn2 = throttle(ajax, 1000)
document.querySelector('#a').onclick = fn2;
```
节流函数的作用：连续触发事件的时候，只会固定频率执行业务逻辑，如果触发事件的事件间隔少于设定的频率时间，也可以马上执行

## continue
后续遇到具体的业务场景，结合着对简单应用的防抖与节流函数进行“升级”。