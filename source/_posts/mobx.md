---
title: mobx 实践与理解 （一）
date: 2019-08-02 14:27:02
tags:
	- mobx
---
## 前言
mobx 是一个简单、可拓展的状态管理库。核心就是定义可观察的状态，对状态的变更作出相对应的响应。使用mobx，有助于理解数据驱动视图，状态管理。和redux的思想做对比，会得到更多思考和收获。

## 环境搭建

读者可以按下面步骤搭建实践demo，也可以直接拉去代码实践操作。[demo地址](https://github.com/CrewS/mobx-app)
#### 第一步
这里是用yarn 通过create-react-app 去快速构建项目结构，也可以自己处理
yarn create react-app mobx-app
#### 第二步
使用eject命令，将配置释放出来
yarn eject

#### 第三步
接下来安装mobx与mobx-react框架
yarn add mobx-react --save
yarn add mobx --save

#### 第四步
使用mobx preset处理代码中的装饰器
需要安装两个包
yarn add babel-preset-mobx
yarn add @babel/plugin-transform-react-jsx-source

```json
// package.json
{
	//...
	"babel": {
		"presets": [
			"react-app",
			"mobx"
		]
	}
}
```

## 实践mobx

#### demo1 计数器
```js
// pageA index.js
import React from "react";
import store from "./store";
import { observer } from "mobx-react";

@observer
class PageA extends React.Component {
  render() {
    console.log("render");
    return (
      <div className="content-box">
        <div style={{ marginBottom: "10px" }}>demo1 计数器</div>
        <div>
          <input
            onClick={() => {
              store.setCount(store.count + 1);
            }}
            type="button"
            value="+"
            id="add"
          />
          <span id="incomeLabel">{store.count}</span>
          <input
            onClick={() => {
              store.setCount(store.count - 1);
            }}
            type="button"
            value="-"
            id="minus"
          />
        </div>
      </div>
    );
  }
}
export default PageA;
```
```js
// store.js
import { observable, action } from "mobx";

class Store {
  @observable count = 0;

  @action
  setCount = count => {
    this.count = count;
  };
}
export default new Store();

```
涉及到2个文件
其中store.js中定义了观察的变量，和action 与redux有点类似  
在pageA/index.js中使用了 observer，作用是封装组件的render函数，使其能够响应数据变化。  
给`+` `-`的按钮绑定了事件触发store里定义的action方法，改变观察值，从而让组件重新渲染。  
mobx的使用和它定义的非常符合。定义可观察的状态，改变状态，对应的视图响应状态变化


#### demo2 跨组件通信
demo1只是在同一个组件中使用了mobx,改变的状态和响应视图的都是在同一个组件里。demo2将给大家实现跨组件的状态响应。  
这里演示兄弟组件通信。
```js
// pageB index.js
import React from "react";
import InputView from "./input";
import ShowView from "./show";
import store from "./store";

class PageB extends React.Component {
  render() {
    return (
      <div className="content-box">
        <div style={{ marginBottom: "10px" }}>demo2 跨组件通信</div>
        <div>
          <span>InputView 组件</span>
          <span>输入值：</span>
          <InputView searchStore={store} />
        </div>
        <div>
          <span>ShowView 组件</span>
          <span>响应输出值:</span>
          <ShowView searchStore={store} />
        </div>
      </div>
    );
  }
}
export default PageB;

```
pageB 作为父组件只做了store的导入，和组件的展示
```js
// store.js
// 这里和demo1一样，定义了观察变量和action方法
import { observable, action } from "mobx";

class SearchStore {
  @observable searchText = "";

  @action
  setSearchText = searchText => {
    this.searchText = searchText;
  };
}
export default new SearchStore();

```
```js
// input 组件中 实现了输入的交互和触发了 action方法
import React from "react";
import { observer } from "mobx-react";

@observer
class SearchInput extends React.Component {
  handleInputChanged = event => {
    const { searchStore } = this.props;
    searchStore.setSearchText(event.target.value);
  };

  render() {
    const { searchStore } = this.props;
    return <input value={searchStore.searchText}  onChange={this.handleInputChanged} />;
  }
}
export default SearchInput;

```
```js
// SearchShow 组件中 只做了searchText数据的展示。
// 具体效果：在SearchInput组件中改变searchText的值，会在SearchShow响应显示变化
import React from "react";
import { observer } from "mobx-react";

@observer
class SearchShow extends React.Component {
  render() {
		const { searchStore } = this.props;
    return <span>{searchStore.searchText}</span>;
  }
}
export default SearchShow;

```

## 小结
通过demo1 与 demo2的实现，我们能够理解mobx的使用是比较的方便和简洁的。对于小需求小项目中应用mobx，能够比较快的实现。学习难度也不大。不过还有几个问题可能需要思考和解决的。

- 1、如何管理store，对团队合作编程来说，代码规范也是很重要的事情
- 2、对异步action的处理
- 3、组件销毁与数据初始化问题

下一篇将回答以上问题

## 参考资料
- [mobx中文文档](https://cn.mobx.js.org/)
- [create-react-app中文文档](https://www.html.cn/create-react-app/docs/getting-started/)
