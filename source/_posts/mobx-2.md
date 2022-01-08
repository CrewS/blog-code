---
title: mobx 实践与理解 （二）
date: 2019-08-04 16:52:06
tags:
	- mobx
---

## 前言
在《mobx 实践与理解 一》中，我们只是简单的使用了mobx，实现了类似redux那样的状态管理功能。实际工作需求要处理的场景远远不止这些。
今天要给大家解决《一》中提出的几个问题。

## 实践操作
### 异步action问题
异步在前端的工作中是总要处理的问题，比如最简单的场景就是异步获取接口数据。那么在mobx中我们怎么使用呢？
可以看到官方中文文档中有介绍几种，我这里将演示async/await方法。[demo地址](https://github.com/CrewS/mobx-app.git)
```js
// store.js
import { observable, action, runInAction } from "mobx";
// 异步获取数据的函数
const getData = async () => {
  const data = await new Promise((resovle, reject)=> {
    setTimeout(() => {
      const d = [];
      const c = Math.random() * 10;
      for(let i =0;i< c;i++){
        d.push(i);
      }
      resovle(d)
    }, 500);
  });
  return data;
}
class Store {
  @observable list = [];
  @observable loading = false;
	// 异步action
  @action
  getData = async () => {
    this.setLoading();
    const data = await getData();
    runInAction(() => {
      this.list = data;
      this.loading = false;
    })
    console.log(data)
  };
  setLoading = () => {
    this.loading = true;
  }
}
export default new Store();

```
```js
// pageC/index.js
import React from "react";
import store from "./store";
import { observer } from "mobx-react";

@observer
class PageA extends React.Component {
  componentWillMount(){
    // store.reset();
  }
  componentWillUnmount(){
    store.reset();
  }
  render() {
    console.log("render");
    const { list } = store;
    return (
      <div className="content-box">
        <div style={{ marginBottom: "10px" }}>demo3 异步action</div>
        <div>
          {
            store.loading === true ?
            '加载中....'
            :
            <div
              onClick={() => {
                store.getData();
              }}
              className="btn"
            >
              点击获取新数据
            </div>
          }
          <ul>
            {list.map(x => {
              return <li key={x}>{x}</li>;
            })}
          </ul>
        </div>
      </div>
    );
  }
}
export default PageA;
```
可以看到getData是一个异步生成数据的函数，然后action getData则是异步action。定义了action名后，用async 修饰函数。
在view中调用这个action和调用普通action是一样的。直接store.getdata() 即可。
在action中修改数据，会一次次触发渲染。这里有个工具函数 runInAction，将
runInAction 合并action状态操作，只触发一次渲染。可以通过将代码`this.list = data;this.loading = false;`放在外面和里面对比render的次数得到结论。

### 数据重置问题
数据的是独立的，在组件销毁后，数据对象还存在内存当中。所以再次挂载组件的时候，原数据并没有清除。这里有两种方式处理数据。
1、在componentWillMount()调用 reset()函数清空数据
2、在componentWillUnmount() 调用 reset() 函数清空数据
3、每个组件均有init()方法，然后componentWillMount的时候初始化数据

### 开发规范
接触mobx没多久，看mobx的使用都是比较简单的，怎么样都能实现功能，如果没有一套规范。那么团队代码中去维护代码就变得非常困难。
暂时我个人的实现规范是，一个组件中包含着
- view： 组件UI、处理视图的
- store： 类似reducer，处理状态数据的
- server： API接口请求的
这个是组件中的文件结构，但是一般项目中会有些公共的内容，比如公共的接口，公共的store数据这可怎么处理呢？我的想法是在根节点中注入一个全局store，这个store是存储共享数据的。公共的组件接口则抽取出来放在common目录下。整个目录结构可参考demo代码中的。


## 小结
这篇博客中我们处理了，异步action、数据重置、代码规范问题。我将在（三）中对mobx的源码进行一些解析。让我们知道为什么定义了观察的状态后，视图就能够响应。还有代码规范，我将参考下网上的最佳实践，看看还有没有更好的处理方案。


## 参考资料
- [mobx中文文档](https://cn.mobx.js.org/)

