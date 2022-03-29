---
title: 「svelte」是前端排名前三的框架，你知道吗？
date: 2022-03-18 22:29:23
tags: [前端、框架]

index_img: /img/post/js-hook.png
banner_img: /img/bg/banner.webp
---

## 前言

可能大家对svelte这个框还比较陌生，甚至名字都没有听说过。先带大家看下官方的说法

>Svelte 是一种全新的构建用户界面的方法。传统框架如 React 和 Vue 在*浏览器*中需要做大量的工作，而 Svelte 将这些工作放到构建应用程序的*编译阶段*来处理。与使用虚拟（virtual）DOM 差异对比不同。Svelte 编写的代码在应用程序的状态更改时就能像做外科手术一样更新 DOM。

大概讲的就是让代码具备响应式处理的阶段不一样，像React、vue 都有核心core去做一些事情，让数据具备响应式。可能这样讲，还比较抽象，接下来以教程的方式让大家了解svelte怎么写。本篇又名为《svelte》入门教程



## 开始

按[官方文档](https://svelte.dev/)的指令直接敲

```
npx degit sveltejs/template my-svelte-project
cd my-svelte-project
npm install
npm run dev
```

![简单hello world](https://crews-note-1253247308.cos.ap-guangzhou.myqcloud.com/note/image-20220318225633124.png)

简单的hello world界面就出来了！



#### 简单的计数器

这个代码有点像vue，先敲一个Button按钮出来

```export let name;```定义了props传入值name，在后面模板中只需要括号输出即可（大多数模板输出变量都是这样的语法）

```on:click```定义了所有事件都这样传递

```vue
<script>
  // Button.svelte
	export let name;
</script>

<div
	class="button"
	on:click
>
	{
		name
	}
</div>

<style>
	.button{
		display: inline-block;
		padding: 10px;
		border: 1px solid #ddd;
		cursor: pointer;
	}
</style>
```

```vue
<script>
	import Button from './Button.svelte'
	let count = 0;
</script>

<main>
	<Button
		on:click={() => {
			console.log(1)
			count+=1
		}}
		name={`计数器${count}`}
	>
	</Button>
</main>

<style>
</style>
```

最终效果就是完成一个简单的计数器

![计数器](https://crews-note-1253247308.cos.ap-guangzhou.myqcloud.com/note/image-20220318231401083.png)

点击按钮计数会进行增加。整体的编码还是很流畅的，并没有react中的状态管理，也没有vue里的data初始化管理，超级简单的模板语法，大家也可以尝试编码一下，学习曲线很低。



#### 通信

1、**「父子通信props」**上面的例子中也体现了这个点，name传值进去就能展示

2、「Events」事件机制实现跨组件通信 [参考例子](https://www.sveltejs.cn/examples#event-forwarding)

实际上是利用了createEventDispatcher 这个方法进行事件的广播

3、writable store 也能实现跨组件通信，[参考例子](https://www.sveltejs.cn/tutorial/auto-subscriptions)

简单概括下实现的方式就是通过引入同一个store里的值例如count，然后对count进行update方法或者set的执行

就可以达到变更值，与读取值的变化。这里有个语法糖，通过$符号可以实现compute 自动计算的效果。

![$count](https://crews-note-1253247308.cos.ap-guangzhou.myqcloud.com/note/image-20220318233800019.png)

4、具体例子看[context](https://www.sveltejs.cn/tutorial/context-api)，通过getContext与setContext 达到通信的效果

> Contexts and stores seem similar. They differ in that stores are available to *any* part of an app, while a context is only available to *a component and its descendants*. This can be helpful if you want to use several instances of a component without the state of one interfering with the state of the others.

Contexts 和 stores 非常的像，区别的地方就是当使用单例组件的时候如果用store会互相干扰，用Contexts则没有这个问题

当然很多场景可能是Contexts 与 stores会一起使用到。



## 后续

svelte在2021年的欢迎程度已经远超了angular，目前在国外非常流行，因为具备简单的语法快速上手而且体积小，响应快速对于简单的页面来说，使用svelte无疑是一把利刃。对比框架的使用当然没有想象中的简单，需要对比实际的使用场景，结合团队的实际情况才能考虑是否落地推动。当然对svelte的学习并不止于此，后续还会深入svelte的原理，探讨下为什么svelte能够在国外火起来呢！

感谢观看~

觉得有帮助的朋友可以关注一波噢！

「程序与浪漫」持续更新前端/爬虫/安全相关的知识

![公众号](https://crews-note-1253247308.cos.ap-guangzhou.myqcloud.com/note/qrcode.jpg)

