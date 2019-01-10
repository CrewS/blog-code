---
title: js 数组去重复的两三事
date: 2018-10-17 14:38:34
tags:
    - javascript
---
### 前言
js 数组去重复是我们经常会遇到的问题，今天再次看到有人问，第一时间就是想到es6里的set的一个特性，通过这个特性到达数组去重复的效果
可是仔细想想，并不知道底层的实现原理是怎样的，也不知道算法效率，也不知道这个的比较是 == 还是 ===
所以今天来探究一下js数组去重复

### 方法一：es6方式去重复
es6 set数据结构的特性就是 成员的值是唯一的，没有重复值
通过数组构建set数据结构的时候自动去重复，然后再转回数组的时候就完成了数组去重复的工作
```js
let arr = [1,2,34,41,1,2,34];
let set = new Set(arr);
console.log(Array.from(set));
// [1, 2, 34, 41]
```
想要知道这个去重复的效果是 == 还是 ===，则需要看set的成员值唯一性
1 与 “1” 是不同的
obj 与 obj是不同的
NaN 与 NaN是相同的（在set里）
```js
let set = new Set();
let a = NaN;
let b = NaN;
set.add(a);
set.add(b);
set // Set {NaN}
```
### 方法二 object keys的去重方式
在纯数字的数组中进行去重复的操作，可以用以下方法
只是最后需要将其再转回数字类型
这种方法的去重复
1 与 “1” 是相同的
NaN 与 NaN 是相同的
对象 与对象是相同的
```js
arr = [1,2,3,45,1]
let object = {}
arr.map(item => {
  object[item] = '';
})
console.log(Object.keys(object));
// ["1", "2", "3", "45"]
```

### 方法三： array inclues 去重方式
遍历数组将数组，判断是否存在新组中，不存在则添加
1 与 “1” 不同
NaN 与 NaN 相同
对象与对象不同
```js
arr = [1,NaN,3,45,NaN, 1, 3]
let start = Date.now()
let result = []
arr.map(item => {
  result.includes(item) ? null : result.push(item)
});
// [1, NaN, 3, 45]
```


### 测试10万数组去重复的性能时间
数组生成代码
```js
function generateRandomArray () {
  let arr = []
  let i = 100000
  while(i >= 0){
    arr.push(Math.ceil(Math.random() * 10000))
    i--
  }
  return arr
}
```
方法一时间：5ms
方法二时间：15ms
方法三时间：550ms

由次可见 1 > 2 > 3 性能效率

### continue...
疑问
- 性能是 N, logN 还是其他 值得去探究
- set 构造函数的处理方法原理

