---
title: "Spring笔记-黑马版"
date: 2022-05-03T15:50:36+08:00
draft: true
toc: true
---
补充了黑马视频版本的笔记，黑马版相比之下更加深入，对原理的讲解也要清晰很多。

## 一、概述

IoC和AOP为内核，full-stack轻量级开源框架

体系结构

核心容器Core Container

## 二、耦合解耦

操作数据库jdbc

```java
public class App 
{
    public static void main( String[] args ) throws SQLException {
        //1. 注册驱动
        DriverManager.registerDriver(new com.mysql.cj.jdbc.Driver());
        //2. 获取连接
        Connection conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/mybatis", "root", "123456");
        //3. 获取操作数据库的预处理对象
        PreparedStatement pstm = conn.prepareStatement("select * from user");
        //4. 执行SQL，得到结果集
        ResultSet rs = pstm.executeQuery();
        //5. 遍历结果集
        while (rs.next()){
            System.out.println(rs.getString("name"));
        }
        //6. 释放资源
        rs.close();
        pstm.close();
        conn.close();
    }
}
```

### 耦合

程序间的依赖关系

- 类之间的依赖
- 方法间的依赖

### 解耦

降低程序间的依赖关系

new com.mysql.cj.jdbc.Driver()的独立性很差，在编译期间就会产生依赖

实际开发中需要：编译期不依赖，运行时才依赖

- 思路一：使用反射来创建对象，避免使用关键字

改成

```java
Class.forName("com.mysql.cj.jdbc.Driver");
```

运行时才发生错误

- 思路二：读取配置文件来获取要创建对象的全限定类名

## 三、工厂模式

创建bean对象的工厂BeanFactory

Bean：可重用组件

JavaBean != 实体类，JavaBean范围大很多——用Java编写的可重用组件

- 配置文件配置
  - 唯一标识=全限定类名（key=value）（限定类名，就是类名全称，带包路径的用点隔开
  - 类型为xml或者properties（或yml）
- 反射读取配置文件内容创建对象

### 实现步骤

#### 1. 配置文件

bean.properties

配置中的为实现类

```properties
accountDao=org.example.dao.impl.AccountDaoImpl
accountService=org.example.service.impl.AccountServiceImpl
```

#### 2. Bean工厂

```java
/**
 * Bean工厂
 */
public class BeanFactory {

    private static Properties props;

    /*
      初始化prop
     */
    static {
        try {
            //实例化对象
            props = new Properties();
            //读取配置文件
            InputStream in = BeanFactory.class.getClassLoader().getResourceAsStream("bean.properties");
            props.load(in);
        } catch (Exception e) {
            System.out.println("初始化配置失败");
            e.printStackTrace();
        }
    }

    /**
     * 反射获取实例
     * @param beanName bean名字
     * @return 实体类
     */
    public static Object getBean(String beanName) {
        Object bean = null;

        try {
          	//读取bean的全限定类名
            String beanPath = props.getProperty(beanName);
            bean = Class.forName(beanPath).getDeclaredConstructor().newInstance();
        } catch (Exception e) {
            System.out.println("获取实例失败");
            e.printStackTrace();
        }

        return bean;
    }
}
```

#### 3. 调用Bean工厂

注释的为最开始的高耦合写法

```java
//private final IAccountDao accountDao = new AccountDaoImpl();
private final IAccountDao accountDao = (IAccountDao) BeanFactory.getBean("accountDao");
```

```java
//IAccountService as = new AccountServiceImpl();
IAccountService as = (IAccountService) BeanFactory.getBean("accountService");
```

### 存在问题

使用循环创建，目前是多例

![截屏2022-07-11 16.01.08](https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202207111601133.png)

#### 1. 多例vs单例

多例即每次都创建一个新的实例，成员每次都会初始化

单例即只被创建一次，类中成员只会初始化一次

```java
bean = Class.forName(beanPath).getDeclaredConstructor().newInstance();
```

这句话表示每次都会用默认构造器创建，应该要存起来——>使用容器Map

#### 2. 改进后

