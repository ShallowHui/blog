---
title: Mybatis入门
date: 2021-01-23 19:02:44
tags: 
    - Mybatis
    - Spring
categories: Java
cover: https://cdn.jsdelivr.net/gh/shallowhui/cdn/top_img/mybatis.jpg
top_img: https://cdn.jsdelivr.net/gh/shallowhui/cdn/top_img/mybatis.jpg
description: Mybatis是一个强大的ORM(Object/Relational Mapping，即对象关系映射)框架，可以很好地将Java实体类型映射为数据库中的关系表，并将数据持久化。
---
## Mybatis简介

百度百科：

> MyBatis是一款优秀的持久层框架，它支持定制化SQL、存储过程以及高级映射。MyBatis避免了几乎所有的JDBC代码和手动设置参数以及获取结果集。MyBatis可以使用简单的XML或注解来配置和映射原生信息，将接口和Java的POJOs(Plain Ordinary Java Object，普通的Java对象)映射成数据库中的记录。

**Mybatis框架也是一个ORM框架，使用ORM框架后，应用程序不再直接访问底层数据库，而是以面向对象的方式来操作持久化对象(Persisent Object，PO)，而ORM框架则会通过映射关系将这些面向对象的操作转换成底层的SOL操作。**

当前的ORM框架有很多，常见的有Hibernate、Mybatis。我们选择的是Mybatis，因为Mybatis在国内开发中更加常见，并且Mybatis更加适合复杂和需要SQL性能优化的项目。

### Mybatis官方

