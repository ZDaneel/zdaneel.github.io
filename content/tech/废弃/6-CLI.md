---
title: "[VUE]6-CLI脚手架"
date: 2022-07-31T13:13:44+08:00
draft: true
toc: true
---

## 一、什么是Vue Cli

开发大型项目必须需要

Cli（Command-Line Interface）命令行界面，俗称“脚手架”

Vue Cli是官方发布的脚手架工具

快速搭建Vue开发环境以及对应的webpack配置

使用前提：

- Node 
- Webpack

## 二、使用

### 1. 安装Vue脚手架

```bash
sudo -s npm install -g @vue/cli

vue --version //检查版本（学习时已经是5.0.1）
```

-g代表global，全局安装

学习的是脚手架2和3

需要按照之前的模版

```bash
sudo -s npm install @vue/cli-init -g
```

Vue2初始化项目

```bash
vue init webpack my-project
```

Vue3以后（不知道5怎么样）

```bash
vue create my-project
```

据说可以直接图形化安装了

```bash
vue ui
```

## 三、Vue2Cli2

```bash
vue init webpack my-project
```

![image-20220302204103077](https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202203022041735.png)

之后会自动生成一个项目目录

### 1. package.json

大都是刚才在创建项目时选择的内容

### 2. build.js

Node可以直接运行js文件

rm()的rm，即remove，删除之前build过的内容

之后配置webpack

### 3. dev

搭建本地服务器

<img src="../../../../../../../Application%20Support/typora-user-images/image-20220302215046819.png" alt="image-20220302215046819" style="zoom:67%;" />

### 4. babelrc

ES6的转化

```
"browsers": ["> 1%", "last 2 versions", "not ie <= 8"]
```

市场份额大于1%，最新的两个版本，不是ie

## 四、两种模式区别

axios进行网络请求封装

### 1. 代码区别

runtime+compiler与runtimeonly的区别

只在main.js中

区别

在runtime+compiler中

```js
new Vue({
	el: "#app",
	tempate: '<App/>'
	components: {App}
})
```

而在runtimeonly中

```js
new Vue({
	el: "#app",
	render: h => h(App)
})
```

### 2. 程序运行区别

使用runtime+compiler

template -> ast -> render -> virtual DOM -> 真实DOM

使用runtimeonly（性能更好、代码量更少）

render -> virtual DOM -> 真实DOM

### 3. render函数

render代表渲染，不编译直接渲染

箭头函数 -> 相当于

```js
render: function (h) {
	return  h(App)
}
```

h作为createElement的别名（Vue生态系统的惯例）

```js
render: function (createElement) {
	return  createElement('div')
}
```

div就会替换掉挂载app里的东西

App即一个组件对象，同样可以传入，好处是不需要编译template

组件中（App.vue）的template，到了箭头函数之前，就已经被编译成render函数

由vue-template-compiler把template编译成了render函数

### 4. npm run build

![image-20220303185108398](https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202203031851363.png)

### 5. npm run dev

![image-20220303185223997](https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202203031852174.png)

## 五、Vue2Cli5

### 1. 创建

```bash
vue create my-project
```

创建比较简单，创建成功后，进入项目文件夹，输入下面的命令启动项目

```bash
npm run serve
```

main.js代码

```js
new Vue({
  render: h => h(App),
}).$mount('#app')
```

如果是Vue3的话（之后自己查找学习）

```js
createApp(App).mount('#app')
```

没有内部挂载el，使用mount来挂载

### 2. Vue ui

输入命令，就可以打开vue的项目管理器图形化界面

导入项目后就可以管理项目

例如图形化

## 六、补充

### 1. 箭头函数的基础使用

与原本的函数对比

```js
//1. 定义函数的方式function
const obj = {
  bbb: function() {

  },
  //简化函数
  aaa() {

  }
}

//3. ES6中的箭头函数
//无参
const ccc = () => {

}
```

参数和返回值问题

```js
<script>
  //1. 两个参数
  const sum = (num1, num2) => {
    return num1 + num2;
  }

  //2. 一个参数
  const power = num => {
    return num * num;
  }
</script>
```

### 2. this指向

箭头函数的this指向上一层的this，而一般函数的this指向的是window（上一层的this也有可能是windowct，也有可能是Object）