```java
/**
 * Bean工厂
 */
public class BeanFactory {

    //定义一个配置对象
    private static Properties props;

    //定义一个Map，用于存放创建的对象，称为容器
    private static Map<String,Object> beans;

    /*
      初始化prop
     */
    static {
        try {
            //实例化对象
            props = new Properties();
            //读取配置文件
            InputStream in = BeanFactory.class.getClassLoader().getResourceAsStream("bean.properties");
            props.load(in);
          
            //实例化容器
            beans = new HashMap<>();
            //取配置文件的keys
            Enumeration<Object> keys = props.keys();
            //遍历枚举
            while (keys.hasMoreElements()) {
                //取出key
                String key = keys.nextElement().toString();
                //key获取value
                String beanPath = props.getProperty(key);
                //反射创建对象
                Object value = Class.forName(beanPath).getDeclaredConstructor().newInstance();
                //存入map容器
                beans.put(key, value);
            }
        } catch (Exception e) {
            System.out.println("初始化配置失败");
            e.printStackTrace();
        }
    }

    /**
     * 反射获取实例
     * @param beanName bean名字
     * @return 实体类
     */
    public static Object getBean(String beanName) {
        return beans.get(beanName);
    }
```

之后打印：

![截屏2022-07-11 16.18.08](https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202207111618069.png)

## 四、IoC

```java
//private final IAccountDao accountDao = new AccountDaoImpl();
private final IAccountDao accountDao = (IAccountDao) BeanFactory.getBean("accountDao");
```

IoC（控制反转）：放弃了自己new的能力，将控制权交给工厂来处理

## 五、创建bean方式

### 1. 默认构造函数

配置文件中使用bean标签，加上id和class属性，通过id找到class，反射创建class对应的文件

类中没有默认构造函数，则无法创建

### 2. 普通工厂中方法

使用普通工厂中的方法创建对象

factory-bean指定普通工厂的bean，factory-method指定工厂里的方法

### 3. 使用静态方法

class指定一个类，factory-method指定工厂里的静态方法

## 六、作用范围

- singleton：单例（默认值）
- prototype：多例
- request：作用于web应用的请求范围
- session：作用域web应用的会话范围
- global-session：作用于集群环境的会话范围（多个服务器共享一个session）

## 七、生命周期

### 1. 单例

和容器的生命周期相关

- 出生：当容器创建时
- 活着：容器存在
- 死亡：容器销毁

### 2. 多例

- 出生：当使用对象时创建
- 活着：使用过程中（Spring不知道何时要销毁
- 死亡：对象长时间不用，且没有对象引用时，GC回收

## 八、注入

### 1. 依赖注入

Dependency Injection（DI）

#### 1.1 构造函数注入

使用的标签：constructor-arg

标签出现的位置，bean标签的内部

标签中的属性

- type：用于指定要注入的数据的数据类型，该数据类型也是构造函数中某个或某些参数的类型
- index：用于指定要注入的数据给构造西数中指定素引位置的参数赋值。索引的位置是从e开始
- name：用于指定給构造函数中指定名称的参数赋值

以上三个用于指定给构造函数中哪个参数赋值

- value: 用于提供基本类型和String类型的数据
- ref：用于指定其他的bean类型数据。指的就是在spring的Ioc核心容器中出现过的bean对象

优势：

在获取bean对家时，注入数据是必须的操作，否则对象无法创建成功

弊端：

改变了bean对家的实例化方式，使我们在创建对家时，如果用不到这些数据，也必须提供

#### 1.2 set方法注入

涉及的标签：property

出现的位置：bean标签的内部

标签的属性

- name：用于指定注入时所调用的set方法名称
- value：用于提供基本类型和String类型的数据
- ref：用于指定其他的bean类型数据。

优势：

创建对象时没有明确的限制，可以直接使用默认构造函数

弊端：

如果有某个成员必须有值，则获取对象是有可能set方法没有执行



复杂类型具体参考狂神版

List结构注入的标签：list、array、set

Map结构注入的标签：map、props

#### 1.3 注解提供

开启注解使用aop的jar包

告知spring在创建容器时要扫描的包

```xml
<!--略约束-->
<context:component-scan base-backage="需要扫描包的全限定名"/>
```

- 创建对象

@Component（@Repository、@Service、@Controller）

作用：将当前类对象存入spring容器

