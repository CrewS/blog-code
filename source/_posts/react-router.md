---
title: React Router v4
date: 2019-02-16 11:15:23
tags:
    - react
---
## 前言
之前使用react router都是参考着demo写路由，此次将整体过一遍 React Router v4的教程与文档。
系统学习react router，深入理解前端路由的概念。

## React Router
v4 的版本将路由进行了拆分，将其放到了各自的模块中，不再有单独的 router 模块，充分体现了组件化的思想；另外，`<BrowserRouter>` 的使用与之前作为 history 属性传入的方式也不同了。v4中route可以当作component一样使用，只不过只在路由匹配后才会生效。

## 路由匹配规则
#### 1、包含式路由与exact
react router的路由path匹配是包含式路由匹配。
例如path: /bar/foo 会匹配 [/,/bar,/bar/foo]的路由
如果单纯使用<Route>,路由匹配的组件将所有都生效。 
**exact** 当值为true时，则要求路径与location.pathname必须完全匹配。

#### 2、独立路由 Switch
<Switch> 组件则是渲染匹配地址的第一个 <Route> 或者 `<Redirect>`。

#### 3、重定向 Redirect
渲染`<Redirect>` 的时候将会导航到一个新的地址。这个新的地址将会覆盖在访问历史记录里面的原地址，就像服务端的重定向（HTTP 3XX）一样。
`<Redirect>` 参数对应的作用如下
to：重定向目标地址或URL
push：当设置为 true 时，重定向（redirecting）将会把新地址加入访问历史记录里面，而不是替换掉目前的地址。
from：需要被重定向的path，当渲染一个包含在<Switch>里面的`<Redirect>`的时候，这可以用作匹配一个地址。
#### 4、组合使用
实际的开发生产中，基本都需要组合使用路由规则实现我们的需求
例子如下
```js
<main>
<Switch>
    <Route exact path='/' component={Home}/>
    <Route path='/roster' component={Roster}/>
    <Route path='/schedule' component={Schedule}/>
    <Route path='error' component={Error}/>
    <Redirect to='/error' />
</Switch>
</main>
```
路由与页面分别对应
/ => home
/roster => roster
/schedule => schedule
/error => error
当什么都没匹配到的时候重定向到 /error 路由然后页面为 error

## 嵌套布局概念
v4中的概念是路由可以当组件一样的使用，那么在布局的时候可以采用这样的方式进行布局
看以下官方的介绍文档中的例子，对比两种方式区别，理解v4特性带来的好处。
需求是 “拓展用户模块”，需要一个可以浏览用户信息界面和每个人的个人用户信息页面
```react
const PrimaryLayout = props => {
  return (
    <div className="primary-layout">
      <PrimaryHeader />
      <main>
        <Switch>
          <Route path="/" exact component={HomePage} />
          <Route path="/users" exact component={BrowseUsersPage} />
          <Route path="/users/:userId" component={UserProfilePage} />
          <Route path="/products" exact component={BrowseProductsPage} />
          <Route path="/products/:productId" component={ProductProfilePage} />
          <Redirect to="/" />
        </Switch>
      </main>
    </div>
  )
}

const BrowseUsersPage = () => (
  <div className="user-sub-layout">
    <aside>
      <UserNav />
    </aside>
    <div className="primary-content">
      <BrowseUserTable />
    </div>
  </div>
)

const UserProfilePage = props => (
  <div className="user-sub-layout">
    <aside>
      <UserNav />
    </aside>
    <div className="primary-content">
      <UserProfile userId={props.match.params.userId} />
    </div>
  </div>
)
```
以上的方法是直接对每个页面增加路由以及对应的页面，不过这样会造成重复代码
重构代码如下
```react
const PrimaryLayout = props => {
  return (
    <div className="primary-layout">
      <PrimaryHeader />
      <main>
        <Switch>
          <Route path="/" exact component={HomePage} />
          <Route path="/users" component={UserSubLayout} />
          <Route path="/products" component={ProductSubLayout} />
          <Redirect to="/" />
        </Switch>
      </main>
    </div>
  )
}

const UserSubLayout = () => (
  <div className="user-sub-layout">
    <aside>
      <UserNav />
    </aside>
    <div className="primary-content">
      <Switch>
        <Route path="/users" exact component={BrowseUsersPage} />
        <Route path="/users/:userId" component={UserProfilePage} />
      </Switch>
    </div>
  </div>
)

const BrowseUsersPage = () => <BrowseUserTable />
const UserProfilePage = props => <UserProfile userId={props.match.params.userId} />
```
这种方式是将相似的界面抽取出来做成layout容器组件，在子组件中再去进行路由匹配渲染不同页面。route 组件化思想得到充分的利用。

## match 对象
match 对象包含了 <Route path> 如何与URL匹配的信息。match 对象包含以下属性：
- params：（ object 类型）即路径参数，通过解析URL中动态的部分获得的键值对。
- isExact：( 当为 true 时，整个URL都需要匹配。
- path：（ string 类型）用来做匹配的路径格式。在需要嵌套 <Route> 的时候用到。
- url：（ string 类型）URL匹配的部分，在需要嵌套 <Link> 的时候会用到。

获取match对象的方式
- 在 Route component 中，以 this.props.match 方式。
- 在 Route render 中，以 ({ match }) => () 方式。
- 在 Route children 中，以 ({ match }) => () 方式
- 在 withRouter 中，以 this.props.match 方式
- matchPath 的返回值

## 组件Link与Navlink
Navlink是Link的一个特定版本, 会在匹配上当前URL的时候会给已经渲染的元素添加样式参数
`<Link>`属性
to: 目标路由
replace:当设置为 true 时，点击链接后将使用新地址替换掉访问历史记录里面的原地址。当设置为 false 时，点击链接后将在原有访问历史记录的基础上添加一个新的纪录。 默认为false

`<Navlink>`属性
activeClassName：激活时添加类名
activeStyle：激活时添加样式
exact：控制是否完全匹配
strict：匹配路由是否包/闭合
isActive: 添加额外函数用于判断是否激活

## 参考
- [react router v4中文文档](https://router.happyfe.com/)
- [React Router 4：痛过之后的豁然开朗](https://www.jianshu.com/p/bf6b45ce5bcc)
- [All About React Router 4](https://css-tricks.com/react-router-4/)