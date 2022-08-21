---
title: SpringMVC入门
date: 2020-10-14 15:50:12
tags: 
    - SpringMVC
    - Spring
categories: Java
cover: https://cdn.jsdelivr.net/gh/shallowhui/cdn/top_img/spring.png
top_img: https://cdn.jsdelivr.net/gh/shallowhui/cdn/top_img/spring.png
description: Spring是SSM框架的基石，而Spring本身提供的SpringMVC则是实现MVC设计模式的轻量级框架，我们可以用SpringMVC来开发MVC的控制层。
---
## 概述

`MVC`这个著名的设计模式想必大家都很了解，在此不再赘述。

在传统的JavaWeb开发过程中，MVC的`C`层，也就是控制层，是用`servlet`技术开发的，servlet技术也的确是JavaWeb后端的基础，需要好好掌握。

而SpringMVC正是基于servlet技术，对servlet进行了封装的一个框架，本质上还是servlet技术。SpringMVC可以帮我们大大简化之前servlet开发模式的流程，它提供了一个前端控制器用于统一接收请求，不用我们自己去配置每个servlet映射的请求了，并且SpringMVC支持多种视图技术，不再仅限于JSP技术。

SpringMVC还有很多优点，下面简单介绍如何引入框架和其工作原理。

## 引入SpringMVC

首先导入框架的依赖包，除了Spring的核心包之外，还要导入两个包：

+ spring-web.jar

+ spring-webmvc.jar

