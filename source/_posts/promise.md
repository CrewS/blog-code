---
title: Promise 学习笔记
date: 2019-01-08 15:32:33
tags:
    - promise
    - es6
---
## 前言
Promise是异步编程的一种解决方案，它可以解决异步回调地狱的问题，防止层层嵌套对程序代码带来的难维护性。
在用javascript编码的过程中，我们少不了使用异步回调的时候。学会使用promise解决异步编程问题，将更有效得解决异步问题。
接下来让我们一起学习ES6中的Promise。

## promise对象
>一个 Promise 就是一个代表了异步操作最终完成或者失败的对象。大多数人都在使用由其他函数创建并返回的 Promise。

promise是一个代理对象（代理一个值），一个 Promise有以下几种状态:
- pending: 初始状态，既不是成功，也不是失败状态。
- fulfilled: 意味着操作成功完成。
- rejected: 意味着操作失败。
- 
在应用 Promise 时，我们将会有以下约定：
- 在 JavaScript 事件队列的当前运行完成之前，回调函数永远不会被调用。
- 通过 .then 形式添加的回调函数，甚至都在异步操作完成之后才被添加的函数，都会被调用，如上所示。
- 通过多次调用 .then，可以添加多个回调函数，它们会按照插入顺序并且独立运行。

因此，Promise 最直接的好处就是链式调用。
## 实践
可能纯概念的内容无法理解，举两个例子讲解promise如何解决异步回调问题
定时器是最简单的异步回调问题
```js
// example1:
const clock = function(){
    setTimeout(() => {
        console.log('已经过了5秒钟');
        // callback
    }, 5000);
}
clock()

// example2
const clock = function(){
    return new Promise(function(resolve, reject){
        setTimeout(() => {
            console.log('已经过了5秒钟');
            resolve('执行回调函数');
        }, 5000);
    })
}
clock().then(function(dialog){
    console.log(dialog)
})

```
以上两种方法均是设置定时器为5秒，在指定时间到了以后执行回调函数
只不过第二种方式是用promise的方法解决，可能单纯一层没办法理解当多层调用的时候，会发现promise的方法更加优雅扁平
```js
// example3
const task = function(){
    setTimeout(() => {
        console.log('1');
        setTimeout(()=>{
            console.log('2');
            setTimeout(() => {
                console.log('3')
            }, 3000);
        },3000)
    }, 3000);
}
task();
// example4
const task2 = function(){
    return new Promise((resolve,reject) => {
        setTimeout(() => {
            console.log('1');
            resolve('1')
        }, 3000);
    })
}
task2()
.then((r) => {
     //callback
    return new Promise((resolve,reject) => {
        setTimeout(() => {
            console.log('2');
            resolve('2')
        }, 3000);
    })
})
.then(() => {
     //callback
    return new Promise((resolve,reject) => {
        setTimeout(() => {
            console.log('3');
            // resolve('3')
        }, 3000);
    })
}).then(() => {
    //callback
})
```
exmaple3 是一个典型的多层嵌套的回调函数，当层数越多的时候，代码维护性越差
通过promise的办法，改造多层嵌套为链式调用函数，代码的阅读性和可维护性提高了
其中promise还有其他API满足业务场景需求
- Promise.prototype.catch()
- Promise.all() // 全部执行,若有一个失败的结果，promise会将结果传入失败的回调中，而不管其他promise是否完成
- Promise.race() // 率先改变的promise实例的返回值 传递给 回去
- Promise.resolve() //有时需要将现有对象转为Promise对象 返回状态为resolve 
- Promise.reject() // 返回状态为reject

## async/await与promise结合
async/await是一种特殊的语法可以和promise系统工作，在语意层面更加容易理解代码

**async**关键字放置在函数前,表明该函数是一个异步函数
```js
const example = async function (){
    const p = new Promise((resolve, reject) => {
        setTimeout(() => {
            console.log('2秒后执行')
            resolve('')
        }, 2000);
    })
    await p;
    console.log('await p之后');
}
example()
console.log('expample后')
// expample后
// 2秒后执行
// await p之后
```
可以看出**async**函数是不会阻塞程序的，在**async**函数内部，可以采用await的方式代替promise.then回调方式
用同步的方式书写异步的代码更加直观, `await p`指等待promise对象 p 完成后，再继续执行后面的代码语意也直观

在实际开发的过程中，异步方法经常需要监听成功与失败的结果，这里需要关注 async/await 的方式书写代码的时候
错误捕抓一般使用try catch的方式，能够有效捕获错误。
```js
try {
    await xxx
} catch (e){
    // do somethings
}
```



## 参考资料
- [MDN使用 Promises](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Using_promises)
- [廖雪峰Promise](https://www.liaoxuefeng.com/wiki/001434446689867b27157e896e74d51a89c25cc8b43bdb3000/0014345008539155e93fc16046d4bb7854943814c4f9dc2000)