属性：value用于指定bean的id，默认为当前类名，首字母小写

- 注入数据

@Autowired：自动按照类型注入byType，使用@Qualifer指定名称（就是bean的id）

@Resource：自动按照名称注入byName

![image-20220712164656460](https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202207121845761.png)

@Value：用于注入基本类型和String类型的值

- 改变作用范围

@Scope：用户指定bean的作用范围

value来指定，默认还是单例

- 生命周期（了解）

@PreDestory：用于指定销毁方法

@PostConstruct：用于指定初始化方法

## 九、代理

生产+销售售后的分离，代理商负责销售售后，买家通过代理商购买或者售后产品

### 基于接口的代理（JDK）

[如何理解动态代理? - bravo1988的回答 - 知乎](https://www.zhihu.com/question/40536038/answer/658146278)

具体参考作者对应的小册文章

#### 1. 引入

场景：对一些方法统一添加操作，如事务、日志

解决方法：

#### 2. 直接修改

- 不符合开闭原则。即好的程序应该对拓展开放，对修改关闭。
- 修改量大
- 代码重复
- 不利于维护

#### 3. 静态代理

编写一个**代理类**，实现与目标对象相同的接口，在代理类中维护一个目标对象的引用

- 抽取为接口

```java
public interface Calculator {
    // 加
    public int add(int a, int b);
    // 减
    public int subtract(int a, int b);
    
}
```

- 创建目标类实现接口

```java
public class CalculatorImpl implements Calculator{
    @Override
    public int add(int a, int b) {
        return a + b;
    }

    @Override
    public int subtract(int a, int b) {
        return a - b;
    }
}
```

- 创建代理类实现接口

```java
public class CalculatorProxy implements Calculator{

    //内部维护一个目标对象引用
    private Calculator target;

    //构造函数传入目标对象
    public CalculatorProxy(Calculator target) {
        this.target = target;
    }

    @Override
    public int add(int a, int b) {
        System.out.println("add方法开始");
        int result = target.add(a, b);
        System.out.println("add方法结束");
        return result;
    }

    @Override
    public int subtract(int a, int b) {
        System.out.println("sub方法开始");
        int result = target.subtract(a, b);
        System.out.println("sub方法结束");
        return result;
    }
}
```

使用代理类内创建实现类来实现静态代理

```java
@Test
void contextLoads() {
    Calculator calculator = new CalculatorProxy(new CalculatorImpl());
    int add = calculator.add(2, 4);
    System.out.println(add);

    int subtract = calculator.subtract(5, 7);
    System.out.println(subtract);
}
```

得到结果：

<img src="https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202207141344537.png" alt="截屏2022-07-14 13.44.31" style="zoom:50%;" />

核心问题如下：

- 依然存在重复代码
- 只能接收指定好的目标类

只是简单将直接修改的方法，转移到了代理类上（解耦过程）

改进思路：传入接口+增加的代码，自动返回代理对象

#### 4. 动态代理

需要有Class类的概念（反射部分）

不使用new，直接用反射获取类来创建

代理只有接口，那怎么才能有类？

最重要的是**增强代码+目标对象**，本质上代理对象只要有方法申明即可

通过**目标类实现的接口**来完成动态代理



测试接口和实现类的构造器和方法

- **接口Class对象没有构造方法**，所以Calculator接口不能直接new对象
- 实现类Class对象有构造方法，所以CalculatorImpl实现类可以new对象
- 接口Class对象有两个方法add()、subtract()
- 实现类Class对象除了add()、subtract()，还有从Object继承的方法



接口没有构造器，不能直接new，于是就将其中的**方法**给到另一个Class，再加上**构造器**

**动态代理本质：用接口Class造出一个代理类Class**

Proxy.getProxyClass(类加载器，接口的Class对象)：返回代理类的Class对象

```java
Class<?> proxyClass = Proxy.getProxyClass(Calculator.class.getClassLoader(), Calculator.class);
//查看类型
System.out.println(CalculatorImpl.class.getName());
System.out.println(proxyClass.getName());
//打印代理Class对象的构造器和方法
Constructor<?>[] constructors = proxyClass.getConstructors();
System.out.println("构造器");
printClassInfo(constructors);
System.out.println("\n");
Method[] methods = proxyClass.getMethods();
System.out.println("方法");
printClassInfo(methods);
System.out.println("\n");
```

注意：Proxy有两个包，一个是JDK自带的，一个是Spring框架里的

结果为

```
cs2020.proxy_yuque.demo.CalculatorImpl
jdk.proxy2.$Proxy79
构造器
jdk.proxy2.$Proxy79(java.lang.reflect.InvocationHandler)


方法
add(int,int)
equals(java.lang.Object)
toString()
hashCode()
subtract(int,int)
newProxyInstance(java.lang.ClassLoader,[Ljava.lang.Class;,java.lang.reflect.InvocationHandler)
getProxyClass(java.lang.ClassLoader,[Ljava.lang.Class;)
getInvocationHandler(java.lang.Object)
isProxyClass(java.lang.Class)
wait(long,int)
wait()
wait(long)
getClass()
notify()
notifyAll()
```

接下去使用构造器来创建代理实现类

```java
Class<?> calculatorProxyClass = Proxy.getProxyClass(Calculator.class.getClassLoader(), Calculator.class);
//传入JDK自带的InvocationHandler.class
Constructor<?> constructor = calculatorProxyClass.getConstructor(InvocationHandler.class);
System.out.println(constructor);//public jdk.proxy2.$Proxy79(java.lang.reflect.InvocationHandler)

//构造器执行构造方法，得到代理对象
Calculator calculatorProxyImpl = (Calculator) constructor.newInstance(new InvocationHandler() {
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        return 10086;
    }
});

//获得了方法
System.out.println(calculatorProxyImpl.add(2, 4));
```

最终并没有和预期一样输出6，而是10086

原因：每次调用代理实现类的方法（calculatorProxyImpl.add(2, 4)），JVM就会将方法导向invoke，可以理解为中间多了一层invoke，中间指代理实现类和目标类的方法中间。

于是对invoke进行改进，最low的一种做法

```java
//构造器执行构造方法，得到代理对象
Calculator calculatorProxyImpl = (Calculator) constructor.newInstance(new InvocationHandler() {
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        CalculatorImpl calculatorImpl = new CalculatorImpl();
        System.out.println("方法开始执行");
        Object result = method.invoke(calculatorImpl, args);//传入目标对象和参数
        System.out.println(result);
        System.out.println("方法执行结束");
        return result;
    }
});
```

进行改进：

```java
@SneakyThrows
@Test
void ProxyTest4() {
    CalculatorImpl target = new CalculatorImpl();
    Calculator calculator = (Calculator) getProxy(target);
    calculator.add(2, 4);
    calculator.subtract(4, 1);
}

/**
 * 传入目标对象，获取代理对象
 * @param target 目标对象
 * @return 代理对象
 */
@SneakyThrows
private static Object getProxy(Object target) {
    Class<?> proxyClass = Proxy.getProxyClass(target.getClass().getClassLoader(), target.getClass().getInterfaces());
    Constructor<?> constructor = proxyClass.getConstructor(InvocationHandler.class);
    return constructor.newInstance(new InvocationHandler() {
        @Override
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            System.out.println("方法开始执行");
            Object result = method.invoke(target, args);
            System.out.println("运算结果为: " + result);
            System.out.println("方法执行结束");
            return result;
        }
    });
}
```

抽成一个方法后更加清晰易懂

之后还能进一步优化，抽取InvocationHandler，进一步解耦

JDK提供了Proxy.newProxyInstance()，参数为类加载器、接口数组、InvovationHandler

```java
@Test
void ProxyTest5() {
    CalculatorImpl target = new CalculatorImpl();
    InvocationHandler logInvocationHandler = getLogInvocationHandler(target);
    Calculator calculator = (Calculator) Proxy.newProxyInstance(
            target.getClass().getClassLoader(),
            target.getClass().getInterfaces(),
            logInvocationHandler);
    calculator.add(2, 22);
}

/**
 * 日志增加代码
 * @param target 目标对象
 * @return InvocationHandler
 */
private static InvocationHandler getLogInvocationHandler(final Calculator target) {
    return new InvocationHandler() {
        @Override
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            System.out.println("方法开始执行");
            Object result = method.invoke(target, args);
            System.out.println("运算结果为: " + result);
            System.out.println("方法执行结束");
            return result;
        }
    };
}
```

### 基于子类的代理（cglib）

使用第三方jar包，cglib

使用方法和JDK版大同小异

报错需要在运行配置的VM option里添加--add-opens java.base/java.lang=ALL-UNNAMED

```java
public static void main(String[] args) {
    final Producer producer = new Producer();
    Producer cglibProducer = (Producer) Enhancer.create(producer.getClass(), new MethodInterceptor() {
        /**
         * 执行目标对象的方法就会经过该方法
         * @param o 代理对象本身
         * @param method 对象方法
         * @param objects 方法参数
         * @param methodProxy 当前执行方法的代理对象（一般用不上）
         * @return 方法执行
         * @throws Throwable 异常
         */
        @Override
        public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
            System.out.println("我是代理哈哈哈");
            return method.invoke(producer, objects);
        }
    });
    cglibProducer.saleProduct(100);
}
```

## 十、AOP

### 1. AOP

面向切面编程

- 减少重复代码
- 维护方便
- 提高开发效率

使用动态代理实现

### 2. Spring中的AOP

配置分为注解和xml

术语：

- Joinpoint（连接点）：业务层接口里的方法，通过方法加上增强代码

- Pointcut（切入点）：对哪些Joinpoint进行拦截，即选择部分方法加上增强代码
- Advice（通知/增强）：拦截到Joinpoint后要做的事情

通知类型：前置、后置、异常、最终、环绕

指的是在method.invoke()之前、之后、catch中、fianlly中，整个invoke方法执行就是环绕通知，在环绕通知中有明确的切入点方法通知

- Introduction（引介）：特殊通知，不修改类代码前提下，动态添加（目前无用）
- Target（目标对象）：即要被代理的对象
- Weaving（织入）：增强应用到目标对象来创建新的代理对象的过程
- Proxy（代理）：织入处理后，结果代理类
- Aspect（切面）：切入点和通知的结合，配置增强代码何时执行

#### XMl文件头

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:p="http://www.springframework.org/schema/p" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
     http://www.springframework.org/schema/beans/spring-beans-4.1.xsd
     http://www.springframework.org/schema/context
     http://www.springframework.org/schema/context/spring-context-4.1.xsd
     http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.1.xsd">

</beans>
```

具体配置

1. 通知的Bean交给Spring管理
2. aop:config标签表明开始AOP的配置
3. aop:aspect标签表明配置切面

   - id：切面唯一标识

   - ref：指定通知类bean的id
4. 在aop:aspect内部使用对应的标签，指明使用的位置，即通知的类型
   - method：用于指定切面里的哪个方法
   - pointcut：指定切入点，对业务层的哪些方法进行加强

<img src="https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202207151636919.png" alt="截屏2022-07-15 16.35.58" style="zoom: 50%;" />

切入表达式写法：

- 关键字：execution(表达式)

- 表达式：访问修饰符 返回值 包名…类名.方法名(参数列表)

- 写法案例：

  ```
  execution(public void cs2023.service.impl.AccountServiceImpl.saveAccount())
  ```

- 表达式简化

  - 访问修饰符可以省略

    ```
    void cs2023.service.impl.AccountServiceImpl.saveAccount()
    ```

  - 返回值可以使用通配符表示任意

    ```
    * cs2023.service.impl.AccountServiceImpl.saveAccount()
    ```

  - 包名可以使用通配符表示任意包，几级的包就写几个*

    ```
    * *.*.*.AccountServiceImpl.saveAccount()
    ```

    或者再使用..表示当前包及其子包

    ```
    * *..AccountServiceImpl.saveAccount()
    ```

  - 类名和方法名都使用*实现通配

    ```
    * *..*.*()
    ```

    参数列表当前为空，只会匹配空参列表的函数

  - 参数基本类型直接写名称，引用类型写包名.类名的方式 java.lang.String，使用*表示有参数

    ```
    * *..*.*(int)
    ```

  - ..表示匹配任意的参数，全通配符：

    ```
    * *..*.*(..)
    ```

- 项目通常的写法

  切到业务层实现类下的所有方法

  ```
  * cs2023.service.impl.*.*(..)
  ```

### 3.常用通知类型

- 前置通知——before：切入点方法执行之前
- 后置通知——after-returning：切入点方法正常执行之后
- 异常通知——after-throwing：切入点方法执行产生异常之后
- 最终通知——after：无论切入点方法是否正常执行都会在后面执行

#### 配置信息

```xml
<!--前置通知-->
<aop:before method="beforePrintLog" pointcut="execution(* cs2023.service.impl.*.*(..))"/>
<!--后置通知-->
<aop:after-returning method="afterReturnPrintLog" pointcut="execution(* cs2023.service.impl.*.*(..))"/>
<!--异常通知-->
<aop:after-throwing method="afterThrowPrintLog" pointcut="execution(* cs2023.service.impl.*.*(..))"/>
<!--最终通知-->
<aop:after method="afterPrintLog" pointcut="execution(* cs2023.service.impl.*.*(..))"/>
```

#### 正常运行

```
========Logger类打印前置通知before========
执行了保存
========Logger类打印后置通知afterReturn========
========Logger类打印最终通知after========
```

#### 出现异常

```
========Logger类打印前置通知before========
执行了保存
========Logger类打印异常通知afterThrowing========
========Logger类打印最终通知after========

java.lang.ArithmeticException: / by zero
```

### 4. 切入点表达式

```xml
<!--表达式切入点-->
<aop:pointcut id="pc1" expression="execution(* cs2023.service.impl.*.*(..))"/>
<!--最终通知-->
<aop:after method="afterPrintLog" pointcut-ref="pc1"/>
```

如果写在aop:aspect外面，代表着所有切面可用；根据约束，必须在使用的aop:aspect之前

### 5. 环绕通知

配置之后，切入点方法没有执行，只有环绕方法执行

原因：动态代理的环绕通知有明确的切入点方法调用

解决：Spring提供了一个接口ProceedingJoinPoint，里面有个方法proceed()，相当于明确调用切入点方法

```java
public Object aroundPrintLog(ProceedingJoinPoint proceedingJoinPoint) {
    Object rtValue = null;
    try {
        System.out.println("========Logger类打印环绕前置========");
        Object[] args = proceedingJoinPoint.getArgs();
        rtValue = proceedingJoinPoint.proceed(args);
        System.out.println("========Logger类打印环绕后置========");
    } catch (Throwable t) {
        System.out.println("========Logger类打印环绕异常========");
        throw new RuntimeException(t);
    } finally {
        System.out.println("========Logger类打印环绕最终========");
    }
    return rtValue;
}
```

### 6. 基于注解的AOP

xml文件

```xml
    <!--配置spring创建容器要扫描的包-->
    <context:component-scan base-package="cs2023"/>

    <!--spring开启aop的注解-->
    <aop:aspectj-autoproxy/>
```

aop类

大致用法和xml配置相似

```java
/**
 * 记录日志的工具类
 */
@Component
@Aspect//表示为切面类
public class Logger {

    @Pointcut("execution(* cs2023.service.impl.*.*(..))")
    private void pointcut(){}

    /**
     * 前置通知
     */
    @Before("pointcut()")
    public void beforePrintLog(){
        System.out.println("========Logger类打印前置通知before========");
    }

    /**
     * 后置通知
     */
    @AfterReturning("pointcut()")
    public void afterReturnPrintLog(){
        System.out.println("========Logger类打印后置通知afterReturn========");
    }

    /**
     * 异常通知
     */
    @AfterThrowing("pointcut()")
    public void afterThrowPrintLog(){
        System.out.println("========Logger类打印异常通知afterThrowing========");
    }

    /**
     * 最终通知
     */
    @After("pointcut()")
    public void afterPrintLog(){
        System.out.println("========Logger类打印最终通知after========");
    }
}
```

存在问题，输出顺序有错误，所以建议使用环绕通知

```
========Logger类打印前置通知before========
执行了保存
========Logger类打印最终通知after========
========Logger类打印后置通知afterReturn========
```

不使用xml的话，java配置类里面加一个注解@EnableAspectAutoProxy
