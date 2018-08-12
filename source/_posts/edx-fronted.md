title: open edx 前后端分离初探
date: 2018-07-28 14:26:30
tags:
    - edx
---

## 前言
opene edx 是开源的教育平台，在未来的开发计划中，open edx也在往前后端分离的方向进行。现在就来初探一下，edx前后端分离的思路
笔者从开始了解edx的studio-frontend前端仓库，到尝试构建，到成功搭建启动的过程，还是遇到了许多问题，有可能读者按官方的流程来走，可能会遇到挺多坑。

## 初探edx前后端分离
### 前提说明
笔者搭建的环境：Mac os
后端server: [edx/devstack [h版本]](https://github.com/edx/devstack)
Docker版本: [Version 18.06.0-ce-mac70](https://www.docker.com/community-edition)

### 安装搭建
假设devstack已搭建。如果没有搭建，请看[devstack搭建教程](../edx-fronted/)

第一步: 克隆仓库代码 `git clone git@github.com:edx/studio-frontend.git`

第二步: `cd studio-frontend` 编辑 Dockerfile文件第六行
~~RUN npm i -g npm@5.8.0~~ 
RUN npm i -g npm@latest // 简单来说就是将npm的版本改为最新版本

第三步: `make up`

第四步: 等待启动完毕后，打开 `http://localhost:18011/assets.html`，点击 `Log in`，输入账号 [edx@example.com](mailto:edx@example.com) 密码 edx 登录系统后，回到 assets页面，即可看到如下页面


![image.png | left | 747x373](https://cdn.nlark.com/yuque/0/2018/png/151680/1533003086237-da40c61b-b4d7-4243-971a-247126e08667.png "")

至此，一个官方的studio-frontend 前端搭建起来
### 分析
__studio-frontend __前后端分离采用的技术是
__webpack__ + __react__ + __redux__ + __bootstrap__ + __paragon__
__webpack__是前端工程流工具
__react__ 与 __paragon__、__bootstrap__ 构建了高级的功能组件
__redux__ 集中式对页面数据进行管理
该项目采取的是多页面多入口的打包方式，打包构建后的结果如下
├── accessibilityPolicy.min.css  
├── accessibilityPolicy.min.js  
├── accessibilityPolicy.min.js.map  
├── assets.min.css  
├── assets.min.js  
├── assets.min.js.map  
├── common.min.css  
├── common.min.js  
├── common.min.js.map  
├── courseHealthCheck.min.css  
├── courseHealthCheck.min.js  
├── courseHealthCheck.min.js.map  
├── courseOutlineHealthCheck.min.css  
├── courseOutlineHealthCheck.min.js  
├── courseOutlineHealthCheck.min.js.map  
├── editImageModal.min.css  
├── editImageModal.min.js  
├── editImageModal.min.js.map  
├── i18n
│   └── messages
│       ├── ar.json
│       ├── es\_419.json
│       ├── fr.json
│       └── zh\_CN.json
├── i18nMessages.min.js
├── i18nMessages.min.js.map
├── runtime.min.js
└── runtime.min.js.map
最后将js.css文件嵌入在后端模版文件中，构成页面


![image.png | left | 747x380](https://cdn.nlark.com/yuque/0/2018/png/151680/1533008098233-a538f9fa-9af0-410d-844d-737157a297b6.png "")

这样做其实还没有完全独立前后端分离，并且一些css可能会与原来系统中的css有冲突
官方表示在未来，理想中的前后端分离是前端代码完全静态托管在另一个server服务器中，该服务器通过REST（或[GraphQL](http://graphql.cn/)）API 与Studio通信
### 文件与目录
#### 目录结构


![image.png | left | 334x435](https://cdn.nlark.com/yuque/0/2018/png/151680/1533004031441-ab1ae521-9a1d-4286-bcfa-e601d8c15eb4.png "")

package.json包相关内容


![image.png | left | 332x387](https://cdn.nlark.com/yuque/0/2018/png/151680/1533005413128-8a486aee-d770-484f-baa2-720fbdc74aff.png "")

可以看到几个比较重要的包
* @edx/edx-bootstrap
* @edx/paragon
* react、react-redux、redux-thunk
* react-intl
在一篇介绍[studio-frontend](https://openedx.atlassian.net/wiki/spaces/FEDX/pages/548766004/Studio-Frontend+Developing+Frontend+Separate+from+Platform) 前后端分离思路的文章中提及到 edx前后端分离采用的是react作为基础框架，在此基础上，进行前后端分离开发， paragon是官方的组件库，用来组合构建更高级的组件服务于edx平台，bootrstrap + Open edX 主题将为paragon设置样式内容。redux ，react-intl则是react生态圈中的一些插件，组合使用完成系统功能需求。
#### 构建调试
官方的方式是构建出一个docker，在docker中配置好了环境，启动了项目，再将地址共享在主机中访问
API的请求则是采用了proxy代理的方式，代理转发到后端服务中
有特定的需求可以在webpack.dev.config中配置，例如接口地址的改变
笔者也尝试过，直接克隆项目代码。在项目下安装依赖包，然后开始构建调试前端代码，只不过需要对webpack proxy代理地址做一些调整

#### 生产发布
指令很简单 `npm run build`
如果需要增加入口，则需在 webpack.common.config 的entry中相应增加
如果对生产发布的个性化调整，只需要对webpack进行相关配置即可

### 结语
这个只是笔者对open edx前后端分离思路的初步探索的一些总结，还有许多内容的值得去深挖。
将来如果对此有更深的理解和实践，再给大家进行分享
continue...



