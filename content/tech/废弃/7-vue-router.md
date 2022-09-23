---
title: "[VUE]7-路由router"
date: 2022-07-31T13:13:44+08:00
draft: true
toc: true
---

## 一、路由

### 1. 路由基础

路由是网络工程里的术语

通过互联网把信息从源地址传输到目的地

- 分配内网ip

- 猫里接收到的信息进行转发（通过映射表 Mac地址与内网ip）

### 2. 前端渲染与后端渲染

#### 2.1 后端

后端渲染：很早的时候使用JSP、PHP类似的开发，java代码从数据库中获取数据，动态放到页面中

后端路由：后端处理url和页面之间的映射关系

缺点：都是由后端来编写和维护、前端人员难以编写、各种代码混在一起

#### 2.2 前后端分离

ajax的出现，出现了前后端分离模式

后端只负责提供数据

前端把后端传来的数据给解析，并渲染到页面上

一个url对应一套的html+css+js

![IMG_F2F6615B18DC-1](https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202203041148482.jpeg)

#### 2.3 SPA页面

SPA：单页面富页面

单页面：整个网页只有一个html（index.html

开发后首先获取全部资源（都是一个html+css+js）

点击事件切换显示的页面——这时候就需要前端路由

前端路由中配置映射关系，生成url，抽取资源（对应组件），再渲染到首页

![IMG_B1B0136FA7B0-1](https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202203041353381.jpeg)

前端路由核心就是生成url，抽离资源并渲染

### 4. 请求服务器

如何做到，改变url之后，不会重新请求服务器，页面不刷新

#### 4.1 改变url的hash

起初的url为http://localhost:8080/#/

控制台指令location.hash = ‘aaa’

url变为http://localhost:8080/#/aaa 页面没有刷新

#### 4.2 history模式

是HTML5中的东西

##### 4.2.1 pushState

类似是进栈

```js
history.pushState(data. title, url)
//例子
history.pushState({}, '', 'home')
```

之后url变成了：http://localhost:8080/home 同样页面没有刷新

url显示的就是最新一个push（压入）的，即栈顶的内容

使用

```js
history.back()
```

返回上一个页面

##### 4.2.2 replaceState

```js
history.replaceState(data. title, url)
```

类似替换栈顶，无法使用back

##### 4.2.3 go

```js
history.go(?delta)
//例子
history.go(-1) //back, 即往前退一步
//如果为正，向前一步
//类似
history.go(1)
//等价于
history.forward()
```

## 二、基本使用

使用脚手架时选择安装router，即可使用

多了一个router文件夹，里面有一个index.js

手动配置路由相关的信息

### 1. 导入路由

```js
import Router from 'vue-router' //导入路由
```

### 2. 安装插件

```js
import Vue from 'vue' //导入Vue
Vue.use(Vue.use(Router)) //安装插件
```

### 3. 创建对象

```js
const routes = [

]
const router = new Router({
	//配置路由和组件之间的应用关系(下面用的是增加语法)
	routes
})
```

### 4. 传递router

在main.js中使用router，就需要导出和导入

index.js导出

```js
export default router
```

main.js导入

```js
import router from './router' //如果是文件夹，会自动找index文件
new Vue({
  el: '#app',
  router,
  render: h => h(App)
})
```

### 5. 配置映射

路径部分，@默认指向src，而../就是上一级目录，./是同一级目录

```js
import Home from '../components/Home'
import About from '@/components/About'

Vue.use(Router)

const routes = [
  {
    path: '/home', //路径
    component: Home
  },
  {
    path: '/about',
    component: About
  }
]
```

### 6. 挂载实例

在App.vue中挂载

使用router-link（源码中已经注册了全局组件）

```js
<div id="app">
  <router-link to="/home">首页</router-link>
  <router-link to="/about">关于</router-link>
</div>
```

与a标签类似，点击后url改变（但不刷新）

之后使用router-view来渲染

```js
<router-view></router-view>
```

是一个占位的东西，点那个就会在那个位置切换内容

### 7. 默认值

想要路由有默认值，即不需要点击就已经渲染了

重定向：redirect

```js
const routes = [
  {
    path: '/',
    redirect: '/home'
  }
]
```

### 8. 改变history

默认的是改变hash，如果需要使用history模式，只需要加一句

