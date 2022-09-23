---
title: "Mybatis学习"
date: 2022-07-28T01:48:57+08:00
draft: true
toc: true
---


# Mybatis

[官方文档](https://mybatis.org/mybatis-3/zh/index.html)

## 一、简介

### 1. 什么是Mybatis

是一个持久层框架，方便与数据库进行交互。xml或注解来配置和映射

```xml
    <!--Mybatis-->
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.5.9</version>
    </dependency>
```

### 2. 持久化

持久化就是数据在持久状态和瞬时状态转化的过程

### 3. 持久层

完成持久化的代码块——dao层

### 4. 为什么用Mybatis

- 方便
- 简化jdbc操作
- 自动化
- 容易上手
- sql与代码分离

## 二、使用

### 1. 搭建环境

#### 1.1 数据库

```mysql
CREATE DATABASE `mybatis`;

USE `mybatis`;

CREATE TABLE `user`(
	`id` INT(20) NOT NULL PRIMARY KEY,
	`name` VARCHAR(30) DEFAULT NULL,
	`pwd` VARCHAR(30) DEFAULT NULL
)ENGINE=INNODB DEFAULT CHARSET=utf8;

INSERT INTO `user` (`id`, `name`, `pwd`) VALUES 
(1, '张三', '12345'),
(2, '李四', '1234'),
(3, '王五', '123')
```

#### 1.2 新建项目

新建maven项目，导入依赖

```xml
 				<!--Mybatis-->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.5.9</version>
        </dependency>
        <!--mysql-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.47</version>
        </dependency>
```

### 2. 创建模块

#### 2.1 编写核心配置文件

从 XML 中构建 SqlSessionFactory

XML 配置文件中包含了对 MyBatis 系统的核心设置，包括获取数据库连接实例的数据源（DataSource）以及决定事务作用域和控制方式的事务管理器（TransactionManager）

注意

- xml不识别的话，把文件名中的「-」删去

- 改成自己的数据库信息
- &加上amp;为转义符号，就表示一个&

```xml
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useSSL=false&amp;useUnicode=true&amp;characterEncoding=utf-8"/>
                <property name="username" value="root"/>
                <property name="password" value="123456"/>
            </dataSource>
        </environment>
    </environments>

</configuration>
```

#### 2.2 编写工具类

将获取SqlSessionFactory的方法封装成一个工具类，从SqlSessionFactory中获得SqlSession，SqlSession完全包含了数据库执行sql所需的所有方法，可以直接执行已映射的sql语句。

```java
public class MybatisUtils {

    private static SqlSessionFactory sqlSessionFactory;

    static {
        try {
            //获取对象
            String resource = "mybatisConfig.xml";
            InputStream inputStream = Resources.getResourceAsStream(resource);
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    //获得sqlSession实例
    public static SqlSession getSqlSession() {
        return sqlSessionFactory.openSession();
    }
}
```

### 3. 编写代码

（后续直接使用easyCode就可以生成

#### 3.1 编写pojo和dao

pojo要和数据库一一对应，名字不能出现偏差

```java
public interface UserDao {
    public List<User> getUserList();
}
```

#### 3.2 XML配置映射

新建一个xml项目

注意：

- namespace，也就是命名空间，要和dao接口一致
- 语句的id要和dao接口里的方法一致，此处即getUserList
- 返回结果一般用resultType，返回值写范型中的内容，即cs2022.pojo.User（路径最好要全

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--命名空间-->
<mapper namespace="cs2022.dao.UserDao">
    <select id="getUserList" resultType="cs2022.entity.User">
        select * from mybatis.user
    </select>
</mapper>
```

#### 3.3 注册mapper

在核心配置文件中的<configuration>里面注册

注意：

- 此处的UserMapper.xml与核心配置文件都在resources目录的同一级下
- 如果UserMapper.xml在其他地方，需要考虑资源过滤的问题

```xml
<!--每一个mapper.xml都需要在Mybatis核心配置文件中注册-->
<mappers>
    <mapper resource="UserMapper.xml"/>
</mappers>
```

可能存在配置文件无法被导出的问题，解决方案是在pom.xml的build中配置resources

```xml
<!--静态资源导出问题    -->
    <build>
        <resources>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <filtering>false</filtering>
            </resource>
            <resource>
                <directory>src/main/resources</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <filtering>false</filtering>
            </resource>
        </resources>
    </build>
```

#### 3.4 测试

此处使用的是try-with- resources，自动关闭sqlSession

```java
public class UserDaoTest {
    @Test
    public void test() {
        try(SqlSession sqlSession = MybatisUtils.getSqlSession()) {
            //执行SQL
            UserDao userDao = sqlSession.getMapper(UserDao.class);
            List<User> userList = userDao.getUserList();

            for (User user : userList) {
                System.out.println(user.toString());
            }
        }
    }
}
```

可能遇到的问题：

- 配置文件没有注册
- 绑定接口名错误
- 方法名不一致
- 返回类型不对
- Maven导出资源问题

## 三、CRUD

### 1. 事务

增删改需要提交事务，否则无法改变数据库的信息

```java
sqlSession.commit();
```

### 2. Map 

Map进行传递参数，在sql中取出key即可

```java
Object insertMap(Map<String, Object> map);
```

```xml
<select id="insertMap" resultType="java.lang.Integer" parameterType="map">
    insert into mybatis.user (id, name, pwd) values (#{userid}, #{username}, #{userpwd})
</select>
```

注意，在mapper接口里的参数，前面不能加上@Param

问题：

- 参数加注解的意义和使用时机
- 使用Map后的返回值情况，因为貌似出错了是传的NULL（？
- Map插入能直接提交事务（？？

### 3. 模糊

like % %

sql拼接用通配符

## 四、配置解析

### 1. 核心配置文件

即mybatis-config.xml文件，包括了很多的设置和属性信息



### 2. 环境配置

```xml
<environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${driver}"/>
                <property name="url" value="${url}"/>
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
            </dataSource>
        </environment>
    </environments>
```

环境可以有多个，通过默认的环境ID选择需要的

- 事务管理器配置——如设置JDBC，就是直接使用了JDBC的提交和回滚设置（还有managed

- 数据源配置——连接数据库（数据池POOLED的概念，即用完可以回收，一般用池
  - dbcp
  - c3p0
  - druid
  - hikari

每个数据库对应一个sqlSession实例

### 3. 属性

使用peoperty可以获取数据池的配置信息

使用${}就可以实现读取外部文件

在核心配置文件中引入，注意xml中的属性顺序，必须在environment前面

```xml
<!--引入外部配置文件-->
<properties resource="db.properties"/>
```

### 4. 别名优化

为Java类型设置一个短的名字，减少冗余

```xml
<typeAliases>
    <typeAlias type="cs2022.entity.User" alias="User"/>
    <!--没有注解，使用Bean的首字母小写；不然就用注解的信息@Alias("xxxx")-->
    <package name="cs2020.entity"/>
</typeAliases>
```

### 5. 其他配置

类型处理器、对象工厂…

还有插件

- 通用mapper
- mybatis-plus

### 6. 映射器

即mappers，在核心配置文件中进行注册

```xml
<!--每一个mapper.xml都需要在Mybatis核心配置文件中注册-->
<mappers>
    <mapper resource="UserMapper.xml"/>
</mappers>
```

还有其他一些方法，具体参考文档

## 五、结果集映射

处理实体类和数据库的字段冲突

column为数据库中的字段，property为实体类的字段

```xml
<resultMap id="UserMap" type="User">
    <result column="id" property="id"/>
    <result column="name" property="name"/>
    <result column="pwd" property="password"/>
</resultMap>

<select id="getUserList" resultMap="UserMap">
    select * from mybatis.user
</select>
```

resultMao是mybatis最强大的元素

复杂情况associatior、collection

## 六、日志

环境配置里的logImpl

- slf4j
- log4j
- STDOUT_LOGGING

…

### 1. STDOUT_LOGGING

```xml
<settings>
    <setting name="logImpl" value="STDOUT_LOGGING"/>
</settings>
```

![截屏2022-05-22 22.00.40](https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202205222200754.png)

### 2. log4j

添加依赖，用配置文件进行配置

```xml
<settings>
    <setting name="logImpl" value="LOG4J"/>
</settings>
```

log4j.properties

```xml
# suppress inspection "LossyEncoding" for whole file
#将等级为DEBUG的日志信息输出到console和file这两个目的地，console和file的定义在下面的代码
log4j.rootLogger=DEBUG,console,file

#控制台输出的相关设置
log4j.appender.console = org.apache.log4j.ConsoleAppender
log4j.appender.console.Target = System.out
log4j.appender.console.Threshold=DEBUG
log4j.appender.console.layout = org.apache.log4j.PatternLayout
log4j.appender.console.layout.ConversionPattern=[%c]-%m%n

#文件输出的相关设置
log4j.appender.file = org.apache.log4j.RollingFileAppender
log4j.appender.file.File=./log/zd.log
log4j.appender.file.MaxFileSize=10mb
log4j.appender.file.Threshold=DEBUG
log4j.appender.file.layout=org.apache.log4j.PatternLayout
log4j.appender.file.layout.ConversionPattern=[%p][%d{yy-MM-dd}][%c]%m%n

#日志输出级别
log4j.logger.org.mybatis=DEBUG
log4j.logger.java.sql=DEBUG
log4j.logger.java.sql.Statement=DEBUG
log4j.logger.java.sql.ResultSet=DEBUG
log4j.logger.java.sql.PreparedStatement=DEBUG
```

简单使用

对象为类的名字，注意导包是log4j的包

``` java
static Logger logger = Logger.getLogger("UserDaoTest");
...
    @Test
    public void log4jTest(){
        logger.info("info信息");
        logger.debug("debug信息");
        logger.error("error信息");
    }
```

或者直接用Lombok的注解@Log4j

```java
@Log4j
public class UserDaoTest {
    @Test
    public void log4jTest(){
        log.info("info信息");
        log.debug("debug信息");
        log.error("error信息");
    }
}
```

![截屏2022-05-22 22.23.37](https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202205222223396.png)

## 七、分页

pagehelper插件

### 1. 导入依赖

```xml
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper</artifactId>
    <version>5.3.0</version>
</dependency>
<dependency>
    <groupId>com.github.jsqlparser</groupId>
    <artifactId>jsqlparser</artifactId>
    <version>4.2</version>
</dependency>
```

### 2. 配置插件

```xml
<plugins>
    <!-- com.github.pagehelper为PageHelper类所在包名 -->
    <plugin interceptor="com.github.pagehelper.PageInterceptor">
        <!-- 表示使用mysql的分页方法 -->
        <property name="helperDialect" value="mysql" />
        <!-- 表示当页码长度为0 的时候,就不进行分页查询 -->
        <property name="pageSizeZero" value="true"/>
    </plugin>
</plugins>
```

### 3. 把查询条件封装

```java
public class QueryObject {
    private Integer currentPage;
    private Integer pageSize;
}
```

### 4. mapper建立分页查询

```java
public List<User> getUserLimit(QueryObject qo);
```

### 5. 在mapper中配置

```xml
<select id="getUserLimit" resultMap="UserMap">
  select * from mybatis.user
</select>
```

### 6. 编写分页的业务

核心是PageHelper.startPage();

```java
public class UserServiceImpl implements IUserService{
    private UserDao userDao;

    public UserServiceImpl(UserDao userDao) {
        this.userDao = userDao;
    }

    @Override
    public PageInfo<User> page(QueryObject qo) {
        PageHelper.startPage(qo.getCurrentPage(), qo.getPageSize());
        List<User> userList= userDao.getUserLimit(qo);
        return new PageInfo<>(userList);
    }
}
```

### 7. 测试

```java
public void pageTest(){
    try(SqlSession sqlSession = MybatisUtils.getSqlSession()) {
        //业务对象
        UserDao userDao = sqlSession.getMapper(UserDao.class);
        UserServiceImpl userService = new UserServiceImpl(userDao);
        //查询条件
        QueryObject queryObject = new QueryObject();
        queryObject.setCurrentPage(2);
        queryObject.setPageSize(2);
        //分页方法
        PageInfo<User> pageInfo = userService.page(queryObject);
        for (User user : pageInfo.getList()) {
            System.out.println(user);
        }
    }
}
```

最终能够根据查询条件的当前页面，进行自动设置limit

## 八、注解开发

mybatis一般用xml，主要取决于项目的复杂程度，简单的项目可以用注解

暂时跳过，包括执行流程

## 九、复杂环境

### 1. 多对一

在resultMap里进行设置

- 对象：association
- 集合：collection

外键获取信息：

#### 1.1 子查询

多写一个查询，association里调用

```xml
<resultMap id="BaseResultMap" type="Student">
  <id column="id" jdbcType="INTEGER" property="id" />
  <result column="name" jdbcType="VARCHAR" property="name" />
  <!--外键-->
  <association property="teacher" column="tid" javaType="Teacher"
  select="getTeacher"/>
</resultMap>

<select id="getAll" resultMap="BaseResultMap">
  select * from student
</select>
<select id="getTeacher" resultType="Teacher">
  select * from teacher where id = #{id}
</select>
```

![截屏2022-05-24 11.14.54](https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202205241114372.png)

#### 1.2 按照结果嵌套

联表查询，直接在association里嵌套一个result

```xml
<resultMap id="BaseResultMap" type="Student">
  <result column="id" jdbcType="INTEGER" property="id" />
  <result column="name" jdbcType="VARCHAR" property="name" />
  <!--外键-->
  <association property="teacher" column="tid" javaType="Teacher">
    <result property="name" column="tname"/>
  </association>
</resultMap>

<select id="getAll" resultMap="BaseResultMap">
  select s.id, s.name sname, t.name tname
  from student s, teacher t
  where s.tid = t.id
</select>
```

![截屏2022-05-24 11.21.00](../../../../../../Application%20Support/typora-user-images/%E6%88%AA%E5%B1%8F2022-05-24%2011.21.00.png)

### 2. 一对多

例：一个老师多个学生

一个客户多个订单

使用collection集合，对于其中的范型，使用ofType

讲得和乱…

```xml
<resultMap id="BaseResultMap" type="Teacher">
  <id column="ttid" jdbcType="INTEGER" property="id" />
  <result column="name" jdbcType="VARCHAR" property="name" />
  <collection property="students" ofType="Student">
    <result column="sid" property="id"/>
    <result column="sname" property="name"/>
    <result column="tid" property="tid"/>
  </collection>
</resultMap>

<select id="selectByPrimaryKey" resultMap="BaseResultMap">
  select t.id ttid, t.name tname,s.id sid, s.name sname, tid
  from mybatis.teacher t, mybatis.student s
  where t.id = s.tid and t.id = #{id}
</select>
```

```java
public static void main(String[] args) {
        try(SqlSession sqlSession = MybatisUtils.getSqlSession()) {
            //执行SQL
            TeacherDao teacherDao = sqlSession.getMapper(TeacherDao.class);
            Teacher teacher = teacherDao.selectByPrimaryKey(1);
            List<Student> studentList = teacher.getStudents();
            for (Student student : studentList) {
                log.info(student);
            }
        }
    }
```

![截屏2022-05-24 11.56.49](https://cdn.jsdelivr.net/gh/ZDaneel/cloudimg@main/img/202205241156641.png)

跳过子查询

## 十、动态sql

if、choose（when、otherwise）、trim（where、set）、foreach

### 1. if

```sql
<select id="queryBlog" resultMap="BlogMap">
    select * from mybatis.blog where 1=1
    <if test="title != null">
        and title = #{title}
    </if>
</select>
```

### 2. choose

类似switch，如果都不满足when，就执行otherwise

```xml
<select id="queryBlogChoose" resultMap="BlogMap">
    select * from mybatis.blog
    <where>
        <choose>
            <when test="title != null">
                and title = #{title}
            </when>
            <when test="author != null">
                and author = #{author}
            </when>
            <otherwise>
                and views = #{views}
            </otherwise>
        </choose>
    </where>
</select>
```

### 3. trim

where和set本质上是自定义的trim，具体可参考官方文档

#### 3.1 where

使用<where>语句，根据情况自动去除开头的and或or

```xml
<select id="queryBlog" resultMap="BlogMap">
    select * from mybatis.blog
    <where>
        <if test="title != null">
            and title = #{title}
        </if>
    </where>
</select>
```

####  3.2 set

根据情况会自动去掉最后的逗号

```xml
<update id="updateBlog" parameterType="map">
    update mybatis.blog
    <set>
        <if test="title != null">
            title = #{title},
        </if>
        <if test="author != null">
            author = #{author},
        </if>
    </set>
    where id = #{id}
</update>
```

### 4. foreach

通常在构建IN条件语句的时候

例：查询1、2、3号记录的博客

```xml
<select id="queryBlogForeach" resultMap="BlogMap">
    <include refid="select_base"/>
    <where>
        <foreach collection="ids" item="id" open="(" close=")" separator="or">
            id = #{id}
        </foreach>
    </where>
</select>
```

### 5. sql片段

将公共部分抽取出来以供复用

- <sql id=""></sql>
- 引用时用<include refid=""/>

```xml
<sql id="select_base">
  select * from mybatis.blog
</sql>
```

最好单表定义

不要有<where>标签
