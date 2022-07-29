---
title: "SpringMVC学习"
date: 2022-07-28T01:48:57+08:00
draft: false
toc: true
---

## 概述  

SpringMVC是一种基于Java、实现了Web MVC设计模式、请求驱动类型的轻量级Web框架，即使用了MVC架构模式的思想，将Web层进行职责解耦。基于请求驱动指的就是使用请求-响应模型， SpringMVC框架的目的就是帮助我们简化Web开发过程。——老师发的ppt

![image-20220216004451324](https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202202160044664.png)

MVC框架有model、view、controller三部分组成。model一般为一些基本的Java Bean，view用于相应的页面显示(视图)，controller用于处理用户的请求。

------

## 一、SpringMVC

ssm：Spring+SpringMVC+Mybatis

MVC三层架构：

- Model：模型（相当于dao和service）
  - 业务逻辑
  - 保存数据的状态（持久层）
- View：视图（相当于之前的jsp）
  - 显示页面
- Controller：控制器（相当于之前Servlet层）
  - 取得表单数据
  - 调用业务逻辑
  - 转向指定页面

优点：

- 轻量级，简单易学
- 高效
- 与Spring兼容好
- 约定优于配置
- 功能强大：RESTful、数据验证、格式化、本地化、主题等
- 简洁灵活

围绕DispatcherServlet设计（调度Servlet）作用是将请求分发到不同的处理器（注解声明）

用doService来处理req和resp

## 二、配置文件

#### springmvc-servlet.xml配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans 
       http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!--处理映射器-->
    <bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping"/>

    <!--处理器适配器-->
    <bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter"/>

    <!--视图解析器-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" id="internalResourceViewResolver">
        <!--前缀-->
        <property name="prefix" value="/WEB-INF/jsp/"/>
        <!--后缀-->
        <property name="suffix" value=".jsp"/>
    </bean>
</beans>
```

## 三、DispatcherServlet

### web.xml

```xml
<servlet>
    <servlet-name>springmvc</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <!--关联一个springmvc配置文件-->
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:springmvc-servlet.xml</param-value>
    </init-param>
    <!--启动级别-->
    <load-on-startup>1</load-on-startup>
</servlet>

<!--匹配所有请求-->
<servlet-mapping>
    <servlet-name>springmvc</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

### 1. 注册

注册DispatcherServlet

### 2. 关联

关联springmvc的配置文件springmvc-servlet.xml

### 3. 匹配请求

匹配所有的请求——完成调度

/ 是匹配所有请求，但不包括.jsp

