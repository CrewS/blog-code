---
title: JS基础知识补给包
date: 2022-03-6 11:00:36
tags: [js逆向, 爬虫]

index_img: /img/post/js-base.png
banner_img: /img/bg/banner.webp
---

## 前言

在逆向一些网站的时候，扣出来JS代码，能运行但是不知道为啥能这样跑。相信很多小伙伴有这样的疑惑，本篇文章主要讲解JavaScript里几个基础的知识点，帮助大家更好理解js代码。
下面是**主要内容**，已经了解的同学可以不用看啦~

- JavaScript原型链
- 函数作用域


## JavaScript 原型链

直接讲述概念可能会特别的抽象，这里举个例子JavaScript原型的使用场景

```js
function Person(name, age){ 
    this.name = name;
    this.age = age;
}

Person.prototype.skill = "呼吸"

let xiaoming = new Person("小明", 18)

console.log(xiaoming)
```

![简单原型示意图](https://crews-note-1253247308.cos.ap-guangzhou.myqcloud.com/note/image-20220306112021201.png)

可以看到我们创建了一个Person的函数（类），然后内部有2个属性`name`，`age`，后面再给这个Person的原型上添加`skill=呼吸` ，最后通过Person创建出来的对象“小明”，除了具备初始传参进去的“名字“和“年龄“，还有一个通用的技能 “呼吸”。假设没有原型，如果要给每个通过Person创建出来的对象添加“呼吸”这个技能就会特别麻烦。不过通过console输出这个对象，会发现 skill这个属性是挂在 `__proto__`上的，这也是javascript 原型链的特点，

> JavaScript 对象有一个指向一个原型对象的链。当试图访问一个对象的属性时，它不仅仅在该对象上搜寻，还会搜寻该对象的原型，以及该对象的原型的原型，依次层层向上搜索，直到找到一个名字匹配的属性或到达原型链的末尾。

通过原型这一特点，也让JavaScript实现了“类”的继承、封装。

JavaScript原型链还有几个比较重要的概念 (比较容易混淆)

`__proto__`:  属性__proto__是一个对象，它有两个属性，constructor和__proto__

`prototype`，`constructor` 原型对象prototype有一个默认的constructor属性，用于记录实例是由哪个构造函数创建

这个是原型链比较著名的一张图

比较特殊的是 **Object**、**Funtion** 这2个原生函数

`Object.__proto__.__proto`__ 指向null

`Funtion.__proto__` 指向本身的原型Funtion.prototype

![javascript原型、原型链神图](https://crews-note-1253247308.cos.ap-guangzhou.myqcloud.com/note/1183435-20170915105226032-1488063174.jpg)





## 作用域

要理解JavaScript作用域，先得理解什么是**作用域**

> 作用域产生于程序源代码中定义变量的区域，在程序编码阶段就确定了。JavaScript 中分为全局作用域(Global context： `window`/`global` )和局部作用域（Local Scope , 又称为函数作用域 Function context）。简单讲作用域就是当前函数的`生成环境`或者`上下文`，包含了当前函数内定义的变量以及对外层作用域的引用。

JavaScript里作用域的类型可分为几种

- window/global Scope 全局作用域
- function Scope 函数作用域
- Block Scope 块级作用域
- eval Scope eval作用域



**全局作用域**

在代码中任何地方都能访问到的对象拥有全局作用域，一般来说以下几种情形拥有全局作用域

- 最外层函数和在最外层函数外面定义的变量拥有全局作用域
- 所有末定义直接赋值的变量自动声明为拥有全局作用域
- 所有 window 对象的属性拥有全局作用域



**函数作用域**

是指声明在函数内部的变量，和全局作用域相反，局部作用域一般只在特定的代码片段内可访问到，最常见的例如函数内部。

```js
var val = 1 // 全局变量
function init() {
  val2 = 222 // 未定义的变量直接复制默认是全局变量
  var val3 = 333
}
init()
function test() {
	console.log(val)  // 1
  console.log(val2) // 2
  // console.log(val3) // val3属于init函数的内部变量，如果这里使用val3 运行会提示 ReferenceError: val3 is not defined
  // test函数的函数作用域内没有val3，顺着作用域链寻找val3也找不到该变量的定义
}
test()
```

![demo](https://crews-note-1253247308.cos.ap-guangzhou.myqcloud.com/note/image-20220306124038993.png)



#### 闭包

> 一个函数和对其周围状态（**lexical environment，词法环境**）的引用捆绑在一起（或者说函数被引用包围），这样的组合就是**闭包**（**closure**）。也就是说，闭包让你可以在一个内层函数中访问到其外层函数的作用域。在 JavaScript 中，每当创建一个函数，闭包就会在函数创建的同时被创建出来。

```js
function makeAdder(x) {
  return function(y) {
    return x + y;
  };
}

var add5 = makeAdder(5);
var add10 = makeAdder(10);

console.log(add5(2));  // 7
console.log(add10(2)); // 12
```



看下这个例子，就可以发现add5，add10 这2个函数中的x不一样，是由函数创建时候的x的值确定的。

闭包的运用通常是用来存储一些不希望外部直接改变的变量，或者是一些缓存信息

比如常见在初始化过程中或者一些函数执行过程中生成的函数内部具有一些变量，而这些变量的值在是由函数创建时候确定的，所以逆向的时候需要找到该函数创建时候的堆栈

```js
// 简单的例子
init(key) {
  return function encode(word) {
     // key 是闭包中的变量
   	 return key + word
	}
}
var oneEncodeFn = init('hello')

```



## 后续

JavaScript基础知识补给包送到大家手中， 后面一篇将会讲浏览器环境与Node环境的区别，再带大家一起实践下如何补环境。

