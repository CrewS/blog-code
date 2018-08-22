---
title: redux 原理理解 & 源码阅读 （一）
date: 2018-08-22 16:02:15
tags: 
    - redux
    - 源码
---
### 前言
react-redux在实现react前端项目中使用还是比较频繁的，虽然用了比较多次。但是对原理的理解还是不够深入，所以今天回顾下redux。

### redux

redux是什么，为什么要用redux，这应该是许多初步接触redux开发者的疑问
这里摘抄官方文档的说明  
#### what
>Redux 是 JavaScript 状态容器，提供可预测化的状态管理。  

#### why
> 随着 JavaScript 单页应用开发日趋复杂，JavaScript 需要管理比任何时候都要多的 state。（状态）state 在什么时候，由于什么原因，如何变化已然不受控制。

redux通过一些原则限制状态更新发生的时间和方式，让这些变化变得可测

### 三大原则
- 单一数据源
- state是只读
- 使用纯函数来执行（reducer修改state状态，reducer是纯函数）

### 数据流
严格的单向数据流是 Redux 架构的设计核心。
redux 应用中数据的生命周期（数据流动）
- 调用 store.dispatch(action)
- Redux store 调用传入的 reducer 函数。
- 根 reducer 应该把多个子 reducer 输出合并成一个单一的 state 树。
- Redux store 保存了根 reducer 返回的完整 state 树。

### redux源码阅读
```javascript
function dispatch(action) {
    if (!isPlainObject(action)) {
      throw new Error(
        'Actions must be plain objects. ' +
          'Use custom middleware for async actions.'
      )
    }

    if (typeof action.type === 'undefined') {
      throw new Error(
        'Actions may not have an undefined "type" property. ' +
          'Have you misspelled a constant?'
      )
    }

    if (isDispatching) {
      throw new Error('Reducers may not dispatch actions.')
    }

    try {
      isDispatching = true
      currentState = currentReducer(currentState, action)
    } finally {
      isDispatching = false
    }

    const listeners = (currentListeners = nextListeners)
    for (let i = 0; i < listeners.length; i++) {
      const listener = listeners[i]
      listener()
    }

    return action
  }
```
```javascript
export default function createStore(reducer, preloadedState, enhancer){
    let currentReducer = reducer;
    ....
}
```
dispatch 将 currentState传入 currentReducer，执行currentReducer函数,
currentReducer是createStore时候传入的reducer  
所以可以看出dispatch是这里将action传递给reducer  

#### **疑问：dispatch的匹配原理（或者说action与reducer的匹配原理）是什么？**  
开始我理解的就是，每次dispatch一个action，会只走到对应reducer中，去匹配对应的action执行相应代码，然后我将官方例子的代码修改了一下，去验证是否对的
```javascript
let id = 1000;// 增加的
const todos = (state = [], action) => {
    console.log('todos',state)
    switch (action.type) {
      case 'ADD_TODO':
        return [
          ...state,
          {
            id: action.id,
            text: action.text,
            completed: false
          }
        ]
      case 'TOGGLE_TODO':
        return state.map(todo =>
          (todo.id === action.id) 
            ? {...todo, completed: !todo.completed}
            : todo
        )
      //修改的
      default:
        return [
          ...state,
          {
            id: id++,
            text: `default ${id++}`,
            completed: false
          }
        ]
    }
  }
  
  export default todos
```
结果就是dispatch非 ADD_TODO or TOGGLE_TODO的时候，也会增加todo任务
就证明了我的猜想是错误的，dispatch给reducer后，reducer会将这个action匹配给它下面所有子reducer，这也说明了为什么action type不能重复命名的原因
还有规范的开发中，需要用actionType.js,用常量定义actiontype，方便协同开发避免出错

#### **疑问：数据流的第三步，“根 reducer 应该把多个子 reducer 输出合并成一个单一的 state 树。” 这里怎么理解，store tree是怎么形成的**  
阅读源码 combineReducer
```javascript
....
export default function combineReducers(reducers) {
  const reducerKeys = Object.keys(reducers)
  const finalReducers = {}
  for (let i = 0; i < reducerKeys.length; i++) {
    const key = reducerKeys[i]

    if (process.env.NODE_ENV !== 'production') {
      if (typeof reducers[key] === 'undefined') {
        warning(`No reducer provided for key "${key}"`)
      }
    }

    if (typeof reducers[key] === 'function') {
      finalReducers[key] = reducers[key]
    }
  }
  const finalReducerKeys = Object.keys(finalReducers)

  let unexpectedKeyCache
  if (process.env.NODE_ENV !== 'production') {
    unexpectedKeyCache = {}
  }

  let shapeAssertionError
  try {
    assertReducerShape(finalReducers)
  } catch (e) {
    shapeAssertionError = e
  }

  return function combination(state = {}, action) {
    if (shapeAssertionError) {
      throw shapeAssertionError
    }

    if (process.env.NODE_ENV !== 'production') {
      const warningMessage = getUnexpectedStateShapeWarningMessage(
        state,
        finalReducers,
        action,
        unexpectedKeyCache
      )
      if (warningMessage) {
        warning(warningMessage)
      }
    }

    let hasChanged = false
    const nextState = {}
    for (let i = 0; i < finalReducerKeys.length; i++) {
      const key = finalReducerKeys[i]
      const reducer = finalReducers[key]
      const previousStateForKey = state[key]
      const nextStateForKey = reducer(previousStateForKey, action)
      if (typeof nextStateForKey === 'undefined') {
        const errorMessage = getUndefinedStateErrorMessage(key, action)
        throw new Error(errorMessage)
      }
      nextState[key] = nextStateForKey
      hasChanged = hasChanged || nextStateForKey !== previousStateForKey
    }
    return hasChanged ? nextState : state
  }
}
```
可以看到combineReducers将reducers的每个子节点（key）都获取出来 reducerKeys
中途校验子节点，最后得到 finalReducerKeys 最终的一个节点列表。 
然后循环这个节点列表，构造state tree,以下这两句就是构造state tree的过程  
```const nextState = {}``` 与 ```nextState[key] = nextStateForKey```
但是你会发现，在createStore后，执行store.getState()，也能输出完整的state tree为何呢
查看源码发现在createStore.js中有 dispatch({ type: ActionTypes.INIT })，完成初始化的过程

### 总结
这次回顾reduxr，收获的不仅是对redux原理的理解，还有学习源码的方式，因为很多源码晦涩难懂
而且读着读着就走偏，不是所有都是你关心的内容，所以认识到带着问题去阅读源码是比较高效的一种方式
下面是我这次源码阅读的思路
#### 问题驱动型阅读源码方式
在重新阅读redux 中文文档的时候，有了以下的疑问，发现文档理也没有深入说明所以就带着疑问驱动去阅读框架的源码

- 理解redux的数据流提出疑问 
- 对store tree 形成的好奇探究 => combineReducer
- 对dispatch 匹配规则的深究 =>  createStore.dispatch 


