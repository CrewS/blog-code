---
title: JavaScript执行上下文
date: 2019-01-13 11:12:38
tags:
    - javascript
---

## 前言
笔者在学习JavaScript基础知识的时候，对于一些概念的理解都是看网上前辈的经验总结，但是对于为什么是这样没有过多深入理解，比如变量提升，函数作用域，闭包的原理。今天我们从JavaScript执行上下文开始，从原理出发理解概念。

## 1.什么是JavaScript执行上下文
JavaScript运行代码时环境（三种）
- 全局代码：代码默认运行的环境，最先会进入到全局环境中
- 函数代码：在函数的局部环境中运行的代码
- Eval代码：在Eval()函数中运行的代码

javascript是一个单线程语言，这意味着在浏览器中同时只能做一件事情。当javascript解释器初始执行代码，它首先默认进入全局上下文。每次调用一个函数将会创建一个新的执行上下文。
每次新创建的一个执行上下文会被添加到作用域链的顶部，有时也称为执行或调用栈。浏览器总是运行位于作用域链顶部的当前执行上下文。一旦完成，当前执行上下文将从栈顶被移除并且将控制权归还给之前的执行上下文。

## 2、执行上下文创建的过程
全局执行上下文是在JavaScript代码初始执行时候创建的，所以全局上下文在作用域链的底部。
函数执行下文是函数调用的时候创建的，创建的过程如下
1、创建变量对象vo
2、确定作用域链scope
3、确定this指向

#### 2.1、变量对象vo创建过程
1、建立arguments对象，检查当前上下文中的参数，建立该对象下的属性以及属性
2、检查当前上下文中的函数声明，在vo下建立一个属性，属性值就是指向该函数在内存中的地址的一个引用，如果函数名已经在vo对象下了，那么该值会被新的引用覆盖
3、检查当前上下文的变量声明，在vo下建立对应的属性，并赋值该属性undefined,如果vo下存在该属性，则跳过

从变量对象创建的过程中，我们能够理解到为什么有时候会**变量提升(hoisting)**了。
举个经典的面试题目
```js
function test(){
    console.log(a);
    console.log(b);
    var a = 1;
    function a(){}
    var b= function(){};
    console.log(a)
}
test();
```
```js
// 以下为输出
// ƒ a(){}
// undefined
// 1
```
test函数调用的时候，创建了函数执行上下文，以下为变量对象的创建过程
1、建立arguments对象
2、检索函数声明，找到function a，在vo对象下建立属性a指向function a的地址的引用
3、检索变量声明，找到变量b, 在vo对象下建立属性b,赋值为undefined。
到了代码执行过程
1、console.log(a)，此时变量对象vo下 a的指向是函数地址
2、console.log(b)，此时变量对象vo下 b的值为undefined
3、变量a赋值1，
4、函数地址的引用赋值给b
5、console.log(a)，此时变量对象vo下a的值为1.
这样就解释了变量提升的原理，是由于创建函数执行下文过程中，对函数声明与变量声明处理的不一致造成的结果。

#### 2.2、确定作用域链scope
关于scope作用域链的解释
>The scope chain property of each execution context is simply a collection of the current context's [VO] + all parent’s lexical [VO]s.
Scope = VO + All Parent VOs
Eg: scopeChain = [ [VO] + [VO1] + [VO2] + [VO n+1] ];
译：每一个执行上下文的作用域链属性就是简单的收集当前上下文中的VO（变量对象）和其所有父级上下文中的词法VO。

创建函数的时候，会将创建函数当前执行上下文的scope的引用赋值给函数的属性scope，
在函数调用的时候，会创建函数执行上下文，此时会将函数的scope与当前执行上下文的变量对象vo组合赋值给当前执行上下文的scope，形成当前执行上下文的作用域链scope。
```js
function example (){

}
example()
```
1、在example函数创建的时候，将当前执行上下文的scope保存在函数内部属性[[scope]]中
```js
example.[[scope]] = [
    globalContext.VO
];
```
2、在example函数调用的时候，会创建函数执行上下文和变量对象vo，在确定scope的时候
会将函数[[scope]]属性赋值到执行上下文的scope中
```js
exampleContext = {
    scope: globalContext.VO
}
```
然后会将当前执行上下文的vo压入作用域链顶部，形成当前执行上下文的作用域链
```js
exampleContext = {
    scope: [
        vo,
        globalContext.VO
    ]
}
```
#### 2.3、确定this指针的指向
等待理解补充
参考[avaScript深入之从ECMAScript规范解读this](https://github.com/mqyqingfeng/Blog/issues/7)

## 补充说明
[JavaScript深入变量对象](https://github.com/mqyqingfeng/Blog/issues/5) VO/AO的理解
>未进入执行阶段之前，变量对象(VO)中的属性都不能访问！但是进入执行阶段之后，变量对象(VO)转变为了活动对象(AO)，里面的属性都能被访问了，然后开始进行执行阶段的操作。
它们其实都是同一个对象，只是处于执行上下文的不同生命周期。


## 参考资料
- [JavaScript深入之作用域链](https://github.com/mqyqingfeng/Blog/issues/6)
- [从执行上下文深入理解闭包](https://juejin.im/post/5c257b61e51d451b1c6de48c)
- [理解Javascript之执行上下文(Execution Context)](https://www.cnblogs.com/MinLee/p/5862271.html)