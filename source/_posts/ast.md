---
title: 小知识补给包：AST抽象语法树基础
date: 2022-03-26 21:06:15
tags:
index_img: /img/post/ast.png
banner_img: /img/bg/banner.webp
---


## 抽象语法树

抽象语法树AST是什么？看维基百科中的说明

> 在[计算机科学](https://zh.wikipedia.org/wiki/计算机科学)中，**抽象语法树**（**A**bstract **S**yntax **T**ree，AST），或简称**语法树**（Syntax tree），是[源代码](https://zh.wikipedia.org/wiki/源代码)[语法](https://zh.wikipedia.org/wiki/语法学)结构的一种抽象表示。它以[树状](https://zh.wikipedia.org/wiki/树_(图论))的形式表现[编程语言](https://zh.wikipedia.org/wiki/编程语言)的语法结构，树上的每个节点都表示源代码中的一种结构。之所以说语法是“抽象”的，是因为这里的语法并不会表示出真实语法中出现的每个细节

简单的来说就是一种标准的树形结构数据来代表一段代码函数，通过特定的工具能够实现源码code 与 ast的互相转换。



## AST有什么用？

在前端领域，想必大家都了解**babel**。通过babel，能够将大家书写的ES 高级特性的语法转历史版本里浏览器能够兼容运行的ES5代码。其中babel就是通过不同的转换器，将新特性中的高级语法做了一层转换。通常我们在编写代码中都不需要了解这些，只需要配置下，就能生效起作用。在爬虫领域中javascript代码的混淆对抗离不开AST，了解AST的一些基本知识内容，无论是对“大神”编写的反混淆脚本还是自己编码脚本都是非常有帮助的。今天我们来探究下怎么“玩”AST。



## Babel工具库

把code转化成ast的库很多，这里就不一一列举。这里说的都是babel的周边库

- @babel/parser  解析器
- @babel/traverse 遍历器
- @babel/types 类型判断、ast结构构造
- @babel/generator code生成器



```js
mkdir test-ast
cd test-ast
npm install -D @babel/parser @babel/traverse @babel/types @babel/generator
```

通过上面指令，创建文件夹安装babel库，尝试自己编码简单的脚本

```js
// test-code.js
const a = 1 + 2;
```

```js
// test.js
const parser = require("@babel/parser");
const fs = require('fs');
const traverse = require("@babel/traverse").default;
const t = require("@babel/types");
const generator = require("@babel/generator").default;


var jscode = fs.readFileSync('./code-test.js', {
	encoding: "utf-8"
});

const ast = parser.parse(jscode)
const BinaryExpressionEx = (path) => {
	const{confident, value} = path.evaluate();
	path.replaceInline(t.valueToNode(value))
}
traverse(
	ast,
	{
		BinaryExpression: {
			exit: [BinaryExpressionEx]
		},
	}
)
var { code } = generator(ast, {
	jsescOption: {
			// 自动转义
			minimal: true,
	}
});

console.log(code)
// const a = 3;
```

编写 test-code.js【源码文件】test.js【脚本文件】

然后node test.js 执行脚本文件，最后发现源码里```const a = 1+2``` 变成  ```const a = 3```

这是一个简单的例子，但是如果没有对每个库的作用和AST的基本内容了解，想要编写也是有点难。看过了简单里，我们先来了解AST基础知识。



## AST基础知识

这里介绍几个实用网站

- [在线转化AST ](https://astexplorer.net/) 可以在线看到行级别的代码对应的AST树的结构
- [babel使用手册](https://github.com/jamiebuilds/babel-handbook/blob/master/translations/zh-Hans/plugin-handbook.md#toc-scopes) 对上面4个工具库有比较完整的介绍



Babel 的三个主要处理步骤分别是： **解析（parse）**，**转换（transform）**，**生成（generate）**。.

通过解析后就得到AST抽象语法树，像下图所示一层一层的。每一个层级是一个「**Node**」节点

![抽象语法树](https://crews-note-1253247308.cos.ap-guangzhou.myqcloud.com/note/image-20220327213824762.png)

想要对AST操作的话，就需要对AST树进行递归遍历，在babel工具库 traverse 封装好遍历器，只需要编码对应的函数即可以实现遍历节点并修改。

```js
const MyVisitor = {
  // 变量遍历
  Identifier() {
    console.log("Called!");
  },
  // 表达式遍历 
  BinaryExpression() {
	}
  // ....
};
```

所有的遍历都是有进入和出去2个过程，可针对不同的场景在不同时机处理



**概念 Path**：刚刚所说的遍历器，遍历的并非节点，而是path。

>AST 通常会有许多节点，那么节点直接如何相互关联呢？ 我们可以使用一个可操作和访问的巨大可变对象表示节点之间的关联关系，或者也可以用**Paths**（路径）来简化这件事情。.**Path** 是表示两个节点之间连接的对象。

个人对path理解就是链表的数据结构，通过path能够找到关联的parent 和其他相关的信息，从而找到该节点的位置信息，进行判断和修改

**概念 Scopes（作用域）**

在之前的文章中也讲解过一些JavaScript作用域相关的知识内容，而在AST中，也有对应的概念**Scope**。

这是AST中scope的数据结构，包含了关联的父级节点路径、当前节点路径与所有引用的“绑定”

```js
{
  path: path, // 当前路径
  block: path.node, // 当前节点
  parentBlock: path.parent, // 父级路径
  parent: parentScope, // 父级作用域
  bindings: [...] // 所有引用
}
```

作用域这块内容相对于Node、Path、遍历器来说是比较复杂的一块，后续可以通过更多的例子了解如何处理作用域。



#### 一个简单的对象合并例子

这个是从反混淆脚本`ob-decrypt.js`中摘取一个方法

```js
const parser = require("@babel/parser");
const fs = require('fs');
const traverse = require("@babel/traverse").default;
const t = require("@babel/types");
const generator = require("@babel/generator").default;


var jscode = fs.readFileSync('./code.js', {
	encoding: "utf-8"
});


const ast = parser.parse(jscode)
traverse(
	ast, 
	{
		VariableDeclarator: {
			exit: [merge_obj]
		},
	}
);

function merge_obj(path) {
	// 将拆分的对象重新合并
	const {id, init} = path.node;
	// 判断是否为对象
	if (!t.isObjectExpression(init))
			return;

	let name = id.name;
	// 对象的属性
	let properties = init.properties;

	// 拿到当前路径的作用域
	let scope = path.scope;
	let binding = scope.getBinding(name);
	if (!binding || binding.constantViolations.length > 0) {
			return;
	}
	// scope.block 对应的是当前节点的node
	scope.traverse(scope.block, {
			// AssignmentExpression 是赋值表达式遍历
			AssignmentExpression: function(_path) {
					const left = _path.get("left");
					const right = _path.get("right");
					// 判断是否是对象成员表达
					if (!left.isMemberExpression())
							return;
					const object = left.get("object");
					const property = left.get("property");
					// object.isIdentifier({name: name})  判断对象是否为当前遍历的对象
					// property.isStringLiteral() 判断是否为字符串类型的赋值 例如 obj['a']这种赋值方式
					// property.isIdentifier() 则判断是否 obj.a 这种方式
					// 重新将作用域内的这些赋值表达在初始对象的时候赋予 properties
					// 	_path.remove() 移除当前节点 避免重复赋值
					if (object.isIdentifier({name: name}) && property.isStringLiteral() && _path.scope == scope) {
							properties.push(t.ObjectProperty(t.valueToNode(property.node.value), right.node));
							_path.remove();
					}
					if (object.isIdentifier({name: name}) && property.isIdentifier() && _path.scope == scope) {
							properties.push(t.ObjectProperty(t.valueToNode(property.node.name), right.node));
							_path.remove();
					}
			}
	})
}

var { code } = generator(ast, {
	jsescOption: {
			// 自动转义
			minimal: true,
	}
});

console.log(code)


```



![执行脚本结果](https://crews-note-1253247308.cos.ap-guangzhou.myqcloud.com/note/image-20220327220202630.png)

发现确实可以将对象赋值进行合并简化。这个是对babel工具库的简单应用。通过这个例子也让我们了解到如何去利用作用域找到所有关联的信息并一一修改，这个例子中简单概括就是

- 遍历对象定义
- 找到对象定义的作用域内所有添加属性的地方
- 将添加属性值的“行为”放在初始化对象的时候
- 然后删除该属性定义的节点
- 重复执行至到最后



## 后续

AST的水还是比较深，因为JavaScript编码的过程就比较灵活，所以babel脚本通常需要不断的编码修复各种异常现象。通过了解AST基础知识，想必大家也学会了一些小技巧。留个课后作业题给大家尝试编码，关注后回复公众号 “AST1答案“ ，返回答案给你哦。

```js
// 简单混淆
// decode为还原函数
decode = function(number){
	return	String.fromCharCode(number)

}
const getToken = (hash) => {
	const obj = {}
	obj[decode(97)] = 6
	obj[decode(115)] = 6
	obj[decode(116)] = 6
}
```



