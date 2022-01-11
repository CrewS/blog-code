---
title: Rollup原理分析（二）插件使用
date: 2022-01-10 20:22:08
tags: [rollup, web打包工具系列]

index_img: /img/post/rollup.jpeg
banner_img: /img/bg/banner.webp
---

## 前言

当前分析的Rollup版本：`2.63.0`

对于js的打包，在面对不同的业务场景时候，是需要不同的打包”姿势“。那怎么使用不同姿势呢？webpack有loader、plugin在打包的过程中处理各种各样的场景。Rollup说：我万变不离其宗，插件就能够做到相同的效果。今天这篇文章，主要讲的就是怎么使用Rollup插件。



## Rollup插件的简单使用

```JS
const path = require('path')
const pkg = require('./package.json')

import resolveNode from '@rollup/plugin-node-resolve';
import commonjs from '@rollup/plugin-commonjs';

const resolve = function(...args){
	return path.resolve(__dirname, ...args)
}
export default {
  input: resolve('./src/index.js'),
  format: 'cjs',
	external: [
		'ms'
	],
  output: {
		file: resolve('./', pkg.main),
		format: 'umd',
		name: 'test',
		globals: {
			ms: "ms"
		},
		exports: 'named',
	},
	name: 'tool',
	Plugin: [
		resolveNode(),
		commonjs()
	]
}
```

用上篇文章的例子来说，这里使用了2个插件 `resolveNode`，`commonjs`插件

第一个插件：`@rollup/plugin-node-resolve `是用来告诉 Rollup 如何查找外部模块

第二个插件：`@rollup/plugin-commonjs `就是用来将 CommonJS 转换成 ES2015 模块的。npm 中的大多数包都是以 CommonJS 模块的形式出现的，所以这个插件使用率也是非常高。

Rollup的插件用法都是特别简单，先导入插件，然后在`Plugin`字段执行放入插件数组当中。

## Rollup源码的插件分析

既然我们了解了插件`如何使用`，那我们就来探究下不同的插件究竟做了什么事情。

我们的目标是研究Rollup源码框架，那就从框架本身的插件入手，先看看Rollup自己使用了什么插件呢！

