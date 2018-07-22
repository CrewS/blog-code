title: hexo admin的使用记录
author: Crews
tags:
  - hexo
categories: []
date: 2018-07-20 16:43:00
---
## hexo admin
#### hexo在本地编辑的功能还是挺强大的，不过在多端同步问题上，可能就没有那么方便了，开始是想着自己做一套后台系统，不过发现流程真的挺复杂了，就找大佬相关的资料，发现了这个admin后台系统可能样式什么的并不能满足，不过后期可以fork代码进行编辑优化

### 开始

```bash
	// package.json 编辑 hexo-admin 为 git+ssh://fork地址
	// 目前是采用胡子哥修改过的代码 git+ssh://git@github.com:barretlee/hexo-admin.git
	$ npm i
	$ hexo server -d
	$ open http://localhost:4000/admin/
```

### 备注
- 可能遇到hexo插件里引用包版本问题，例如hexo-fs中使用 fs.SyncWriteStream node 8.0+ 弃用的API 需要更新
- server数据库出异常,需要清空缓存, 使用指令 **hexo clean**
- deploy功能的使用,需要在config中配置,后面可能针对优化
```
admin:
	deployCommand: hexo generate --deploy
```