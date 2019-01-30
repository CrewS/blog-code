---
title: webpack（二）模块化
date: 2019-01-05 19:40:46
tags:
    - webpack
    - commonjs
---

##  前言
在上节中，我们已经了解学习webpack的基础知识。在此基础上，我们慢慢深入了解webpack的工作原理。
其中对webpack模块化原理的学习，将是我们深入了解webpack的第一步。
webpack是模块打包器，那其中他是如何在浏览器中实现模块化的呢？带着这样的疑问我们将去了解webpack模块原理


## CommonJS & AMD
#### CommonJS
webpack模块化实现的标准是CommonJS标准，在学习webpack模块化实现原理之前我们先来了解一下什么是CommonJS标准
>CommonJS 规范是为了解决 JavaScript 的作用域问题而定义的模块形式，可以使每个模块它自身的命名空间中执行。该规范的主要内容是，模块必须通过 module.exports 导出对外的变量或接口，通过 require() 来导入其他模块的输出到当前模块作用域中。

CommonJS是同步加载模块，所以一般是用在node服务端。浏览器的资源是异步加载的，所以CommonJS规范并不能直接使用。不过浏览器也相似的实现，是将所有模块定义好建立ID索引，通过ID索引查找相关依赖
#### AMD（Asynchronous Module Definition）规范
AMD规范是采用异步加载模块，模块的加载不影响后续代码的运行。所有以来这些模块的代码都写在回调函数中，等待加载完成后再执行回调函数。
require.js实现了AMD规范，`require([module], callback)`，module是依赖模块的列表，callback是回调函数

## webpack实现commonjs（笔记）
webpack是模块打包器，我们知道webpack能将我们的代码打包成一个js文件。不过具体如何实现，则需要进一步了解
我们将从打包出来的代码分析webpack是如何在浏览器中实现模块化的。
index.js
```js
//index.js
import a from './a';
a.bar()


// a.js

exports.bar = function(){
    return 1
}
```
打包出来的main.js
```js
(function (modules) { // webpackBootstrap
    // The module cache
    var installedModules = {};

    // The require function
    function __webpack_require__(moduleId) {

        // Check if module is in cache
        if (installedModules[moduleId]) {
            return installedModules[moduleId].exports;
        }
        // Create a new module (and put it into the cache)
        var module = installedModules[moduleId] = {
            i: moduleId,
            l: false,
            exports: {}
        };

        // Execute the module function
        modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);

        // Flag the module as loaded
        module.l = true;

        // Return the exports of the module
        return module.exports;
    }
    ....
    return __webpack_require__(__webpack_require__.s = 0);
})
    ({
        "./src/a.js":
            (function (module, exports) {

                eval("\n\nexports.bar = function(){\n    return 1\n}\n\n//# sourceURL=webpack:///./src/a.js?");


            }),
        "./src/index.js":
            (function (module, __webpack_exports__, __webpack_require__) {

                "use strict";
                eval("__webpack_require__.r(__webpack_exports__);\n/* harmony import */ var _a__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__(/*! ./a */ \"./src/a.js\");\n/* harmony import */ var _a__WEBPACK_IMPORTED_MODULE_0___default = /*#__PURE__*/__webpack_require__.n(_a__WEBPACK_IMPORTED_MODULE_0__);\n\n_a__WEBPACK_IMPORTED_MODULE_0___default.a.bar()\n\n//# sourceURL=webpack:///./src/index.js?");


            }),

        0:
            (function (module, exports, __webpack_require__) {

                eval("module.exports = __webpack_require__(/*! ./src/index.js */\"./src/index.js\");\n\n\n//# sourceURL=webpack:///multi_./src/index.js?");
            })

    });
```
去掉不必要的注释，以及省略了一些代码
为了更方便理解,整理结构如下,是个IIFE函数
```js
(function(modules){...主体函数})({
    'a.js':function(module, exports){},
    'index.js':function(module, __webpack_exports__, __webpack_require__){},
    '0': function(module, exports, __webpack_require__){}
})
```
webpack将模块依赖列表生成一个模块对象列表
其中`__webpack_require__`则是模块加载核心
具体分析 `__webpack_require__`函数
```js
// 1、模块缓存对象
var installedModules = {};
// 2、webpack实现的require
function __webpack_require__(moduleId) {
    // 3、判断是否已缓存模块
    if(installedModules[moduleId]) {
        return installedModules[moduleId].exports;
    }
    // 4、缓存模块
    var module = installedModules[moduleId] = {
        i: moduleId,
        l: false,
        exports: {}
    };
    // 5、调用模块函数，递归调用
    modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);
    // 6、标记模块为已加载
    module.l = true;
    // 7、返回module.exports
    return module.exports;
}
// 8、require第一个模块
return __webpack_require__(__webpack_require__.s = 0);
```
经过序号的几个步骤，所有模块加载完毕
模块的调用传入几个参数 `module`, `exports`, `__webpack_require__`
`module`指向了模块本身,`exports`是`module.exports`的引用，`__webpack_require__`是require的实现
就此在模块中实现了CommonJS规范

## webpack处理es6 module（笔记）
第一种es6 module
```js
// a.js
const bar = function(){
    return 1
}
export default  bar
// index.js
import a from './a';
console.log(a)
// main.js
// a module
__webpack_require__.r(__webpack_exports__);
const bar = function(){
    return 1
}
/* harmony default export */ __webpack_exports__["default"] = (bar);
// index module
__webpack_require__.r(__webpack_exports__);
/* harmony import */ var _a__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__(/*! ./a */ "./src/a.js");
console.log(_a__WEBPACK_IMPORTED_MODULE_0__["default"])
```
非es6 mudle情况
```js
// a.js
exports.foo = function () {
    return 1;
}
//index.js
import a from './a';
console.log(a)
// main.js
// a module
exports.foo = function () {
    return 1;
}
// index module
__webpack_require__.r(__webpack_exports__);
/* harmony import */ var _a__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__(/*! ./a */ "./src/a.js");
/* harmony import */ var _a__WEBPACK_IMPORTED_MODULE_0___default = /*#__PURE__*/__webpack_require__.n(_a__WEBPACK_IMPORTED_MODULE_0__);
console.log(_a__WEBPACK_IMPORTED_MODULE_0___default.a)

```

## webpack处理异步模块加载 codespliting（笔记）

## continue

## 参考资料