/*匹配所有请求，包括.jsp

## 四、Controller

### 1. Controller类

（繁琐写法）

继承Controller接口

```java
public class HelloController implements Controller {
    @Override
    public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
        //模型和视图
        ModelAndView mv = new ModelAndView();

        //封装对象
        mv.addObject("msg", "HelloSpringMVC");
        //封装要跳转的视图
        mv.setViewName("hello");
        return mv;
    }
}
```

### 2. jsp

取出msg的信息

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
    ${msg}
</body>
</html>
```

### 3. 注册Bean

注册bean

```xml
<!--Handler-->
<bean id="/hello" class="cs2022.controller.HelloController"/>
```

### 4. 测试

路径后加入hello，即可跳出HelloSpringMVC

注意：

如果出现404，可能是配置tomcat的时候，没有在工件里添加lib目录，需要去添加lib目录才能运行成功。

也可能是Handler的bean里的id错误

## 五、执行原理

图片来自狂神MVC

<img src="https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202204121451879.png" alt="image-20220412145124898" style="zoom:50%;" />

![image-20220412145332012](https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202204121453454.png)

实线为SpringMVC处理的

虚线为自己编写



### 1. DispatcherServlet

接受所有请求并拦截

路径最后的hello相当于控制器

```xml
<servlet>
    <servlet-name>springmvc</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <!--关联一个springmvc配置文件-->
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:springmvc-servlet.xml</param-value>
    </init-param>
    <!--启动级别-->
    <load-on-startup>1</load-on-startup>
</servlet>

<!--匹配所有请求-->
<servlet-mapping>
    <servlet-name>springmvc</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

### 2. HandlerMapper

处理器映射，DispatcherServlet调用，根据url查询Handler（在bean里）

```xml
<!--Handler-->
<bean id="/hello" class="cs2022.controller.HelloController"/>
```

### 3. HandlerExcution

表示具体的Handler，主要作用是根据url查找控制器

```xml
<!--处理映射器-->
    <bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping"/>
```

### 4. HandlerExcution传递回DispatcherServlet

将控制器传给DispatcherServlet，进行接下去的操作——？模糊

### 5. HandlerAdapter

表示处理器适配器，按照特定的规则去执行之前拿到的控制器——？模糊

特定的规则：去找Controller，如hello找到HelloController

```xml
 <!--处理器适配器-->
    <bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter"/>
```

### 6. Controller

Handler让具体的Controller执行handleRequest方法，如HelloController

```java
public class HelloController implements Controller {
    @Override
    public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
        //模型和视图
        ModelAndView mv = new ModelAndView();
      
        //调用业务层
        //...

        //封装对象
        mv.addObject("msg", "HelloSpringMVC");
        //封装要跳转的视图
        mv.setViewName("hello");
        return mv;
    }
}
```

### 7. 执行信息返回给HandlerAdapter

Controller执行后的得到的数据，如ModelAndView，返回给HandlerAdapter

### 8. HandlerAdapter传递给DispatcherServlet

HandlerAdapter将视图逻辑名或模型传递给DispatcherServlet

### 9. ViewResolver解析

DispatcherServlet调用视图解析器（ViewResolver）解析视图逻辑名

```xml
 <!--视图解析器-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" id="internalResourceViewResolver">
        <!--前缀-->
        <property name="prefix" value="/WEB-INF/jsp/"/>
        <!--后缀-->
        <property name="suffix" value=".jsp"/>
    </bean>
```

### 10. 解析信息传递给DispatcherServlet

解析的视图逻辑名传递给DispatcherServlet

### 11. DispatcherServlet调用视图

DispatcherServlet根据解析的结果，调用具体的视图

### 12. 展示

把视图view呈现给用户

## 六、注解开发

### 1. 引入依赖

SSM全家桶了属于是

```xml
<dependencies>
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>javax.servlet-api</artifactId>
        <version>4.0.1</version>
    </dependency>
    <dependency>
        <groupId>org.apache.tomcat</groupId>
        <artifactId>jsp-api</artifactId>
        <version>6.0.53</version>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>1.18.22</version>
        <scope>provided</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>5.3.17</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>5.3.17</version>
        <scope>test</scope>
    </dependency>
    <!--spring-jdbc-->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-jdbc</artifactId>
        <version>5.3.17</version>
    </dependency>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.13.2</version>
        <scope>test</scope>
    </dependency>
    <!--Mybatis-->
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.5.9</version>
    </dependency>
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis-spring</artifactId>
        <version>2.0.7</version>
    </dependency>
    <!--mysql-->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.28</version>
    </dependency>
</dependencies>
```

### 2. 配置web.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

  <servlet>
    <servlet-name>springmvc</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <!--关联一个springmvc配置文件-->
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath:springmvc-servlet.xml</param-value>
    </init-param>
    <!--启动级别-->
    <load-on-startup>1</load-on-startup>
  </servlet>

  <!--匹配所有请求-->
  <servlet-mapping>
    <servlet-name>springmvc</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>
</web-app>
```

### 3. SpringMVC配置文件

此处就需要考虑注解的问题

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/mvc
       http://www.springframework.org/schema/mvc/spring-mvc.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd">
    <!-- 扫描web相关的bean，使得包下的注解生效，由IOC容器统一管理 -->
    <context:component-scan base-package="cs2023.controller"/>

    <!-- 开启SpringMVC注解模式，支持mvc注解驱动 -->
    <mvc:annotation-driven/>

    <!-- 静态资源默认servlet配置，让SpringMVC不处理静态资源 -->
    <mvc:default-servlet-handler/>

    <!-- 配置jsp 显示ViewResolver -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" id="internalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/views/"/>
        <property name="suffix" value=".jsp"/>
    </bean>
</beans>
```

### 4. 创建Controller

```java
@Controller
public class HelloController {
    @RequestMapping("/hello")
    public String hello(Model model) {
        //封装数据
        model.addAttribute("msg", "Hello,MVCAnnotation");

        return "hello"; //视图的名字，会被视图解析器处理
    }
}
```

## 七、Controller配置

控制器Controller

- 接口（麻烦）
- 注解（推荐）

解析用户请求

一个控制器类可以包含多个方法

```java
@Controller
public class HelloController {
    @RequestMapping("/hello")
    public String hello(Model model) {
        //封装数据
        model.addAttribute("msg", "Hello,MVCAnnotation");

        return "hello"; //视图的名字，会被视图解析器处理
    }
}
```

@Controller——被Spring托管，IOC容器自动扫描到

如果返回值为String，并且有具体页面可以跳转，就会被视图解析器解析

 @RequestMapping()——映射访问地址

- 类（class）上作用——类中所有的方法前面会有个路径
- 方法上作用——加上路径

返回的结果是视图的名称，加上配置文件的前后缀变成WEB/views/hello.jsp

如果是@RestController，就不会被视图解析器解析，返回的就是一个字符串（用于json）

## 八、RequestMapping

用于映射url到控制器类或一个特定的处理程序方法，可作用于类或者方法上。，作用类上，类中所有响应请求方法都以该地址为父路径。

```java
@Controller
@RequestMapping("/c")
public class ControllerTest {

    @RequestMapping("/t1")
    public String test(Model model) {
        model.addAttribute("msg", "HelloST");
        return "hello";
    }
}
```

不建议放在类上面，不如直接在方法上面把路径写死。

## 九、RestFul风格

传统的风格：localhost:8080/query.action?id=1（查询get）、localhost:8080/save.action（新增post）——通过链接区分，达到不同的请求效果

RestFul：localhost:8080/item/1——通过不同的请求方式来实现，可以是查询的链接，也可以是删除的链接

全是斜线来表示 ，更加简洁、高效、安全

传统写法：

```java
//http://localhost:8080/add?a=1&b=2
@RequestMapping("/add1")
public String test(int a, int b, Model model) {
    int res = a + b;
    model.addAttribute("msg", "结果为"+res);

    return "hello";
}
```

RestFul方法：

```java
//http://localhost:8080/add2/1/2
//可以限定方式请求
//@RequestMapping(value = "/add2/{a}/{b}", method = RequestMethod.GET)
//上面的一条相当于下面的GetMapping
@GetMapping("/add2/{a}/{b}")
public String test2(@PathVariable int a,@PathVariable int b, Model model) {
    int res = a + b;
    model.addAttribute("msg", "结果1为"+res);

    return "hello";
}

//相同的路径，不同的处理方式，得到的结果也不同
@PostMapping("/add2/{a}/{b}")
public String test3(@PathVariable int a,@PathVariable int b, Model model) {
    int res = a + b;
    model.addAttribute("msg", "结果2为"+res);

    return "hello";
}
```

使用注解@PathVariable

@RequestMapping有方法参数，于是就有@GetMapping等相对应的注解

- @GetMapping
- @PostMapping
- @PutMapping
- @DeleteMapping
- @PatchMapping

## 十、结果跳转

### 1. ModelAndView

页面：{视图解析器前缀} + viewName + {视图解析器后缀}

```java
public class HelloController implements Controller {
    @Override
    public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
        //模型和视图
        ModelAndView mv = new ModelAndView();
        mv.addObject("msg", "HelloSpringMVC");
        //封装要跳转的视图
        mv.setViewName("hello");
        return mv;
    }
}
```

### 2. ServletAPI

- HttpServletResponce输出
- HttpServletResponce重定向（sendRedirect）
- HttpServletRequest转发（getRequestDispatcher）

```java
@RequestMapping("/m/t1")
public String test1(HttpServletRequest request, HttpServletResponse response) {
    HttpSession session = request.getSession();
    System.out.println(session.getId());
    return "hello";
}

@RequestMapping("/m/t2")
public String test2(Model model) {
    model.addAttribute("msg", "转发成功");
    return "forward:/WEB-INF/views/hello.jsp";
}

@RequestMapping("/m/t3")
public String test3() {
    return "redirect:/index.jsp";
}
```

其中下面两个test2和3需要把视图解析器注释

## 十一、数据处理

### 1. 接受请求

#### 1.1 域名称与参数名一致

/user/t1/?name=xxx

```java
@GetMapping("t1")
public String test1(String name, Model model) {
    //接收前段参数
    System.out.println("参数为: " + name);

    //返回结果传递
    model.addAttribute("msg", name);

    //跳转视图
    return "hello";
}
```

#### 1.2 域名称与参数名不一致

使用@RequestParam()方法

/user/t2/?username=xxx

```java
@GetMapping("t2")
public String test2(@RequestParam("username") String name, Model model) {
    //接收前段参数
    System.out.println("参数为: " + name);

    //返回结果传递
    model.addAttribute("msg", name);

    //跳转视图
    return "hello";
}
```

#### 1.3 提交对象

可以接收三个参数

定义对应的实体类

user/t3?id=4&name=zhang&age=19

```java
@GetMapping("/t3")
public String test3(User user, Model model) {
    //接收前段参数
    System.out.println("参数为: " + user.toString());

    //返回结果传递
    model.addAttribute("msg", user.toString());

    //跳转视图
    return "hello";
}
```

### 2. 数据回显

#### 2.1 ModelAndView

很少用

#### 2.2 Model

上面用的几乎都是Model

相比于ModelMap，就是一个精简版，平常只需要用这个即可

#### 2.3 ModelMap

继承了LinkedHashMap，拥有其全部功能

## 十二、乱码问题

使用表单提交时，get方法正常，post方法出现乱码

过滤器解决乱码

```java
@WebFilter("/*")
public class EncodingFilter implements Filter {
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        request.setCharacterEncoding("utf-8");
        response.setCharacterEncoding("utf-8");
        chain.doFilter(request, response);
    }
}
```

使用了@WebFilter注解，不需要再去web.xml注册

/不过滤.jsp，/*才能全部过滤全部

或者SpringMVC也提供了一个过滤器，在web.xml里配置

```xml
<!--过滤器-->
<filter>
  <filter-name>encoding</filter-name>
  <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
  <init-param>
    <param-name>encoding</param-name>
    <param-value>utf-8</param-value>
  </init-param>
</filter>
<filter-mapping>
  <filter-name>encoding</filter-name>
  <url-pattern>/*</url-pattern>
</filter-mapping>
```

## 十三、JSON

轻量级数据交换格式

JSON健值对

### 1. js端

```js
<script type="text/javascript">
    let user = {
        name:"张三",
        age:18
    };

    //转换为json
    let s = JSON.stringify(user);
    console.log(s)

    //json转换回对象
    let parse = JSON.parse(s);
    console.log(parse)
</script>
```

{"name":"张三","age":18}

### 2. java端

Jackjson（fastjson一堆问题据说是

#### 2.1 导jar

```xml
<!--jackson-->
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.13.2.2</version>
</dependency>
```

#### 2.2 配置springmvc

基础配置，先配置web.xml，再配置springmvc-servlet.xml

#### 2.3 使用

不需要走视图解析器

- 方法一：使用@ResponseBody
- 方法二：使用@RestController

```java
@RestController 
//@Controller
public class UserController {

    //@RequestMapping(value = "/json/j1", produces = "application/json;charset=utf-8")
    @RequestMapping(value = "json/j1")
    //@ResponseBody //就不会走视图解析器，会返回一个字符串
    public String json1() throws JsonProcessingException {
        User user = new User("李四", 34);

        //jackson里面的ObjectMapper
        ObjectMapper mapper = new ObjectMapper();

        //返回json格式数据
        return mapper.writeValueAsString(user);
    }
}
```

但会出现乱码问题，原始的解决方案是

```java
@RequestMapping(value = "/json/j1", produces = "application/json;charset=utf-8")
```

springmvc提供了统一解决乱码的方法StringHttpMessageConverter转换配置

```xml
<!-- 开启SpringMVC注解模式，支持mvc注解驱动 -->
<mvc:annotation-driven>
    <!--json乱码问题配置-->
    <mvc:message-converters>
        <bean class="org.springframework.http.converter.StringHttpMessageConverter">
            <constructor-arg value="UTF-8"/>
        </bean>
        <bean class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter">
            <property name="objectMapper">
                <bean class="org.springframework.http.converter.json.Jackson2ObjectMapperFactoryBean">
                    <property name="failOnEmptyBeans" value="false"/>
                </bean>
            </property>
        </bean>
    </mvc:message-converters>
</mvc:annotation-driven>
```

## 十四、连接池c3p0

常用连接池

- dbcp：半自动化操作，不能自动连接
- c3p0：自动化操作
- Druid
- Hikari

```xml
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
    <property name="driverClass" value="${jdbc.driver}"/>
    <property name="jdbcUrl" value="${jdbc.url}"/>
    <property name="user" value="${jdbc.name}"/>
    <property name="password" value="${jdbc.password}"/>
    <!--c3p0的私有属性-->
    <property name="maxPoolSize" value="30"/>
    <property name="minPoolSize" value="10"/>
    <!--关闭连接后不自动commit-->
    <property name="autoCommitOnClose" value="false"/>
    <!--获取连接超时时间-->
    <property name="checkoutTimeout" value="10000"/>
    <!--当获取连接失败重试次数-->
    <property name="acquireRetryAttempts" value="2"/>
</bean>
```