```js
export default new Router({
  routes,
  mode: 'history'
})
```

放在所有配置的最上面，为了与下面的区别开来，之后好处理

### 9. router-link补充

to属性：指定跳转路径

tag属性：指定渲染成什么组件，默认是a标签

```js
<router-link to="/home" tag="button">首页</router-link>
<router-link to="/about" tag="li">关于</router-link>
```

replace属性：默认是使用pushState，增加replace属性，就无法后退

active-class属性：

点击后，会出现router-link-active的class，从而可以处理style

改变这个属性名字

```js
active-class="active"
```

或者一起改变，在index.js文件中加一句linkActiveClass

```js
export default new Router({
  routes,
  mode: 'history',
  linkActiveClass: 'active'
})
```

### 10. 代码跳转路由

注册方法

```html
<button @click="homeClick">首页</button>
<button @click="aboutClick">关于</button>
```

调用方法this.$router.push或replace（在所有组件里注册了这个属性）

```js
methods: {
  homeClick() {
    this.$router.push('/home')
  },
  aboutClick() {
    this.$router.replace('/about')
  }
}
```

重复点击会报错，index.js加入下面这段

```js
// 解决ElementUI导航栏中的vue-router在3.0版本以上重复点菜单报错问题
const originalPush = Router.prototype.push
Router.prototype.push = function push(location) {
  return originalPush.call(this, location).catch(err => err)
}

Vue.use(Router) //原本就有的导入路由
```

### 11. 动态路由

path是不确定的，例如用户名是不一样的，需要动态改变路由的路径

在index.js中增加一个映射

```js
{
  path: '/user/:userId',
  component: User
}
```

之后路径增加一下即可，点击后会跳转到这个路径

```js
<router-link to="/user/zhangsan">用户</router-link>
```

实际中需要动态绑定（v-bind就可以

```js
<router-link :to="'/user/' + userId">用户</router-link
```

如果需要从某个路由中获取data用户名，并添加到url中（？也可以用props传

```js
{{userId($route.params.userId)}}
```

$route 对应的是active时候的route，看作是某一个路由，而router是整个路由

### 12. 懒加载

js文件太多，需要只请求一些内容（首页等）

懒加载：用到的时候再从服务器进行加载

![image-20220305144730170](https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202203051447653.png)

懒加载的方式

- 结合Vue异步组件和webpack
- AMD写法
- ES6写法（主要用这个）

```js
import('../components/Home.vue')
```

```js
const Home = () => import('../components/Home')
```

而原本的是

```js
import Home from '../components/Home'
```

## 三、嵌套路由

步骤

- 创建对应的子组件，并在路由中配置对应子路由
- 组件内部使用<router-view>

注意，组件内部使用之后，<router-link>的to需要写完整的相对路径

## 四、参数传递

params和query传递

params类型

- 配置路由格式：/router/:id
- path后面加对应的值
- 传递后形成路径/router/123

query

- 配置路由格式：/router（普通配置
- 对象中使用query的key作为传递方式
- 传递后形成路径：/router?id=123

[协议名]: //用户名:密码@主机名:端口/路径?查询参数#片段ID

一般用的是：

schme://host:port/path?query#fragment

## 五、导航守卫

主要用来通过跳转或取消的方式守卫导航

前置和后置钩子都是全局守卫

### 1. 前置钩子

动态修改标题

在index.js中进行配置

beforeEach为前置钩子（守卫）

```js
const router = new Router({
  routes,
  mode: 'history',
  linkActiveClass: 'active'
})

router.beforeEach((to, from, next) => {
  //from跳转到to
  document.title = to.matched[0].meta.title
  next()
})

//传入实例
export default router
```

### 2. 后置钩子

```js
router.afterEach((to, from) => {
  //不需要调用next函数
})
```

### 3. 其他

路由独享、组件内守卫

[官方文档](https://router.vuejs.org/zh/guide/advanced/navigation-guards.html)

## 六、keep-alive

生命周期：创建、载入、更新、销毁

跳转后原来的组件就会被销毁，再跳转回去的时候就会创建一个新的组件

keep-alive与actived和deactived有关，keep-alive里的内容不会被销毁

```js
<keep-alive><router-view></router-view></keep-alive>
```

使用exclude进行排除
