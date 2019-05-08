---
title: react-saga
date: 2019-03-26 13:46:42
tags:
    - react
---
# react-saga的学习和理解

## 前言

在使用react redux的时候，会经常遇到需要处理异步action的情况。处理异步action的方法有几种。
其中redux-thunk，redux-saga都是处理异步action的中间件。利用这些中间件可以很好的达到我们预期效果

## redux-saga

`redux-saga`是一个用于管理应用程序 Side Effect（副作用，例如异步获取数据，访问浏览器缓存等）的 library，它的目标是让副作用管理更容易，执行更高效，测试更简单，在处理故障时更容易。

个人是这么理解redux-saga的，在app中注入redux-saga中间件后，saga effects函数中对相对应的action进行监听，在代码中执行触发dispatch 对应action的时候，saga的effects 函数会对其进行拦截处理，中途进行一些异步、同步、或者是执行调用其他effects函数。在effects函数中，可以同步的方式书写异步代码。
相对于redux-thunk，saga的优势在于effects函数中对各种异步的处理比较方便和容易拓展。

## 计数器demo例子

Counter 组件

```js
import React, { Component, PropTypes } from 'react'

const Counter = ({ value, onIncrement, onDecrement, onIncrementAsync }) =>
      <div>
        <button onClick={onIncrement}>
          Increment
        </button>
        {' '}
        <button onClick={onDecrement}>
          Decrement
        </button>
        <button onClick={onIncrementAsync}>
          delay Increment
        </button>
        <hr />
        <div>
          Clicked: {value} times
        </div>
      </div>

Counter.propTypes = {
  value: PropTypes.number.isRequired,
  onIncrement: PropTypes.func.isRequired,
  onDecrement: PropTypes.func.isRequired,
  onIncrementAsync:  PropTypes.func.isRequired
}
export default Counter
```

main.js

```js
import React from 'react'
import ReactDOM from 'react-dom'
import { createStore, applyMiddleware } from 'redux'
import createSagaMiddleware from 'redux-saga'
import rootSaga from './saga'
import Counter from './Counter'
import reducer from './reducers'

const sagaMiddleware = createSagaMiddleware()
const store = createStore(
  reducer,
  applyMiddleware(sagaMiddleware)
)
sagaMiddleware.run(rootSaga)
const action = type => store.dispatch({type})

function render() {
  ReactDOM.render(
    <Counter
      value={store.getState()}
      onIncrement={() => action('INCREMENT')}
      onDecrement={() => action('DECREMENT')}
      onIncrementAsync={() => action('INCREMENT_ASYNC')} />,
    document.getElementById('root')
  )
}

render()
store.subscribe(render)
```

saga.js

```js
import { delay } from 'redux-saga'
import { put, takeEvery, all } from 'redux-saga/effects'

export function* incrementAsync() {
    yield delay(1000)
    yield put({ type: 'INCREMENT' })
}
export function* watchIncrementAsync() {
    yield takeEvery('INCREMENT_ASYNC', incrementAsync)
}
export default function* rootSaga() {
    yield all([
        watchIncrementAsync()
    ])
}
```

在计数器中有三个按钮 `Increment`、`Decrement`、`Delay Increment`
点击`Increment`，计数马上加1，点击`Decrement`，计数马上减1，点击`Delay Increment`计数延迟1秒加1

原理解析
在saga中，对于`INCREMENT_ASYNC`这个action进行了监听，如果触发了这个action则会执行saga中的incrementAsync函数，在该函数中，有delay延迟函数。yield delay(1000)的意思是等待延迟1秒，然后执行yield put(...)，执行触发`INCREMENT`action，然后触发reducer 对state的值的修改。

整个流程 监听`INCREMENT_ASYNC`，触发`INCREMENT_ASYNC`，执行incrementAsync，dispatch`INCREMENT`，最后修改state，触发UI更新。
在saga的effects函数中可以对异步处理的结果进行再次处理整合，最后再派发执行下一步。
整个流程是有序执行可监控的。这样我们就简单理解了redux-saga的原理和流程，进一步学习redux-saga则需要对API有更深入的了解

## 进阶学习redux-saga

- take
- fork
- takeEvery
- takeLatest

takeLatest & takeEvery 是监听action，一遍一遍执行，不会停止。takeLatest与takeEvery的区别在于，如果同个action多次触发，takeLatest只会执行最后一个action，往前的action会取消调用，而takeEvery则每个action均会执行调用。take则只执行一次。fork则在后台建立任务监听，非阻塞主流程。
所以会经常使用 fork+ while(true)+take的方式实现可控制的effect函数。
查看例子

```js
import { fork, call, take, put } from 'redux-saga/effects'
import Api from '...'

function* authorize(user, password) {
  try {
    const token = yield call(Api.authorize, user, password)
    yield put({type: 'LOGIN_SUCCESS', token})
  } catch(error) {
    yield put({type: 'LOGIN_ERROR', error})
  }
}

function* loginFlow() {
  while(true) {
    const {user, password} = yield take('LOGIN_REQUEST')
    yield fork(authorize, user, password)
    yield take(['LOGOUT', 'LOGIN_ERROR'])
    yield call(Api.clearItem('token'))
  }
}
```

收到`LOGIN_REQUEST`的action的时候，会进入loginFLow，然后提交检验authorize，这个过程是后台任务执行的，此时如果触发了`LOGOUT`action,函数不会被阻塞，会执行`call(Api.clearItem('token'))`。
redux-saga的高级用法就是通过这些API能够很好的控制异步流程。

## 参考资料

- [redux-saga中文教程](https://redux-saga-in-chinese.js.org/)
- [计数器demo例子](https://github.com/redux-saga/redux-saga-beginner-tutorial.git)