推荐通过`Maven`把依赖包导入项目中，maven的使用可以参考我的这篇文章：[Maven基础](https://zunhuier.club/2020/09/29/maven/)

配置`web.xml`文件：

``` xml

...

<!-- 配置SpringMVC前端核心控制器 -->
<servlet>
	<servlet-name>SpringMVC</servlet-name>
	<servlet-class>org.springframework.web.servlet.DispatcherServlet<servlet-class>
    <!-- 指明SpringMVC的配置文件在哪 -->
	<init-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>classpath:springmvc-config.xml</param-value>
	</init-param>
	<!-- 配置服务器启动后立即加载SpringMVC配置文件 -->
	<load-on-startup>0</load-on-startup>
</servlet>
<servlet-mapping>
	<servlet-name>SpringMVC</servlet-name>
	<!-- /:拦截所有请求，除了带.xxx文件后缀的请求 -->
	<url-pattern>/</url-pattern>
</servlet-mapping>

<!-- 配置SpringMVC自带的编码过滤器，拦截所有请求 -->
<filter>
	<filter-name>CharacterEncodingFilter</filter-name>
	<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
	<init-param>
		<param-name>encoding</param-name>
		<param-value>UTF-8</param-value>
	</init-param>
</filter>
<filter-mapping>
	<filter-name>CharacterEncodingFilter</filter-name>
	<url-pattern>/*</url-pattern>
</filter-mapping>

...

```

**从上面的配置就可以看出SpringMVC是基于servlet技术的，`DispatcherServlet`就是框架提供的一个前端控制器，它可以帮我们接收请求，我们只需编写处理请求的代码即可，也就是后端控制器。换句话说，SpringMVC对servlet的功能进行了分离，前端控制器负责接收请求，后端控制器负责处理请求。**

接着创建SpringMVC的配置文件，一般命名为`springmvc-config.xml`，放在类目录的根路径下：

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/mvc
       http://www.springframework.org/schema/mvc/spring-mvc.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd">
    <!-- 配置包扫描器，扫描controller层 -->
    <context:component-scan base-package="edu.gduf.controller" />
    <!-- 加载注解驱动 -->
    <mvc:annotation-driven />
    <!-- 配置视图解析器 -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <!-- 定义视图的前缀和后缀 -->
        <property name="prefix" value="/jsp/" />
        <property name="suffix" value=".jsp" />
    </bean>
</beans>
```

+ `<component-scan>`标签是搭配注解使用的，在SpringMVC框架中编写处理请求的代码有两种方式，一个是实现SpringMVC的核心后端控制器类，一个就是注解开发，这些在后面都会提及。

## SpirngMVC的工作原理

`DispatcherServlet`前端控制器是SpringMVC的核心类，其在框架的架构中就相当于坐镇中央进行调度，当其接收到一个请求时，就会进行如下工作流程：

1. 用户向服务器发送一个请求，请求会被前端控制器按照配置的规则拦截。

2. 前端控制器收到请求后，就会调用`HandlerMapping`即处理器映射器。

3. 处理器映射器会根据请求的URL找到具体的`处理器`也就是我们编写的后端控制器，生成处理器对象，如果有配置拦截器的话，就会把拦截器对象和处理器对象一起返回给前端控制器。

4. 前端控制器根据收到的处理器，选择合适的`HandlerAdapter`即处理器适配器。

5. 处理器适配器调用处理器，对请求进行处理。

6. 处理器处理完后，会返回一个`ModelAndView`对象给处理器适配器，处理器适配器又会把这个对象返回给前端控制器。

7. 前端控制器根据收到的`ModelAndView`对象，选择一个合适的`ViewReslover`即视图解析器。

8. 视图解析器进行解析后，根据处理器中要返回给用户的`视图`，会向前端控制器返回一个具体的视图（比如说JSP模板）。

9. 前端控制器得到具体的视图后，就会对视图进行渲染，即把数据填充至视图中。

10. 最后前端控制器就会把渲染好的视图返回给浏览器客户端。

**上面进行标记的都是SpringMVC的核心组件、核心类。**

+ SpringMVC自带一些常用的拦截器，比如上面配置`web.xml`时配置的一个编码过滤器，我们也可以自定义一个拦截器。SpringMVC的拦截器就类似于servlet的`filter`，这篇文章就不对其进行展开了。

### 实现Controller接口开发后端控制器

根据上面的SpringMVC工作流程，我们只需编写处理请求的后端控制器就行了。

我们定义一个类，实现`org.springframework.web.servlet.mvc.Controller`接口，然后实现接口定义的方法`handleRequest()`。在方法中，我们对请求进行处理，比如通过`HttpServletRequest`对象，保存要向视图填充的数据，然后指定要返回的JSP视图，最后返回一个`ModelAndView`对象。

然后在`springmvc-config.xml`配置文件中，将这个后端控制器作为一个Bean进行配置，主要配置控制器映射的请求。

听起来是不是和servlet开发模式中，要实现doGet()或者doPost()方法差不多？事实上也大致如此，像servlet中内置的request、response、session等对象在SpringMVC中都一样可以使用。

大致介绍完实现接口的开发方式，本质上和servlet开发模式并没有太大的区别，在实际开发中当然不用这种方式，而是使用注解进行开发。

### 注解开发

使用注解开发有很多好处：

+ 不再需要实现接口，只需在一个类上加一个`@Controller`注解，即可将这个类定义为后端控制器，然后配置Spring注解扫描器即可扫描到这个类。

+ 相比于接口实现类只能处理单一请求相比，使用注解的后端控制器可以通过`@RequestMapping`注解处理若干个请求。

注解开发的模板如下：

``` java

...

@Controller
public class HandleController {

    @RequestMapping(value="/请求路径")  //value属性用来指明这个方法映射的请求，当只有这个属性时，属性名可以省略
    public String handleMethod(HttpServletRequest request, HttpServletResponse response) {

        ... //处理请求的代码

        return "view.jsp"; //指明返回的视图，当在SpringMVC的配置文件中配置好了视图解析器后，可以不用加视图文件的前后缀
        // return "view";
    }

    @RequestMapping("/请求路径")
    public ModelAndView handleMethod(...) {

        ModelAndView mav = new ModelAndView();

        ... //处理请求的代码

        return mav;
    }

    ...

}
```

`@RequestMapping`注解部分属性如下：

属性名 | 属性值描述
------ | -------
value | 映射请求的地址，以项目根目录为根路径。当只有这个属性时，属性名可以省略。可以定义多个映射路径，如@RequestMapping("/{firstrequest,secondrequest}")
method | 指定方法用于处理什么类型的请求，如@RequestMapping(method={RequestMethod.GET,RequestMethod.POST})
params | 指定请求中必须包含哪些参数和其值，多个参数以`{}`数组形式指定
headers | 指定请求头中必须包含哪些参数和其值

+ **`@RequestMapping`注解可以加在方法上，也可以加在类上。加在类上并指明value属性，就相当于这个后端控制器类中的所有处理请求的方法，在自身的请求路径前面再加了一个统一的前缀路径。**

#### 请求处理方法的返回类型和参数

返回类型一般有ModelAndView、String、void。ModelAndView类型可以向SpringMVC提供的model对象中添加数据，并指定视图。但添加数据完全可以通过servlet的request等对象完成，所以一般都返回的是String类型，直接指明要返回的视图名字。

方法参数可以自定义，可以没有，也可以自己指定跟请求相关的参数。比如，servlet的request等对象。比如前端发来的请求参数，这涉及到数据绑定，后面会讲。

#### 请求重定向和请求转发

返回类型为String可以进行请求的重定向和转发：

+ 重定向：return "redirect: 视图名字或者别的请求处理方法的请求路径";

+ 转发：return "forward: 视图名字或者别的请求处理方法的请求路径";

#### @RequestBody和@ResponseBody注解

在实际开发中，有一些请求是在前端中通过`ajax`发往后端的，携带的可能是JSON格式的数据。

这两个注解就是用来处理JSON格式的数据：

``` java

...

@RequestMapping("/请求路径")
@ResponseBody
public Object handleMethod(@RequestBody Object object) {

    ... //处理请求的代码

    return object;
}

...

```

`@RequestBody`注解加在请求参数前面，指明将请求中的JSON格式数据转化为一个Java对象绑定到请求参数中。

`@ResponseBody`注解加在请求处理方法上，用于将方法返回的对象转化为JSON格式数据以对客户端进行响应。

## 数据绑定

**在SpringMVC的后端控制器中，SpringMVC会根据接收到的请求中的参数信息，以一定的方式将其进行转换并绑定到请求处理方法的参数中，这就是数据绑定。**

### 数据绑定的工作原理

1. 在Web容器中，SpringMVC会把一个封装了请求信息的ServletRequest对象传递给`DataBinder`，还会将请求处理方法的`形参对象`也传递给`DataBinder`。

2. DataBinder调用`ConversionService`组件进行数据转换、数据格式化等工作，并在最后将ServletRequest对象信息中的数据绑定到代表请求参数的对象中。

3. SpringMVC调用`Validator`组件对绑定好的数据进行检验。

4. 检验好后，返回一个`BindingResult`对象，里面封装好了数据绑定的结果。

5. 最后SpringMVC就会根据BindingResult对象中的绑定结果去调用请求处理方法，并传入相应的数据。


### 简单数据绑定

#### 绑定默认的数据类型

+ HttpServletRequest：通过request对象获得请求中的信息。

+ HttpServletResponse：通过response处理响应信息。

+ HttpSession：通过session对象获取存储在`session域`中的信息。

+ Model/Model：SpringMVC提供的一个对象，作用是将model中的数据填充到request域。

#### 绑定简单数据类型

如果在请求中有一个简单的参数，那就可以通过如下数据绑定的方式即可获取到这个参数的值：

``` java

...

@RequestMapping("/api")
public String api(int id) {   //请求处理方法名一般跟请求同名，但也无所谓

    ...

}

...

```

发送请求`http://localhost:8080/projectname/api?id=1`到服务器，后端就可以直接、正确地接收到这个名为id的参数了。

也就是说请求参数中的一些简单的数据类型，比如int，Sting，Double之类的数据，请求处理方法可以直接通过`同类型、同名`的参数直接接收请求参数。

当然，有时请求中的参数名跟请求处理方法中的参数名不一样，为此SpringMVC提供了`@RequestParam`注解：

``` java

...

@RequestMapping("/api")
public String api(@RequestParam("id") int userID) {

    ...

}

...

```

#### 绑定实体类型

当请求中有多个简单数据类型的参数要传到后端时，如果这些参数都是用来描述一个实体的话，我们可以创建一个实体类，把请求参数都封装进去。然后在请求处理方法中，直接用这个实体类作为参数即可完成对这些请求参数的数据绑定：

``` java

...

@RequestMapping("/api")
public String api(Object object) {

    ...

}

...

```

SpringMVC会自动地把这些请求参数绑定到实体类对象中同名的成员属性。**所以，请求实体类中的成员属性名字必须要跟请求参数名一致。**

#### 绑定复合实体类型

如果绑定的实体类Object中有一个成员属性是另一个实体类ObjectA的对象objectA，则相应的请求参数名设置为`objectA.param`即可完成对Object对象中的ObjectA对象中的成员属性的数据绑定。

### 复杂数据绑定

#### 绑定数组

当请求中有多个同名参数要传到后端时，比如在批量删除的请求中，多个用户的ID都以参数名为id的形式传到后端，可以通过数组的形式完成对这些同名请求参数的数据绑定：

``` java

...

@RequestMapping("/api")
public String api(int[] id) { //通过遍历数组即可访问每一个参数具体的值了

    ...

}

...

```

#### 绑定集合

当要传多个实体到后端时，比如要传多个完整的用户对象，我们可以通过集合的形式完成数据绑定。

首先定义一个包装类：

``` java

...

public class UserVo {

    private List<User> users;

    ...  //可以定义get、set方法

}
```

然后定义请求处理方法：

``` java

...

@RequestMapping("/api")
public String api(UserVo userList) {

    List<User> users = userList.getUsers(); //通过get方法即可获得所有的用户对象了

    ...

}

...

```

最后在请求参数中，注意要指明该参数属于集合中的第几个元素，如`users[index].username`即可。

SpringMVC会自动地把相同序列的属性一同作为一个对象，绑定到集合中相应序列的元素中。

### 自定义数据绑定

在一般情况下，上面的几种类型的数据绑定够用了，但有时会有一些数据绑定无效，比如下面的日期数据绑定。

有一个请求`http://localhost:8080/projectname/api?date=2017-04-08`要发到后端，我们自己是知道date参数的数据类型是日期型的，但SpringMVC是不会把它当做日期型数据的，只会把它当作字符串来处理。

如果在请求处理方法中定义`Date date`参数来接收date参数，是接收不到这个参数的，数据绑定无效，因为数据类型不同。

这时我们就需要自定义一个类型转换器了(`Converter`)：

``` java
package club.zunhuier.converter;

import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;

import org.springframework.core.convert.converter.Converter;

public class DateConverter implements Converter<String, Date> {  //实现Converter接口

	// 定义日期格式
    private String datePatternShort = "yyyy-MM-dd"
    private String datePatternLong = "yyyy-MM-dd HH:mm:ss";

    //实现convert方法
	public Date convert(String source) {

        // 格式化日期
		String thePattern = source.indexOf(":") == -1 ? datePatternShort:datePatternLong;
		SimpleDateFormat sdf = new SimpleDateFormat(thePattern);
		try {
			return sdf.parse(source);
		} catch (ParseException e) {
			throw new IllegalArgumentException("无效的日期格式，请使用这种格式:"+thePattern);
        }
    }
}
```

还要在SpringMVC配置文件中配置这个转换器：

``` xml

...

<!-- 配置自定义类型的数据转换器 -->
<bean id="myConversionService" class="org.springframework.context.support.ConversionServiceFactoryBean">
	<property name="converters">
		<set>
			<bean class="club.zunhuier.converter.DateConverter" />
		</set>
	</property>
</bean>

...

```

配置好后即可完成日期类型的数据绑定了。

## 总结

servlet和SpringMVC的开发模式是有很多共通之处的，需要我们好好掌握servlet。然后就是要熟练掌握SpringMVC的数据绑定，这样才能正常完成前后端的数据交互。