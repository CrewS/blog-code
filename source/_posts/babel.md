---
title: 初探babel与babel插件
date: 2019-01-03 15:05:05
tags:
    - babel
    - javascript
---
## 前言
在使用es6编码开发的时候，或多或少都会遇到浏览器兼容的问题
Babel就是解决这个问题的
但是在使用的时候却发现有Babel-polyfill,有Babel-transform-runtime等等，一下子分不清他们的区别是什么
今天的主题就是了解一下他们的区别以及我们该如何去配置

## 1.babel
babel可以理解为javascript的编译器，更确切地说是源码到源码的编译器，通常也叫做“转换编译器“
babel 的三个主要处理步骤分别是：解析（parse），转换（transform），生成（generate）。
在转换的过程中，babel默认只对javascript的语法进行处理，并不会处理转换新的API
如果需要兼容低版本浏览器使用新的API则需要用 babel-polyfill或者babel-transform-runtime等插件介入转换过程处理js代码

## 2.babel-polyfill与babel-transform-runtime 对比
**babel-polyfill **是当前环境注入这些 es6+ 标准的垫片
优点：一次引用，不再担心兼容问题适合在大型项目中使用
缺点：因为是全局变量覆盖的方式，会污染原生的方法

**babel-transform-runtime** 则是识别并替换代码中的新特性，按需替换
优点：因为是按需替换，体积小，而且不会污染全局变量适合在开发工具中使用
缺点：一些新增的实例方法是没有处理的例如数组的 includes, filter, fill 等

## 3.实验
1、在webpack中采用引用 `@babel/polyfill` 
体积416kb，在IE11下正常运行,且在控制台中输入Promise、Object.assign均正常
2、不引入polyfill，在.babelrc中配置 `@babel/transform-runtime`，默认配置corejs:false
体积38.3kb，在IE11下报错 promise未定义
3、在.babelrc中配置 `@babel/transform-runtime`，设置corejs:2
体积140kb，在IE11下正常运行，控制台输入Promise报错

## 4.webpack中实践配置
```bash
npm install babel-loader @babel/core  @babel/preset-env @babel/plugin-transform-runtime
npm install @babel/runtime-corejs2
npm install @babel/polyfill
```
安装好babel相关依赖后进行配置
在webpack.config.js中 配置loader
```js
module: {
    rules: [
        {
            test: /\.js$/,
            exclude: /node_modules/,
            use: {
                loader: 'babel-loader'
            }
            // test 符合此正则规则的文件，运用 loader 去进行处理，除了exclude 中指定的内容
        }
    ]
}
```
在.babelrc中配置 transform-runtime方式处理JavaScript代码
```json
{
    "presets": [
        [
            "@babel/preset-env",
            {
                "targets": [
                    "last 1 version",
                    "> 2%",
                    "IE >= 9",
                    "not dead"
                ],
                "useBuiltIns": false
            }
        ]
    ],
    "plugins": [
        [
            "@babel/transform-runtime",
            {
                "corejs": 2,
                "helpers": true,
                "regenerator": true,
                "useESModules": false
            }
        ]
    ]
}
```
如果想要引入babel-polyfill的方式,有两种方式
如果是按需引用的话，设置属性 "useBuiltIns": "usage" 即可
如果是全局引用 "useBuiltIns": "entry"，且在入口文件最顶部 `import "@babel/polyfill"`
```json
{
    "presets": [
        [
            "@babel/preset-env",
            {
                "targets": [
                    "last 1 version",
                    "> 2%",
                    "IE >= 9",
                    "not dead"
                ],
                "useBuiltIns": "usage"
            }
        ]
    ]
}
```
## 补充
babel-polyfill是不支持fetch的，所以对fetch兼容的话需要额外添加polyfill

## 参考资料
参考资料均为参考，实验用到的环境是Babel7.0+，相关插件配置可能有变动
- [你真的会用Babel吗](https://github.com/sunyongjian/blog/issues/30)
- [babel官方文档](https://babeljs.io/docs/en/)
- [关于babel-polyfill和babel-runtime](https://juejin.im/post/5b2cc31f51882574d02facff)
