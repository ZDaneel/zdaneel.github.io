---
title: "Mvc过滤器简单应用以及打印表格"
date: 2022-05-06T01:48:57+08:00
draft: false
---

不涉及mvc过滤器的具体原理，从一个简单的需求出发，谈一下我的实现思路。（还是typora写起来舒服，word写实验报告就是一种折磨）

## 一、需求

“通过设计拦截器，使系统只允许合法用户登录后才能访问，否则仅可访问首页的 Web 应用介绍信息”

需要实现的是，从首页只能进入登录页面，如果强行在ur中进入系统，就应该无法进行跳转，返回到首页

## 二、思路

我采用的是思路是将商品管理系统的路径和登录系统的路径分开，登录后的信息存入Session中，拦截器的拦截路径设置为商品路径，对没有Session的路径进行拦截

## 三、具体实现

拦截器采用的是aop思想，不会影响现有的代码，不过如果按照我的思路，就需要进行一些处理，其实应该一开始就处理好的，所以会有一个序号为0的操作

### 0. 基础处理

#### 0.0 路径处理

对于处理商品的Controller，在类的上面加上@RequestMapping("/good")，相当于这个类里的所有方法的路径前，都会有good，如一个方法上有@RequestMapping(value="/queryAll")方法，原本是…/queryAll，现在相当于…/good/queryAll

```java
@Controller
@RequestMapping("/good")
public class GoodController {
  @RequestMapping(value="/queryAll")
  public String queryAll(){...}
}
```

同理，也给登录用的Controller加一个@RequestMapping("/自定义")

#### 0.1 登录写入Session

如何在登录后，保持一个登录的状态，就需要Session（会话），拦截器需要通过Session里的信息来进行判断

登录时写入Session，我写入的是一个customer对象，名字为“customer”（也不一定要写入对象，看具体情况

```
@ResponseBody
public void doLogin(String username, String password, HttpSession session) {

    Customer customer = new Customer(username, password); 
    session.setAttribute("customer", customer);
    
    ...具体处理略...
}
```

### 1. 创建拦截器类

创建interceptor包，并在其中创建相对应的拦截器

![截屏2022-05-06 02.05.09](https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202205060205436.png)

### 2. 实现接口

设置过滤器类的一个方法是实现HandlerInterceptor接口

实现接口后需要实现的三个方法

- *prehandle()* – called before the execution of the actual handler
- *postHandle()* – called after the handler is executed
- *afterCompletion()* – called after the complete request is finished and the view is generated

这次的实现中我会用到preHandle，preHandle在controller处理请求之前进行处理，也就是说，不需要商品管理系统的Controller进行处理就会被拦截

- 返回为true，代表继续执行，也就是商品管理系统的Controller就会处理相关的请求
- 返回为false，代表中断执行

具体思路是从request中获取Session，判断是否存在之前登陆的时候写入的customer，如果不存在，就会中断执行，并被转发到首页；如果存在，就返回true，继续执行后续的操作

```java
public class LoginInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {

        //获得Session
        HttpSession userSession=request.getSession();

        //如果请求用户的属性，则跳转到首页，并阻止通过
        if(userSession.getAttribute("customer")==null) {
            request.getRequestDispatcher("/index.jsp").forward(request, response);
            return false;
        }

        return true;
    }

    @Override
    public void postHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, ModelAndView modelAndView) throws Exception {

    }

    @Override
    public void afterCompletion(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) throws Exception {

    }
}
```

### 3. 配置拦截器

实现了拦截器还是不够的，从上面的代码可以看出，类并不知道它应该去拦截什么路径

在springmvc对应的xml配置文件中进行配置

- 指定映射路径
- 注册拦截器的bean

```xml
<!--配置拦截器-->
<mvc:interceptors>
    <mvc:interceptor>
        <!--/** 包括子路径-->
        <!--/* 不包括子路径-->
        <mvc:mapping path="/good/**"/>
        <!--注册拦截器的bean-->
        <bean id="loginInterceptor" class="cs2022.interceptor.LoginInterceptor"/>
    </mvc:interceptor>
</mvc:interceptors>
```

### 4. 测试

在首页的url里输入…/good/queryAll，就不会进行跳转，而是保持在首页。登录之后就能够顺利跳转

## 四、拓展

登录之后应该还需要一个登出的功能，可以进行一个拓展（直到写博文了才想起来，我还没有实现，移除一下session的信息应该就可以了，不是很难

## 五、进阶

可以看视频、阅读博客、参与项目等等

此外，还有专门的框架技术，如shiro和spring security，从过滤器到拦截器到框架技术，逐渐实现更多更强大的功能
