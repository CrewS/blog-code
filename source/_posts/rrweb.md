---
title: rrweb与谷歌插件的实践记录（web录制与回放）
date: 2019-04-18 19:08:06
tags:
	- rrweb
---

## 前言

看到标题，大家会好奇，rrweb究竟是什么？[rrweb](https://github.com/rrweb-io/rrweb)是一个开源框架，目前主要用于web 屏幕录制和回放。利用rrweb，以及它future提案，将对bug收集，屏幕录制，自动化测试等等有非常大的作用。笔者看了官方对其的[说明文章](https://zhuanlan.zhihu.com/p/60639266)后，非常期待rrweb的后续发展，并且实践了一下rrweb与谷歌插件的集合，实现了对任意网页的屏幕录制与回放功能。

## 起步

通过阅读官方的[使用指南](https://github.com/rrweb-io/rrweb/blob/master/guide.zh_CN.md)，快速实现了一次屏幕录制和回放。然后思考怎么方便的实践。主要思路还是从解决用户怎么控制开启、停止、预览、存储方面考虑。一开始想着是直接在页面中添加对应的代码，发现这样的侵入性太大了，而且并非所有的用户都需要一部分功能，无效的资源则浪费，还增大用户的请求带宽。后来发现谷歌插件的开发思路非常符合我的需求。用户（开发、有问题的用户），安装谷歌插件后，通过热键录制，生成数据后可以观看回放，最后决定是否上传服务器。用户再将bug（问题）与该回放地址关联，指派给对应的开发解决问题。这样的流程非常的清晰，而且在代码层面上是毫无侵入感的，没有安装插件的用户对我们的系统是无感知的，拓展性很强不止单一系统可用，其他系统也可以使用这个插件完成屏幕录制。一处开发，处处使用。

## 主要逻辑思路

主要完成以下几个工作

- 开启录制
- 结束录制
- 数据压缩
- 数据保存（地址or服务器）
- 数据解压
- 回放录屏

通过监听F1键触发事件录制开启

```js
document.querySelector('html').addEventListener('keydown', function keydown (e) {
	if(e.keyCode === 112){
		console.log('start',chrome);
		alert('准备开始')
		start()
	}
})
function start(){
	a = rrweb.record({
		emit(event) {
			// 将 event 存入 events 数组中
			console.log('event')
			events.push(event);
		},
	});
}
```

通过监听F2键触发事件录制停止

```js
document.querySelector('html').addEventListener('keydown', function keydown (e) {
	if(e.keyCode === 113){
		console.log('stop');
		stop();
		alert('完成录制')
		save();
		e.stopImmediatePropagation();
	}
})
function stop(){
	 a && a();
}
```

数据保存,数据压缩使用来pako库，至少压缩来3-4倍大小
插件本地存储和服务器接口存储都能够实现，本地存储可以考虑chrome.storage或者是localstorage或者数据量比较大可以使用indexDB

```js
function zip(str){
	var binaryString = pako.gzip(str, { to: 'string' });
	return btoa(binaryString);
}
function save(){
	if ( events.length === 0) return;
	const data = zip(JSON.stringify(events))
	const body = JSON.stringify({ data });
	chrome.storage.local.get({data: []}, function(items) {
		const d = items.data
			const newData = body;
			d.push({data:newData, time: new Date()})
			chrome.storage.local.set({data: d}, function() {
					console.log('保存成功！');
			});
	});
	events = [];
	return;
	fetch('http://127.0.0.1:7001/review/add', {
		method: 'POST',
		headers: {
			'Content-Type': 'application/json',
		},
		body,
	});
}
```

录制数据的回放则是使用官方的播放器rrwebPlayer

```js
id = ? // 携带的ID
chrome.storage.local.get({data: []}, function(items) {
	const d = items.data
	console.log('读取成功！', d);
	const events = JSON.parse(unzip(JSON.parse(d[id].data).data));
	console.log(JSON.stringify(events))
	new rrwebPlayer({
		target: document.body, // 可以自定义 DOM 元素
		data: {
			events,
		},
	})
});
```

这样回放的数据流就简单的说明了，大家应该比较明白屏幕录制到观看回放实践的过程了。具体的代码等笔者完善后，将一次提供。


## 参考

