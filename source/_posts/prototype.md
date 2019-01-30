---
title: JavaScript原型链与继承
date: 2019-01-25 10:12:35
tags:
    - javascript
---
## 前言
JavaScript的语言特性是每个前端工程师深入学习必须了解的内容。以下是对JavaScript原型、原型链与原型继承的归纳总结。

## 关于原型的几个概念
构造函数、实例对象、原型、__proto__
构造函数：在JavaScript中，构造方法一般命名首字母大写，通过new 命令调用。
实例对象：是通过new 命令调用 构造方法 创建出来的对象。
原型：在javascript中每个函数有个特殊的属性为原型（prototype）。
__proto__： 实例对象中，有个隐藏的属性__proto__, 是从构造函数的prototype属性派生的。每个__proto__属性指向其构造函数的prototype。


## 函数对象与实例对象
实例对象有__proto__属性，函数对象有prototype属性
类似原生的函数例如 Array、String、
同时是函数对象，又是实例对象，这类对象的原型情况分析
Array.__proto__ 指向 Function.prototype [实例对象的proto属性指向其构造函数的原型]
Array.prototype.__proto__ 指向 Object.prototype

**特别的**
Funtion.__proto__ 指向 Funtion.prototype
Object.__proto__ 指向 Funtion.prototype
Function.prototype.__proto__ 指向 Object.prototype
Object.prototype.__proto__ 指向 null

## 类与继承
javascript中是没有直接的类的概念，是通过JavaScript的原型继承实现类与继承的概念，即使是es6中增加的关键字class实际上是语法糖，本质还是原型继承实现的“类”概念。
```js
(function () {
    function Student(name) {
        // this.name = name
    }
    Student.prototype.hello = function () {
        console.log(this.name)
    }
    function F() {

    }
    F.prototype = Student.prototype;
    function P(name) {
        this.name = name;
        Student.call(this);
    }
    P.prototype = new F();
    P.prototype.constructor = P // 修复constructor指向
    p = new P('xiaoming');
    p.hello();
})()
```
在上述的例子中，实现了P类继承了Student类，创建的实例对象p继承类student类中hello方法。通过原型继承的方式实现类别的语言中类继承特性。
可能大家会对F函数的使用产生疑问。
在原型继承中如果不使用代理函数F，采用 new Student()的方式，那么原型链是这样的,P.prototype中会有student构造函数中的一些属性
p => P.prototype => Student.prototype => Object.prototype => null
如果使用代理函数F，采用 new F()的方式，那么P.prototype中则不会有student构造函数中的属性。
p => P.prototype => F.prototype(Student.prototype) => Object.prototype => null

## 原型继承的几种方式

一、**原型链继承**
1、引用类型的数据被所有实例共享，一处改动，影响到所有实例
2、在创建实例时候，无法向父级传递参数。
二、**借用构造函数继承**
1、避免了引用类型的属性被所有实例共享
2、可以在 Child 中向 Parent 传参
缺点：方法的继承需要在构造函数中定义，这样每次创建会重复创建方法。
三、**组合继承**
融合原型链继承和构造函数的优点，是 JavaScript 中最常用的继承模式。
四、**寄生组合式继承**
《JavaScript高级程序设计》: 这种方式的高效率体现它只调用了一次 Parent 构造函数，并且因此避免了在 Parent.prototype 上面创建不必要的、多余的属性。与此同时，原型链还能保持不变；因此，还能够正常使用 instanceof 和 isPrototypeOf。开发人员普遍认为寄生组合式继承是引用类型最理想的继承范式。

## 参考资料
- [JavaScript深入之继承的多种方式和优缺点](https://github.com/mqyqingfeng/Blog/issues/16)
- [MDN原型对象](https://developer.mozilla.org/zh-CN/docs/Learn/JavaScript/Objects/Object_prototypes)