---
title: webpack（一）基础知识
date: 2018-12-28 16:58:10
tags:
    - webpack
---
## 前言
前端工程流越来越被大家所了解使用后，webpack也是目前各位前端工程师所必备的技能了
但是webapck所涵盖的内容又太多了，所以对于日常使用来说，我们优先掌握基础知识和优化策略即可

## webpack基础知识
1、webpack是什么？
官网文档的解释
>本质上，webpack 是一个现代 JavaScript 应用程序的静态模块打包器(static module bundler)。在 webpack 处理应用程序时，它会在内部创建一个依赖图(dependency graph)，用于映射到项目需要的每个模块，然后将所有这些依赖生成到一个或多个bundle。


简单来说webpack就是模块打包器

2、webpack的核心概念
- entry：入口文件，在webpack中可以定义一个或者多个入口。webpack启动后会跟据入口文件创建依赖图
- output：输出文件，可在webpack中配置输出的文件名称以及文件的输出目录
- loader：模块转换和预处理器：可以在加载时处理文件，也可以将不同问文件转化未JavaScript模块
- plugins：webpack插件：可以解决loader无法解决的问题

3、webpack常用的loader与插件
**loader** 
- 样式：style-loader、css-loader、less-loader、sass-loader等
- 文件：raw-loader、file-loader 、url-loader等
- 编译：babel-loader、coffee-loader 、ts-loader等
- 校验测试：mocha-loader、jshint-loader 、eslint-loader等

**plugin**
- UglifyJsPlugin: 压缩和混淆代码。
- CommonsChunkPlugin: 提高打包效率，将第三方库和业务代码分开打包.
- ProvidePlugin: 自动加载模块，代替require和import
- html-webpack-plugin: 可以根据模板自动生成html代码，并自动引用css和js文件。
- extract-text-webpack-plugin: 将js文件中引用的样式单独抽离成css文件
- DefinePlugin:  编译时配置全局变量。
- HotModuleReplacementPlugin： 热更新。
- compression-webpack-plugin：生产环境可采用gzip压缩JS和CSS。

## 参考资料
- [wepack中文文档](https://webpack.docschina.org/concepts/)