[Mybatis's Github](https://github.com/mybatis)

可以在github上面查看Mybatis官方的各种项目和文档，并下载release版本的mybatis3使用。当然，如果是用`Maven`构建项目的话，通过坐标就可以引入Mybatis框架了。

## Mybatis的工作原理

**Mybatis是一种框架，本质上还是对JDBC的一层封装。**

### Mybatis的核心组件

+ SqlSessionFactory：SQL连接的会话工厂，用来和数据库之间创建SQL会话的。

```java
InputStream inputStream = Resources.getResourceAsStream("配置文件"); //读取Mybatis配置文件
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream); //单例模式创建会话工厂
```

+ SqlSession：一个SQL会话对象，就是用来执行SQL语句的。

```java
SqlSession sqlSession = sqlSessionFactory.openSession();
sqlSession.insert("SQL语句");
sqlSession.close();
```

### 工作流程

1. Mybatis首先读取配置文件，配置文件中配置了数据源（可以是DBCP、Hikari等等）以及其它的Mybatis相关配置信息。
2. 读取映射文件，映射文件是Mybatis中十分重要的文件，其中定义了映射关系和要执行的SQL语句。
3. 创建SqlSessionFactory，即会话工厂。
4. 获得SqlSession对象，就可以执行SQL语句了。
5. 在Mybatis底层，SqlSession是通过一个**Executor**接口来操作数据库的。
6. Mybatis在底层设置了**MappedStatement**对象，其就是对映射信息的封装。

## Mybatis与Spring的整合

Mybatis与Spring的整合需要一个中间jar包：

[mybatis-spring.jar](https://mvnrepository.com/artifact/org.mybatis/mybatis-spring)

然后在Spring的IoC容器中添加Mybatis的SqlSessionFactory组件：

```xml
    <!-- 读取数据库配置 -->
    <context:property-placeholder location="classpath:db.properties"></context:property-placeholder>
    <!-- 配置dbcp2数据池 -->
    <bean id="dataSource" class="org.apache.commons.dbcp2.BasicDataSource">
        <property name="driverClassName" value="${jdbc.driver}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
        <property name="maxTotal" value="${jdbc.maxTotal}"/>
        <property name="maxIdle" value="${jdbc.maxIdle}"/>
        <property name="initialSize" value="${jdbc.initialSize}"/>
    </bean>
    <!-- 开启事务管理 -->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>
    <tx:annotation-driven transaction-manager="transactionManager"/>
    <!-- 配置Mybatis -->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <property name="configLocation" value="classpath:mybatis-config.xml"/>
    </bean>
    <!-- 配置Mybatis的映射扫描器 -->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value=""/>
    </bean>
```

+ 其中数据源就不用在Mybatis的配置文件中配了，直接在Spring的配置文件中配置。

## Mybatis的日志功能

Mybatis拥有日志功能，可以对SQL语句的执行过程和结果进行日志记录和打印，具体的使用可以参考下面的链接：

[Mybatis日志](https://www.w3cschool.cn/mybatis/ogqn1im5.html)

## Mybatis的开发方式

在与Spring整合后，我们使用Mybatis不用通过SqlSession之类的了，而是通过映射接口，接口中可以定义各种各样的SQL相关的方法，查询结果就封装到方法的返回值中。

基于映射接口，我们有两种开发方式如下。

### 纯注解的开发方式

如果用纯注解的方式，那就可以不用编写映射文件了，直接就在映射接口的各个方法上添加注解，来定义映射关系和要执行的SQL语句:

```java
import org.apache.ibatis.annotations.Insert;
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Options;
import org.apache.ibatis.annotations.Select;

@Mapper //可以用MapperScannerConfigurer来扫描，不加Mapper注解
public interface CityMapper {

  @Insert("INSERT INTO city (name, state, country) VALUES(#{name}, #{state}, #{country})")
  @Options(useGeneratedKeys = true, keyProperty = "id")
  void insert(City city);

  @Select("SELECT id, name, state, country FROM city WHERE id = #{id}") //要执行的SQL语句
  City findById(long id);

}
```

Mybatis提供了各种各样的注解来完成映射关系的定义。

### 编写映射文件进行开发

使用纯注解的方式固然方便，但如果编写一些复杂的SQL语句会很麻烦。所以，我们还是需要编写映射文件来进行开发。

**这里顺便说一句，这两种开发方式是可以同时混合使用的，不是只能用一种方式。**

#### 映射文件模板

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="映射接口的全类名">

    <select id="映射接口中的方法名" resultType="返回的查询结果映射为Java实体类型的全类名">
        SQL语句
    </select>

</mapper>
```

+ 注意`<mapper>`标签的`namespace`属性一定要写正确完整的全类名，这样才能让映射接口和映射文件绑定。

**通常来说，一个映射文件就对应并操作数据库中的一张关系表，这个映射文件就完成一个实体类型到一个关系表的映射。**

#### 映射文件常用的标签及其属性

+ `<select>`标签：定义一个用来查询的SQL语句

`<select>`标签中常用的属性有：

|属性|作用|
|-------|-------|
|id|表示命名空间中的唯一标识符，常与命名空间组合起来使用。组合后如果不唯一，MyBatis会抛出异常。|
|parameterType|该属性表示传入SQL语句的参数类的全类名或者别名。这是一个可选属性，因为MyBatis可以通过`TypeHandler`推断出具体传入语句的参数。|
|resultType|SQL语句执行完后返回的全部数据对应映射的类的全类名或者别名。如果返回多条记录，那么应该是对应映射的类本身，不用考虑集合类型。|
|resultMap|表示外部`resultMap`的命名引用。指定SQL语句返回的类型可以使用resultType或resultMap。|

+ `<insert>`、`<update>`、`<delete>`标签：见名知义~

+ `<sql>`标签：定义可重用的SQL语句片段：

```xml
<sql id= "customerColumns">id,username,jobs,phone</sql>
...
<select id="findCustomerByld" parameterType="int" resultType="club.zunhuier.po.Customer">
	select <include refid="customerColumns"/>
	from customer
	where id = #(id)
</select>

<!-- SQL语句中用#(参数名)来取出映射接口方法中的参数 -->
```

+ `<resultMap>`标签：最重要的的一个标签，可以定义映射关系，被别的标签使用：

```xml
<resultMap type="指定要映射的实体类型" id="唯一标识">
	<id property="实体类型中的属性" column="数据库中关系表的列名"/>	<!-- 指定主键 -->
	<result property="name" column="t_name"/>	<!-- 其它字段的映射关系 -->
	<result property="age" cOlumn="t_age"/>
	<association property="类型为其它实体类的属性" />	<!-- 用于一对一关联 -->
	<collection property="" />	<!-- 用于一对多关联 -->
</resultMap>
```

#### 动态SQL语句

动态SQL是Mybatis的强大特性之一，它允许在映射文件中像写有逻辑的程序一样，对传过来的参数进行判断，符合条件的SQL语句就拼接起来，动态地生成最终要执行的SQL语句。

常用的标签及其作用如下：

|标签|作用|
|-------|-------|
|`<if>`|判断语句，用于单条件分支判断。|
|`<choose>`、`<when>`、`<otherwise>`|相当于java程序中的switch...case...default语句，用于多条件分支判断。|
|`<foreach>`|循环语句，常用于in语句等列举条件中。|
|`<where>`、`<trim>`、`<set>`|辅助元素，用于处理一些SQL语句拼装、特殊字符问题。|

例子：

```xml
<select id="findCustomerByNameOrJobs" parameterType="club.zunhuier.po.Customer" resultType="club.zunhuier.po.Customer">
	<!-- 一定执行的SQL语句 -->
	select * from t customer where 1=1
	<choose>
		<when test="username !=null and username !=" ">
			<!-- 拼接上模糊查询的SQL语句 -->
			and username like concat('%' , #{username} , '%')
		</when>
		<when test="jobs !=null and jobs !=''">
			and jobs= #{jobs}
		</when>
		<otherwise>
			and phone is not null
		</otherwise>
	</choose>
</select>
```

### Mybatis关联映射

通常来说，一个映射文件对应一张表。但实际上，SQL查询经常是多表查询，即查询出来的结果是多张表联合查询出来的结果。

数据库中的表存在一对一、一对多、多对多的关系，这是我们学过的关系数据库基本理论。

这些关系反映在Java代码的实体类型中如下：

```java
//一对一
class A {
	B b;
}

class B {
	A a;
}

//一对多
class A {
	List<B> b;
}

class B {
	A a;
}

//多对多
class A {
	List<B> b;
}

class B {
	List<A> a;
}
```

+ **比如用户和订单的关系是一对多的，查询一个用户下的所有订单会返回多条记录结果，需要将用户信息封装到一个实体类型的相应属性中，然后这个实体类型中还有一个集合类型的订单类属性，用来封装多条订单记录。称这个实体类型是复杂的，即一个实体类型中包含其它实体类型。**

那么，如何在一个映射文件中实现SQL多表查询返回的结果，可以被正确地映射到相应的实体类型？

这就要用到上面介绍`<resultMap>`标签时，出现的`<association>`和`<collection>`标签。

+ `<association>`标签：映射一对一关系：

```xml
<!-- 嵌套结果 -->
<association property="card" javaType="club.zunhuier.po.Card">
	<id property="id" column="card_id" />
	<result property="code" column="card_code" />
</association>
```

+ `<collection>`标签：映射一对多关系：

```xml
<!-- 嵌套结果 -->
<collection property="items" ofType="club.zunhuier.po.Item">
	<id property="id" column="item_id" />
	<result property="time" column="item_time" />
</collection>
```

## 总结

Mybatis的功能十分强大，需要深入体会Mybatis的映射原理。