---
title: Rollup原理分析（一）基础使用
date: 2022-01-09 14:22:08
tags: [rollup, web打包工具系列]

index_img: /img/post/rollup.jpeg
banner_img: /img/bg/banner.webp
---

## 前言

当前分析的Rollup版本：`2.63.0`

打包工具作为前端最基础的工具之一，想必大家或多或少都会对其实现原理感兴趣。但是由于网上的资料参差不齐，学习起来非常困难，容易劝退。
笔者想要通过`web打包工具分析`系列文章，揭秘web前端打包工具的实现原理，降低大家学习框架源码的门槛。

Rollup源码分析系列将会拆分成以下几部分

- [x] Rollup的简单使用

- [ ] Rollup的插件使用

- [ ] Rollup的打包过程

- [ ] Rollup插件机制

- [ ] 如何编写一个插件（实战）

- [ ] 分析几个主流Rollup插件源码


## Rollup是什么？

首先了解下Rollup是什么东西，Rollup是和webpack齐名的打包工具。只不过与webpack不同的是，Rollup主打的是打包过程，插件机制也与webpack的实现思路不一样。用官方的描述总结一句话：Rollup是 JS模块打包器。

>Rollup 是一个 JavaScript 模块打包器，可以将小块代码编译成大块复杂的代码，例如 library 或应用程序。Rollup 对代码模块使用新的标准化格式，这些标准都包含在 JavaScript 的 ES6 版本中，而不是以前的特殊解决方案，如 CommonJS 和 AMD。ES6 模块可以使你自由、无缝地使用你最喜爱的 library 中那些最有用独立函数，而你的项目不必携带其他未使用的代码。ES6 模块最终还是要由浏览器原生实现，但当前 Rollup 可以使你提前体验。



实现最简单的Rollup使用 [demo](https://github.com/CrewS/rollupCase)

```JS
// src/js
export function logA() {
    console.log('function logA called')
}

export function logB() {
    console.log('function logB called')
}
```

```JS
// rollup.config.js
const path = require('path')
const pkg = require('./package.json')

const resolve = function(...args){
	return path.resolve(__dirname, ...args)
}
export default {
  input: resolve('./src/index.js'),
  format: 'cjs',
  output: {
		file: resolve('./', pkg.main),
		format: 'umd',
		name: 'test'
	}
}
```

可以看到最终打包产物

![打包产物](https://crews-note-1253247308.cos.ap-guangzhou.myqcloud.com/note/image-20220109145058756.png)

源码文件 + 配置文件 = 打包后的文件 （source + option => output）



## Rollup Config 配置

更详情配置可看[官方文档](https://rollupjs.org/guide/en/#configuration-files)，也可以通过源码[type.d.ts](https://github.com/CrewS/rollup/blob/master/src/rollup/types.d.ts)的定义文件看到每个配置项的类型定义

```js
export default {
  input, 		// 入口文件 字符串或者是数组
  output: {	// 文件输出配置
    file，		//输出的文件名, 对于单个文件打包可以使用该选项指定打包内容写入带路径的文件
   	// InternalModuleFormat = 'amd' | 'cjs' | 'es' | 'iife' | 'system' | 'umd';
    format, // 输出的打包格式一般使用 umd, es, cjs这几种
    dir,    // 配置文件打包后统一输出的基本目录，适用于多文件打包，单文件打包也可以用 file 选项代替
  }, 
  plugins: [] // 插件配置
  external: [] // 指定第三方包不参与打包 现在Rollup默认逻辑绝对路径的包都不会参与打包
}
```



## Rollup 配置加载过程

**为什么输出指令 `rollup -c` ，就能够执行打包呢？**

先到源码入口文件 cli.ts  `cli/cli.ts`

```typescript
import help from 'help.md';
import { version } from 'package.json';
import argParser from 'yargs-parser';
import { commandAliases } from '../src/utils/options/mergeOptions';
import run from './run/index';

// 格式化command
// commandAliases 是别名配置
const command = argParser(process.argv.slice(2), {
	alias: commandAliases,
	configuration: { 'camel-case-expansion': false }
});

// 如果是help指令 则输出提示

if (command.help || (process.argv.length <= 2 && process.stdin.isTTY)) {
	console.log(`\n${help.replace('__VERSION__', version)}\n`);
} else if (command.version) {
	console.log(`rollup v${version}`);
} else {
	try {
		require('source-map-support').install();
	} catch {
		// do nothing
	}
	// run 函数执行command
	run(command);
}

```

**Run 函数** `cli/run/index.ts`

第一步：解析command input输入

第二步：解析`command`里的`environment`环境变量配置

第三步：判断是否是 `watch`模式，如果是则走`watch`模式，不是走到第四步

第四步：通过`getConfigs` 获取配置项 如果`command` 有 `config`则取配置文件、没有则走默认的`loadConfigFromCommand` 函数获取配置

第五步：执行**Rollup build**函数 完成构建

### 执行流程图

通过下面的流程图大概理解整个的执行过程

![指令解析执行过程](https://crews-note-1253247308.cos.ap-guangzhou.myqcloud.com/note/%E6%8C%87%E4%BB%A4%E8%A7%A3%E6%9E%90%E6%89%A7%E8%A1%8C%E8%BF%87%E7%A8%8B.jpg)





## 后续

通过本篇大家都能够初步的掌握如何去使用Rollup、怎么**简单**的配置完成代码的打包过程。经过初步的学习使用，大概的了解Rollup的工作原理，再针对Rollup工作流程的每一个小步骤深入深究，带着问题去阅读源码，这样就能够做到**庖丁解牛**一般，将源码模块一个个拆解出来，再慢慢品尝吸收。下一篇将会针对Rollup的插件使用进行讲解，敬请期待。







## 参考

- [Rollup 配置详解](https://blog.cjw.design/blog/old/rollup)
- [Rollup官方文档](https://rollupjs.org/guide/en/#outputinlinedynamicimports)