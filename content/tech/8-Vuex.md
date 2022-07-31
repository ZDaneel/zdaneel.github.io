---
title: "[VUE]8-VueX"
date: 2022-07-31T13:13:44+08:00
draft: false
toc: true
---

## 一、概述

管理状态变量

管理状态：

- 登录状态
- 多个页面共享

### 基础配置略

### VUEX的new Vuex.Store：

- #### state

```js
$store.state.counter
```

取出存放在state里的值

- #### getters

类似计算属性

可以获取state里变异的一些信息

- #### mutations

##### 1. 修改state的值

使用commit提交

```js
this.$store.commit('increment')
```

此外可以有负载，即多传一个值进行处理

count代表增加的数值

```js
this.$store.commit('incrementCount', count)
```

##### 2. Mutation提交风格

type：属性的对象

```js
this.$store.commit({
	type: 'incrementCount',
	count: count
})
```

```js
incrementCount(state, payload) {
	state.counter += payload.count
}
```

##### 3. p137响应式原理略

##### 4. 类型常量

将名字抽离出来单独形成一个常量的文件，引入常量名字

- #### actions

与后端api相关，mutations里只有同步的，

- #### modules

## 二、单一状态树

Single Source of Truth

一个项目只使用一个state
