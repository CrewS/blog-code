---
title: yeoman-generator 深入使用
date: 2019-06-11 17:35:01
tags:
	- generator
---

## 前言
在搭建前端脚手架那篇文章中，对yeoman-generator的解释比较简略，可能使用上会出现一些问题
所以今天来继续介绍yeoman-generator

## yeoman-generator
从[官方文档](https://yeoman.io/authoring/index.html)来看
简单说下用到的功能
- [任务执行系统](https://yeoman.io/authoring/running-context.html)
- [文件系统、模版渲染](https://yeoman.io/authoring/file-system.html)
- [命令行问答系统](https://yeoman.io/authoring/user-interactions.html#prompts)

#### 任务执行系统
这里有个概念自动执行task
命名规则影响是否自动执行，影响执行顺序
下划线开头命名的成员函数视为内部函数 类似 _name(){},不会被自动执行
命名为下面的话会按下面的顺序执行
```js
// 1. initializing - Your initialization methods (checking current project state, getting configs, etc)
// 2. prompting - Where you prompt users for options (where you’d call this.prompt())
// 3. configuring - Saving configurations and configure the project (creating .editorconfig files and other metadata files)
// 4. default - If the method name doesn’t match a priority, it will be pushed to this group.
// 5. writing - Where you write the generator specific files (routes, controllers, etc)
// 6. conflicts - Where conflicts are handled (used internally)
// 7. install - Where installations are run (npm, bower)
// 8. end - Called last, cleanup, say good bye, etc
```

通过这个规则，可以很好的规划任务及其执行顺序处理事务。


#### 文件系统、模版系统
有几个重要的API
```js
this.destinationRoot(); // 获取命令行执行的目录
this.templatePath(); // 获取该generator对应的teamplate的目录
this.fs.copyTpl('templatePath', 'targetPath', 'params'）//三个参数模版目录，目标目录，参数
```
copyTpl的方法内置了ejs模版渲染引擎，按ejs的语法可以实现非常自定义的模版渲染。


#### 命令行问答系统
在使用脚手架过程中，经常需要获取用户的输入，命令行输入对于即时响应的场景是非常适合的。
在generator中，有内置的命令行问答系统，是集成[Inquirer.js](https://github.com/SBoudrias/Inquirer.js),可以查看它的文档调用。
在实际的例子中
确认类型的问答如下
```js
module.exports = class extends Generator {
  async prompting() {
    this.answers = await this.prompt([
      {
        type: "confirm",
        name: "cool",
        message: "Would you like to enable the Cool feature?"
      }
    ]);
  }
  writing() {
    this.log("cool feature", this.answers.cool); // user answer `cool` used
  }
};
```
还有更多类型的命令行式问答：列表单选择、多选、input输入、是否确认
通过这几个系统的结合使用可以很好的辅助我们开发脚手架。

## 后语

笔者一开始单纯阅读脚手架的代码，发现并不是很好理解。但是通过实践编码一个脚手架后，发现了解几个核心功能后，编码一个脚手架并不是一件难事
纸上得来终觉浅，绝知此事要躬行。


## 参考
- [yeoman-generator](https://yeoman.io/authoring/index.html)
- [commander](https://github.com/tj/commander.js#readme)
- [promts(Inquirer.js)](https://github.com/SBoudrias/Inquirer.js)
- [ejs文档](https://ejs.bootcss.com/)
