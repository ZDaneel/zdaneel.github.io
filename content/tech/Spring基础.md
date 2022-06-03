---
title: "Spring基础（暂缺aop）"
date: 2022-05-03T15:50:36+08:00
draft: false
---
## 一、概述

spring是一个轻量级的控制反转（IoC）和面向切面（AOP）的框架

可以用maven的pom.xml进行管理

版本写在属性里，实现更方便的版本控制

```xml
<properties>
  <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  <maven.compiler.source>1.7</maven.compiler.source>
  <maven.compiler.target>1.7</maven.compiler.target>

  <spring.version>4.3.18.RELEASE</spring.version>
</properties>

<dependencies>
    <!--Spring MVC核心包依赖：spring-core、spring-webmvc，另外加上jstl支持。-->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-core</artifactId>
      <version>${spring.version}</version>
    </dependency>
  </dependencies>

```

## 二、组成

7大模块，详细可见官网图

## 三、控制反转

### 1. 概述

[官方文档](https://docs.spring.io/spring-framework/docs/3.2.x/spring-framework-reference/html/beans.html)

IoC（控制反转）是一种设计思想，DI（依赖注入）是实现IoC的一种方式

通过Spring IoC 容器来实现

具体通过xml或注解

![img](https://docs.spring.io/spring-framework/docs/3.2.x/spring-framework-reference/html/images/container-magic.png)

### 2. XML实现

创建Spring的xml文件

原理：bean = 对象，相当于Hello hello = new Hello() 

其中的id就是实例的名称，class为路径，property相当于给属性设置值

其中的property中的value和ref区别

- value是直接给属性赋值
- ref是引用Spring容器中已经创建好的对象

```xml
<bean id="hello" class="cs2022.pojo.Hello">
    <property name="str" value="我是伞兵" />
</bean>
```

有参构造就使用constructor-arg，详情可见官方文档（下标、类型、参数名实现）

```xml
<bean  id = "exampleBean"  class = "examples.ExampleBean" > 
	<constructor-arg index="0" value = "7500000" /> 
  <constructor-arg type="java.lang.String" value = "Hello" /> 
  <constructor-arg name="name" value = "zhangs" /> 
</bean>
```

在测试类中实现

```java
public static void main(String[] args) {
    //获取Spring上下文对象
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("Beans.xml");
    //通过id取bean
    Hello hello = (Hello) applicationContext.getBean("hello");
    System.out.println(hello.toString());
}
```

对象是通过Spring创建的（Spring调用无参构造创建对象，用set注入值），不需要再用new了！

优点：只需要修改xml配置文件，不需要再修改代码

## 四、DI依赖注入环境

### 1. 构造器注入

即上面所说的通过bean下的property或者constructor-arg进行注入

### 2. Set方式注入（重点）

依赖注入：Set注入

- 依赖：bean对象的创建依赖于容器
- 注入：bean对象中的所有属性，由容器来注入

#### 2.1 复杂类型

```xml
<bean id="address" class="cs2020.pojo.Address">
    <property name="address" value="沈阳大街"/>
</bean>
<bean id="student" class="cs2020.pojo.Student">
    <!--普通值注入-->
    <property name="name" value="zou"/>

    <!--ref，Bean注入-->
    <property name="address" ref="address"/>

    <!--数组-->
    <property name="books">
        <array>
            <value>红楼梦</value>
            <value>三国演义</value>
        </array>
    </property>

    <!--list-->
    <property name="hobbies">
        <list>
            <value>看书</value>
            <value>看电影</value>
        </list>
    </property>

    <!--map-->
    <property name="card">
        <map>
            <entry key="身份证" value="22333234"/>
            <entry key="银行卡" value="666444222"/>
        </map>
    </property>

    <!--set-->
    <property name="games">
        <set>
            <value>LOL</value>
            <value>CS</value>
        </set>
    </property>

    <!--null-->
    <property name="wifi">
        <null/>
    </property>

    <!--Property-->
    <property name="info">
        <props>
            <prop key="email">12345@qq.com</prop>
            <prop key="学号">123456</prop>
        </props>
    </property>

</bean>
```

输出：

```
Student(name=zou, address=Address(address=沈阳大街), books=[红楼梦, 三国演义], hobbies=[看书, 看电影], card={身份证=22333234, 银行卡=666444222}, games=[LOL, CS], info={学号=123456, email=12345@qq.com}, wifi=null)
```

#### 2.2 真实测试对象

命名空间约束

- c命名

对应的是有参构造（不能和@Data一起用）

```xml
xmlns:c="http://www.springframework.org/schema/c"
。。。
<bean id="user2" class="cs2020.pojo.User" c:age="19" c:name="张三"/>
```

- p命名

可以直接注入属性值

```xml
xmlns:p="http://www.springframework.org/schema/p"
...
<bean id="user" class="cs2020.pojo.User" p:age="18" p:name="李四"/>
```

### 3. 作用域

scope

- 单例模式（Spring默认机制）：scope="singleton——一个bean的id只有一个实例
- 原型模式：scope="prototype"——每次从容器中get时都会产生一个对象实例

## 五、自动装配Bean

自动装配是spring满足bean依赖的一种方式

上下文中自动寻找，自动给bean装配属性

三种装配方式

- xml中显示配置
- java中显示配置
- 隐式自动装配（重要）

### 1. ByName自动装配

会自动在容器上下文查找，和自己对象set方法后面的值对应的beanid

缺陷：必须要bean的id一致

```xml
<bean id="cat" class="cs2022.pojo.Cat"/>
<bean id="dog" class="cs2022.pojo.Dog"/>

<bean id="person" class="cs2022.pojo.Person" autowire="byName">
    <property name="name" value="张三"/>
</bean>
```

### 2. ByType自动装配

会自动在容器上下文查找，和自己对象属性类型相同的bean

缺陷：只能有一个类型相同的

### 3. 注解（重要！！）

- 导入约束：context约束
- 配置注解支持

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

</beans>
```

@Autowired

直接在属性上使用即可（set方法上也可，用了这个注解可以省去set方法

通过先找ByType，再找ByName来实现

如果标注了@Autowired(required = false)，这个对象可以为null

```java
@Data
public class Person {

    @Autowired
    @Qualifier(value = "cat")
    private Cat cat;
    @Autowired
    private Dog dog;
    private String name;
}
```

@Nullable为空不报错

@Qualifier(value = "cat")指定id名——用于自动配置的环境复杂

@Resource——Java提供的注解，先名字后类型（已经被取消了）



ps. IDEA里推荐使用构造方法

“为什么推荐使用构造方法注入呢？使用构造方法注入并且在构造方法中使用断言，可以有效避免NPE，在Spring容器启动时就会发现错误，防止在使用的时候出现运行时异常。”

## 六、注解开发

首先导入配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

  	<!--指定要扫描的包，包下的注解就会生效-->
    <context:component-scan base-package="cs2022"/>
    <context:annotation-config/>

</beans>
```

### 1. bean

已经不需要了

### 2. 属性注入

```xml
<bean id="user" class="cs2022.pojo.User"/>
```

等价于@Component

在类中只需要这样就可以配置

@Value来给属性赋值

```java
@Component
public class User {
    @Value("张三")
    public String name;
}
```

### 3. 衍生的注解

@Component的衍生注解

- dao——@Repository
- service——@Service
- controllr——@Controller

四个注解的功能是一样的，都是将某个类注册到组件中

这几个注解后面也可以添加一些参数，比如比较常用的一个是注解的括号中加value属性，来表示这个组件在容器中的ID，如果不设置value值时，默认的ID是类名的全称（第一个字母小写）。

### 4. 自动装配

@Autowired

直接在属性上使用即可（set方法上也可，用了这个注解可以省去set方法

@Nullable为空不报错

@Qualifier(value = "cat")指定id名——用于自动配置的环境复杂

@Resource——Java提供的注解，先名字后类型（已经被取消了）

### 5. 作用域

@Scope(“singleton”)——单例

### 6. 小结

xml与注解：

- xml更加万能，适用任何场所
- 注解：不是自己的类使用不了，维护复杂

## 七、JavaConfig

完全不使用Spring的xml配置，交给Java来做。JavaConfig是Spring的一个子项目

### 1. 编写实体类

此处不需要添加@Component

```java
@Data
public class User {
    @Value("张三")
    private String name;
}
```

### 2. 编写Config类

@Configuration代表这是一个配置类，等价于之前的beans.xml

也会被spring容器托管，注册到容器中，本来就是个@Component



@Bean即注册一个bean，相当于之前写的bea标签

- 名字相当于bean的id
- 返回值相当于bean的class

@Bean等价于之前在实体类上的@Component

@Bean(“”)可指定id名字

```java
@Configuration
public class MyConfig {

    @Bean
    public User getUser() {
        return new User();
    }
}
```

### 3. 测试

```java
@Test
public void test() {
    ApplicationContext context = new AnnotationConfigApplicationContext(MyConfig.class);
    User getUser = context.getBean("getUser", User.class);
    System.out.println(getUser.toString());

}
```

问题：出现 has not been refreshed yet错误

原因是在Test类中ApplicationContext context = new AnnotationConfigApplicationContext();中忘记添加配置类对象。

即ApplicationContext context = new AnnotationConfigApplicationContext(MyConfig.class);



注意：（弹幕）

如果开启包扫描，加载配置类以后就可以通过反射拿到配置类中的对象了

@Bean只写在方法上，返回的是一个对象，但一般不获取已经在容器中的对象

@Bean 可以用于通过方法获取数据库连接池Connection这种对象

## 八、AOP

暂略

### 1. 静态代理

### 2. 动态代理

## 九、整合Mybatis

[官方文档](http://mybatis.org/spring/zh/index.html)

在spring的配置文件中配置数据源和sqlSessionFactory

spring-dao.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

    <!--java配置文件-->
    <context:property-placeholder location="classpath:db.properties"/>

    <!--Datasource 配置数据源-->
    <!--使用spring提供的jdbc-->
    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="${driver}"/>
        <property name="url" value="${url}"/>
        <property name="username" value="${username}"/>
        <property name="password" value="${password}"/>
    </bean>

    <!--sqlSessionFactory-->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <!--注入配置-->
        <property name="dataSource" ref="dataSource"/>
        <!--绑定配置文件-->
        <property name="configLocation" value="classpath:mybatisConfig.xml"/>
        <property name="mapperLocations" value="classpath:UserMapper.xml"/>
    </bean>

    <!--sqlSession-->
    <bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">
        <!--没有set方法，只能构造器注入-->
        <constructor-arg index="0" ref="sqlSessionFactory"/>
    </bean>
</beans>
```

注意，其中需要一个Mapper的实现类，才能注入SqlSessionTemplate

```java
public class UserMapperImpl implements UserMapper{

    //使用sqlSessionTemplate执行

    private SqlSessionTemplate sqlSession;

    public void setSqlSession(SqlSessionTemplate sqlSession) {
        this.sqlSession = sqlSession;
    }

    @Override
    public List<User> selectUser() {
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        return mapper.selectUser();
    }
}
```

之后，在主spring配置文件中导入配置，并进行注入

ApplicationContext.xml

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--引入配置-->
    <import resource="spring-dao.xml"/>

    <!--注入-->
    <bean id="userMapper" class="cs2020.mapper.UserMapperImpl">
        <property name="sqlSession" ref="sqlSession"/>
    </bean>
</beans>
```

或者使用继承类的方式实现

或者使用一种继承的方式来实现类

```java
public class UserMapperImpl2 extends SqlSessionDaoSupport implements UserMapper {
    @Override
    public List<User> selectUser() {
        SqlSession sqlSession = getSqlSession();
        return sqlSession.getMapper(UserMapper.class).selectUser();
    }
}
```

```xml
<bean id="userMapper2" class="cs2020.mapper.UserMapperImpl2">
    <property name="sqlSessionFactory" ref="sqlSessionFactory"/>
</bean>
```

此时不需要再用构造器注入sqlSession（原本是为了和实现类的sqlSession相对应）
