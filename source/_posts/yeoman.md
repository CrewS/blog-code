---
title: 前端脚手架搭建
date: 2019-05-08 20:35:02
tags:
	- node
	- typescript
---

## 前言

**什么是脚手架呢？**
在我的理解中脚手架是一个尽可能减少手工操作又可以很好的完成项目搭建的自动化工具。

**脚手架为什么便于项目搭建？**
在工作中，我们很可能会面对类型非常相同的多个项目，在创建新的项目的时候，一般都是通过copy文件到新的目录下，手动修改完成新项目的搭建
通过手工的方式，很有可能会出现一些细小的错误，导致项目运行不起来，浪费时间。
还有通过脚手架可以达到，创建模版文件的效果，例如一个页面是由view.js style.css reducer.js构成，通过脚手架指令可以一次性创建并完成初始化赋值效果。
无论工作还是学习的过程中，我们接触过很多脚手架，一直享受着脚手架带来的便利。不过却比较少去研究，这段时间都在学习和编码自己的脚手架，拓宽自己的知识领域，可以尝试探索学习更多知识。技术团队建设中，对团队框架进行脚手架化，提升团队工作效率和规范标准。


## yeoman-generator

现在搜索脚手架教程，大多数都与 yeoman 有关。确实 yeoman 上有非常多的内置模块，便于脚手架的编写。
例如：自执行函数模块，文件操作模块，命令行问答模块，命令行参数处理等
yeoman 可以写一个 generator 脚手架，使用 yo 指令去执行。不过我们想要更加自定义的方式去实现。例如：创建自己独特的指令 zz，则是利用 yeoman-generator 包，使用 yeoman 的环境调用编写好的 gennerator 结合 bin，就可以完成理想的效果。

## 实战：脚手架编写

在实践前，笔者也是看过很多教程和参考的例子，然后才尝试动手编码。

### 第一个知识点 ：命令行命令的创建。

在目录中创建了一个 index.js

```js
console.log("hello");
```

平时我们使用 node index.js 是可以输出 hello 的，此时我们想尝试使用命令行指令执行，该怎么处理呢
先在 package.json 中增加一个 bin 的属性

```json
{
  "bin": {
    "zz": "./index.js"
  }
}
```

修改 index.js

```js
#! /usr/bin/env node

console.log("hello");
```

一般在发包后， 通过 npm -g 指令安装包会将 zz 指令注入到全局上，在开发过程中想要测试则使用 npm link 指令（具体效果自己可以搜索）
完成以上步骤后，在命令行中输入 zz，显示 hello。
注意事项： 如果 mac 系统下安装 zsh，可能会提示 zsh: permission denied: zz，则需要到 bin 目录下赋予 zz 命令权限
mac 下目录是 `/usr/local/bin`， 执行以下命令 `cd /usr/local/bin`, `chmod +x zz` 完成后即可。

#### 第二个知识点：commander

使用 commander 库，可以极大的拓展指令参数、帮助信息、提示信息、参数校验等操作
example

```js
#! /usr/bin/env node
// index.js
const program = require("commander");
program.command("init <name>").action(function(name) {
  console.log("参数name为: ", name);
});
program.parse(process.argv);
```

安装 commander 包: npm install commander --save，然后 zz init abc
输出结果: 参数 name 为 abc
在 action 的回调函数中，可以获取参数 name，还可以执行后续步骤，例如调用 generator，这样从指令到 commander 参数到执行 generator 的一条线就打通了。

#### 第三个知识点：generator

利用 yeoman-environment 调用 generator

```js
var path = require("path");
module.exports = function(cmd: string) {
  var yeoman = require("yeoman-environment");
  var env = yeoman.createEnv(),
    argumentsArray = Array.prototype.slice.call(arguments, 0);
  var gPath = "./" + path.join("generator-xxx", cmd, "index.js");
  env.register(require.resolve(gPath), cmd);
  env.run(Array.prototype.join.call(arguments, " "));
};
```

这段函数的含义：利用 yeoman-environment 创建了 yeoman 的 env，注册了 yeoman generator，最用调用这个 generator
generator 的编写
查看[官方的教程](https://yeoman.io/authoring/index.html)文档可以了解以下几点内容

- 成员函数 会被自动执行
- 下划线前缀的成员函数 不会被自动执行例如： \_inti(){}
- this.sourceRoot() 获取项目路径
- this.templatePath() 获取
- this.fs.copyTpl(source, target, params) 文件拷贝方法，还能传递变量动态渲染
- this.composeWith() 调用其他的 generator

这些是笔者在构建自己的脚手架中用到的比较多的函数方法，有需要其他的可以阅读官方文档查找相关内容。

#### 第四个知识点

在 generator 中一般会有 templates 文件夹，但是如果 templates 与脚手架绑定一块，只要模版有更新，则脚手架要更新，这样的耦合是不太方便的。
所以了解 download-git-repo 这个 npm 包后，发现可以从远端的 git 仓库下载模版文件，最后 generator 再拷贝渲染模版文件完成项目初始化。
使用方法

```js
// libs
const path = require("path");
const download = require("download-git-repo");

export default (target: any) => {
  console.log(target);
  return new Promise((resolve, reject) => {
    download(
      "direct:https://github.com/yourname/your-project.git#master",
      target,
      { clone: true },
      (err: any) => {
        if (err) {
          reject(err);
        } else {
          // 下载的模板存放在一个临时路径中，下载完成后，可以向下通知这个临时路径，以便后续处理
          resolve(target);
        }
      },
    );
  });
};
```

```js
// app/index.js
if (!fs.existsSync(this.templatePath())) {
  const spinner = spin("Download templates", "Box1");
  spinner.start();
  download(this.templatePath())
    .then(() => {
      spinner.stop();
    })
    .catch(err => console.log(err));
}
```
将download-git-repo封装成promise方法，然后结合命令行的加载动画效果，完成动态加载模版文件，等待模版文件加载完成后再执行后续操作。

## 参考资料
- [yeoman官方文档](https://yeoman.io/authoring/index.html)
- [nodejs脚手架搭建教程](https://juejin.im/post/5a31d210f265da431a43330e#heading-0)