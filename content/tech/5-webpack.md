---
title: "[VUE]5-webpack"
date: 2022-07-31T13:13:44+08:00
draft: false
toc: true
---

## 一、认识webpack

[中文文档](https://webpack.docschina.org/concepts/)

官方解释：现代的JS应用的静态**模块打包**工具（module bundler）

核心概念是模块化与打包

### 1. 模块化

详细可见[[59.12-前端模块化]]

需要处理模块间的依赖、以及打包

webpack中CSS、image、json文件都可以看作是模块来使用

### 2. 打包

各种资源模块进行打包合并成一个或多个包（Bundle）

打包可以对资源进行处理，压缩图片、scss转成css等等

其他的打包软件（如gulp）核心是task，配置完task、定义处理的事务后，就可以执行。没有用到模块化、依赖简单就可以用gulp来打包（更强调前端流程模块化）

webpack更加强大

## 二、 安装

自身依赖node环境，node中有各种包，通过npm（node packages manager）这个工具管理node下面的包

mac下[使用详情](https://www.jianshu.com/p/bc1369e250ba)

#### 1. 安装Node.js

#### 2. mac安装webpack

直接使用npm install -g webpack 报错，解决方式：

```bash
sudo npm install webpack -g --unsafe-perm=true --allow-root
```

#### 3. 安装webpack-cli

```bash
sudo npm install webpack-cli -g --unsafe-perm=true --allow-root
```

## 三、起步

dist文件夹：distribution，打包完成后放到这个文件夹里

src文件夹：开发的文件夹

index.html也需要放入dist（之后会自动生成的）

main.js作为入口

项目文件夹中打开终端，对main.js进行打包

```bash
//下面的成功
webpack ./src/main.js  --output-filename  bundle.js --output-path ./dist --mode development
```

接着只需要index.html中引用bundle.js即可

```html
<script src="dist/bundle.js"></script>
```

使用commonJs和ES6的模块化都可以成功运行

## 四、配置

#### 1. 初始化

生成package.json文件（注意不能用中文名！！！）

```bash
npm init -y
```

#### 2. 引入依赖

```bash
npm install
```

#### 3. 安装到目录

从全局把webpack安装到当前的目录中

```bash
npm install --save-dev webpack 
npm install --save-dev webpack-cli
```

其中的package.json中的devDependencies为开发时的依赖

```json
"devDependencies": {
  "webpack": "^5.69.1",
  "webpack-cli": "^4.9.2"
}
```

#### 4. 根目录下创建文件

创建webpack.config.js，即webpack的配置文件

从node的包里引入path，.resolve把两个字符串进行拼接，__dirname获取当前路径（绝对路径）

目前选择开发模式（不选择会有个warning）

```js
const path = require('path')

module.exports = {
  entry: './src/main.js',
  output: {
    path: path.resolve(__dirname, './dist'),
    filename: "bundle.js"
  },
  mode: 'development'
}
```

#### 5. 打包映射

执行webpack就可以完成打包，为了方便（而且优先可以本地找webpack）在package.json中设置就可以完成映射

```json
"scripts": {
  "test": "echo \"Error: no test specified\" && exit 1",
  "build": "webpack"
},
```

scripts优先在本地找，再去全局找

#### 6. 打包

```bash
npm run build
```

## 五、loader的使用

#### 1. 什么是loader

按照上面的配置，已经可以用模块化开发js文件了，但处理css，需要了解loader

loader是webpack的拓展，用来处理其他文件的转化

#### 2. 依赖

例子使用的是css，在main.js里加入

```js
require('./css/normal.css')
```

发现没有安装loader的话，就会报错

#### 3. 安装loader

安装需要的loader

```bash
npm install --save-dev css-loader
npm install --save-dev style-loader
```

#### 4. 修改配置

webpack.config.js的modules下面进行配置

```js
module: {
  rules: [{ test: /\.txt$/, use: 'raw-loader' }],
},
```

例如css和style就写成下面的（css-loader只负责加载，解析需要style-loader，负责将样式添加到DOM中）

其中的test为正则表达式，\\.是转义，$表示结尾，代表以.css结尾的文件，匹配所有的css文件

webpack时读取多个loader时，从右往左读，先用css-loader

```js
module: {
  rules: [{
    test: /\.css$/,
    use: ['style-loader', 'css-loader']
  }]
}
```

#### 5. 图片

webpack5能够直接显示了

#### 6. babel

ES6转换成ES5，cli自带babel 

先跳过不学

## 六、webpack中配置Vue

#### 1. 配置

必须要有vue的依赖，通过pm安装

```bash
npm install vue -save
```

之后引入Vue

```js
import Vue from 'vue'
```

结果报错

```bash
export 'default' (imported as 'Vue') was not found in 'vue'
```

降低到2.0的版本，报错

```bash
ReferenceError: process is not defined
```

在webpack.config.js中添加一段

```js
  resolve: {
    alias: {
      'vue': 'vue/dist/vue.js'
    }
  }
```

- runtime-only ——不能有template
- runtime-compiler ——代码中可以有template

跟老师讲得不一样，但还是很幸运弄了出来

#### 2. template和el关系

不希望修改index.html

使用template属性

template的东西会自动替换掉div#app内的东西

#### 3. Vue终极使用方案

把组件放入template中，把组件分离成单独文件，即一个.Vue文件

```js
import App from './vue/App.vue'

new Vue({
  el: '#app',
  template: `<App/>`,
  components: {
    App
  }
```

.Vue文件中将模版、script和style分离，进行封装处理

但还不能正常运行，因为没有对应的loader

- vue-loader
- vue-template-compiler

```bash
npm install vue-loader@15.4.2 vue-template-compiler@2.5.21 -save-dev
```

（进行了版本的控制

之后在webpack.config.js中配置

报错

```
Error: Cannot find module 'webpack/lib/RuleSet'
```

nodejs版本过高的因素 [解决方案](https://segmentfault.com/q/1010000040393536?_ea=152307361)

```bash
// 先行删除之前安装的依赖包
rm -rf node_modules
// 将nodeJS的版本切换到12(安装nvm)
nvm install 12
nvm use 12
// 执行依赖安装
npm install
// 启动服务
npm run dev:mp-weixin
```

因为无法安装上nvm，所以暂时放弃了

#### 4. 简写

在webpack.config.js中配置

```js
extensions: ['js', 'css', 'vue']
```

导入导出的时候就可以直接使用相关文件不带后缀名

## 七、plugin

（下面的都没有实际验证过）

插件对webpack功能进行拓展

- npm安装
- webpack.config.js中配置

#### 1. 添加版权声明

webpack自带

```js
const webpack = require('webpack')
...
...
  plugins: [
      new webpack.BannerPlugin('最终版权...')
  ]
```

#### 2. 打包html

index.html需要打包进dist文件夹中，于是发布的时候就可以直接发布了

使用HtmlWebpackPlugin

- 打包index.html
- 打包的js文件，自动通过script插入到bpdy中

（依然要注意版本的问题）

```bash
npm install html-webpack-plugin --save-dev
```

修改webpack.config.js

```js
const htmlWebpackPlugin = require('html-webpack-plugin')
...
...
  plugins: [
      new htmlWebpackPlugin({
        template: 'index.html'
      }),
  ]
```

- template表示用什么模版来生成index.html
- 删除之前output中添加的publicPath（我好像没有加过）

### 3. js压缩

（开发时候不要用，好像现在也用不到了）

版本指定为了和cli2对应（老师的版本 

```bash
npm install uglifyjs-webpack-plugin@1.1.1 --save-dev
```

```js
cosnt uglifyJsPlugin = require('uglifyjs-webpack-plugin')
...
...
  plugins: [
      new uglifyJsPlugin()
  ]
```

之后的bundle.js就变丑了

## 八、本地服务器

修改后需要编译，降低了开发效率

webpack提供的一个本地开发服务器，自动刷新显示修改后的结果

首先安装

```bash
npm install --save-dev webpack-dev-server@2.9.1
```

修改webpack.config.js

devserver

- contentBase——为哪一个文件夹提供本地服务
- port——端口
- inline——页面实时刷新
- historyApiFallback——SPA页面中，依赖HTML5的history模式（路由中详细说明）

直接运行失败，因为是局部安装的，因而指定运行

```bash
./node_modules/.bin/webpack-dev-server
//如果上面的失败，改成绝对路径
```

有更加简洁的做法

在package.json中的scripts中配置，就可以直接打开浏览器

```json
"dev": "webpack-dev-server --open"
```

之后运行

```bash
npm run dev
//据说新版用webpack server
```

## 九、配置文件分离

webpack.config.js中的开发和生成的配置进行分离

多个配置文件可以用

```bash
webpack-merge
```

进行合并

```js
const webpackMerge = require('webpack-merge')
const baseConfig = require('./base.config') //导入基础的配置

module.exports = webpackMerge(baseConfig, {相关的配置})
```

在package.json中配置

bulid和dev分别使用不同的配置文件

```json
"build": "webpack --config ./src/build/prod.config.js",
"dev": "webpack-dev-server --open --config ./src/build/dev.config.js"
```