文件地址 [rollup.config.ts](https://github.com/CrewS/rollup/blob/master/rollup.config.ts)

```js
// ...
import addCliEntry from './build-plugins/add-cli-entry';
import conditionalFsEventsImport from './build-plugins/conditional-fsevents-import';
import emitModulePackageFile from './build-plugins/emit-module-package-file';
import esmDynamicImport from './build-plugins/esm-dynamic-import';
import getLicenseHandler from './build-plugins/generate-license-file';
import replaceBrowserModules from './build-plugins/replace-browser-modules';
// ...
```

这几个都是自身core 源码中的插件包逐个分析

#### addCliEntry

在最开始看框架源码的时候，就一直没有找到编译 cli 的配置入口，想了半天怎么也想不通`bin/rollup` 这个文件是怎么编译出来的。最后看到这个插件，原来是它的作用。

直接看代码、发现就是在构建前通过 `this.emitFile` 增加一个入口配置，这样就能够打包出来一个 `bin/rollup` 的 `bin` 文件提供用户执行 rollup 这样的指令操作。`renderChunk` 则是给 `bin` 文件增加文件头部的配置信息。大家也可以学这一招，生成 bin后再增加这行信息，就可以避免不知道如何在 js， ts 文件顶部增加 `#!/usr/bin/env node `这样的信息了

```js
import MagicString from 'magic-string';
import { Plugin } from 'rollup';

export default function addCliEntry(): Plugin {
	return {
    // 构建前
		buildStart() {
			this.emitFile({
				fileName: 'bin/rollup',
				id: 'cli/cli.ts',
				preserveSignature: false,
				type: 'chunk'
			});
		},
		name: 'add-cli-entry',
    // 渲染 chunk 变代码块
		renderChunk(code, chunkInfo) {
			if (chunkInfo.fileName === 'bin/rollup') {
				const magicString = new MagicString(code);
				magicString.prepend('#!/usr/bin/env node\n\n');
				return { code: magicString.toString(), map: magicString.generateMap({ hires: true }) };
			}
			return null;
		}
	};
}
```

#### conditionalFsEventsImport

这个插件是 2 年前的 PR 合并的代码，功能主要是替换 chokidar 里依赖`fsevents`使用的 `fsevents-handler.js` ，在使用这个库的时候替换成自定义的一段 code

通过当年的 [PR ](https://github.com/rollup/rollup/pull/3331)看看，作者说当时最大的两个依赖 `chokidar`和 `micromatch`解耦出来作为 watch 功能的加载和使用，`chokidar`本身是无痛使用的，但是`chokidar`  依赖  `fsevents` 的部分功能依赖了原生代码，这里做了这个 Plugin 主要也是解决`fsevents` 如果不能正常安装，也是不会影响运行的。

> so that chokidar will now behave the same no matter if fsevents is present and valid or missing.

![PR 文字](https://crews-note-1253247308.cos.ap-guangzhou.myqcloud.com/note/image-20220110222309904.png)

搞懂这个插件，让我了解了 `fsevents`， `chokidar` 这几个包的使用流程

加载 chokidar 的时候会将`fsevents-handler.js `替换成自己的包

![conditional-fsevents-import流程图](https://crews-note-1253247308.cos.ap-guangzhou.myqcloud.com/note/conditional-fsevents-import.ts.jpg)

```typescript
let fsEvents: unknown;
let fsEventsImportError: Error | undefined;

export async function loadFsEvents(): Promise<void> {
	const moduleName = 'fsevents';

	try {
		({ default: fsEvents } = await import(moduleName));
	} catch (err: any) {
		fsEventsImportError = err;
	}
}
// 动态加载 fsevents 如果异常情况则做兼容处理
// A call to this function will be injected into the chokidar code
export function getFsEvents(): unknown {
	if (fsEventsImportError) throw fsEventsImportError;
	return fsEvents;
}

```

这种`优雅降级` 降级的处理方式，又让我学习到了！

#### getLicenseHandler

动态生成两个插件`collectLicenses`， `writeLicense`、是用来给自己打包的库文件头插入 license，如果你也有做开源工作，可以参考这种做法哦

插件式的添加 license！这里依赖另一个插件`rollup-plugin-license`，今天就不作更详细的讲解了

```typescript
// ...
export default function getLicenseHandler(): {
	collectLicenses: PluginImpl;
	writeLicense: PluginImpl;
} {
	const licenses = new Map();
	return {
		collectLicenses() {
			function addLicenses(dependencies: Dependency[]) {
				for (const dependency of dependencies) {
					licenses.set(dependency.name, dependency);
				}
			}

			return license({ thirdParty: addLicenses });
		},
		writeLicense() {
			return {
				name: 'write-license',
				writeBundle() {
					generateLicenseFile(Array.from(licenses.values()));
				}
			};
		}
	};
}
```



#### replaceBrowserModules

这个插件看起来就比较简单一些了，对特定的库用浏览器模块替换模块，用作生成

`dist/rollup.browser.js` 和 `dist/es/rollup.browser.js`

```typescript
import path from 'path';
import { Plugin } from 'rollup';

const ID_CRYPTO = path.resolve('src/utils/crypto');
const ID_FS = path.resolve('src/utils/fs');
const ID_HOOKACTIONS = path.resolve('src/utils/hookActions');
const ID_PATH = path.resolve('src/utils/path');
const ID_RESOLVEID = path.resolve('src/utils/resolveId');

export default function replaceBrowserModules(): Plugin {
	return {
		name: 'replace-browser-modules',
		resolveId: (source, importee) => {
			if (importee && source[0] === '.') {
				const resolved = path.join(path.dirname(importee), source);
				switch (resolved) {
					case ID_CRYPTO:
						return path.resolve('browser/crypto.ts');
					case ID_FS:
						return path.resolve('browser/fs.ts');
					case ID_HOOKACTIONS:
						return path.resolve('browser/hookActions.ts');
					case ID_PATH:
						return path.resolve('browser/path.ts');
					case ID_RESOLVEID:
						return path.resolve('browser/resolveId.ts');
				}
			}
		}
	};
}
```



## 后续

经过这篇博客的讲解，大家是否懂如何去使用插件了呢？不得不说官方 core 使用的插件还是挺有意思的，下次再和大家聊聊如何编写一个插件，插件的几个关键钩子是处于怎么构建环境的什么流程。等我们把插件的机制了解，最后再整体去看 Rollup 打包过程，可能大家学习起来的难度就没有这么大了。



## 参考资料

- [chokidar 仓库代码](https://github.com/paulmillr/chokidar)
- [Rollup watch option 配置](https://www.rollupjs.com/guide/big-list-of-options)

