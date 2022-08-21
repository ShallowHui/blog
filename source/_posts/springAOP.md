---
title: Spring的AOP
date: 2020-08-20 20:12:18
tags:
    - Spring
    - 动态代理
categories: Java
cover: https://cdn.jsdelivr.net/gh/shallowhui/cdn/top_img/spring.png
top_img: https://cdn.jsdelivr.net/gh/shallowhui/cdn/top_img/spring.png
description: Spring两大核心特性之一的AOP，面向切面编程。本篇文章讲解了在Spirng框架中，如何通过动态代理的方式来实现AOP。
---
## 什么是AOP

AOP的全称是Aspect-Oriented-Programming，即面向切面编程。

在传统的业务代码编写中，通常都会在开始处理业务之前进行检查、事务处理，可能还有开启日志记录的操作。这些功能的代码一般单独编写，然后业务类通过继承这些功能类或者组合的方式可以实现代码的重用，这些都是可以通过面向对象编程（OOP）的模式实现的。

但是这样一来，这些功能代码的调用代码，依然会分布在各个业务方法中。如果想开启或关闭某个功能、或者修改了某个功能方法的调用，就要在所有相关的业务代码中进行修改。这就增大了开发人员的工作量，也增大了业务代码出错的概率。

为了解决这个问题，面向切面编程（AOP）的编程思想应运而生。AOP的思想就是，通过横向抽取的机制，把各个业务方法中重复的功能调用代码抽取出来，然后在程序编译或者运行的时候，再将调用代码重新应用到业务方法中需要执行的地方，专业的说法为“织入”。

![AOP的基本思想](https://cdn.jsdelivr.net/gh/shallowhui/cdn/img/springAOP/aop.png)

>这种横向抽取的机制，OOP显然是办不到的，因为OOP只能实现父子关系的纵向的代码重用。顺带一提，虽然AOP是一种新的编程思想，但它不是OOP的替代品，只是一种对OOP的补充和延伸。

**AOP的实现是通过动态代理的方式实现的。**

## 动态代理

上面讲了AOP这种新的编程思想，那它在编码中到底是如何实现的呢？

AOP通过动态代理的方式实现。代理这种设计模式可以理解为：你要访问一个对象，出于安全和封装的考虑和要求，你不是直接访问这个对象的，而是通过访问这个对象的代理对象以达到调用这个对象的目的。类似于现实中的：你要买房→房屋中介→房主。

Java中实现动态代理的方式一般有以下两种。

### JDK动态代理——实现InvocationHandler接口

首先在项目中新建club.zunhuier包，然后在包下建立UserDao接口和其实现类，UserDao实现类的对象就是我们要代理的对象。

``` java
package club.zunhuier;

/**
 * JDK动态代理的实现要求被代理对象必须实现一个或多个接口
 */
public interface UserDao {
    //下面两个方法模拟一下业务，具体在实现类中实现
    public void addUser();
    public void deleteUser();
}
```

``` java
package club.zunhuier;

/**
 * UserDao的实现类，也是要动态代理的对象
 */
public class UserDaoImpl implements UserDao {

    public void addUser(){
        System.out.println("The user is adding...");
    }

    public void deleteUser(){
        System.out.println("The user is deleting...");
    }
}
```

然后再新建一个aspect包，在aspect包下编写实现日志记录等功能的类，也就是要重用的功能代码。在AOP的思想中，我们把这种类称为切面类，一个类就是一个切面。切面类中的方法我们称为advice，或者说通知，通知用来增强被代理的对象。

``` java
package club.zunhuier.aspect;

/**
 * 切面类，增强被代理对象
 */
public class MyAspect {
    //模拟应用启动前的增强
    public void addStart(){
        System.out.println("应用正在启动中...");
    }
    //模拟日志记录增强
    public void addLog(){
        System.out.println("开启日志记录");
        System.out.println("记录应用日志中...");
    }
}
```

再新建一个JDKProxy包，包下面编写动态代理的类，用来对UserDao进行动态代理。

``` java
package club.zunhuier.JDKProxy;

import club.zunhuier.UserDao;
import club.zunhuier.UserDaoImpl;
import club.zunhuier.aspect.MyAspect;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

/**
 * 动态代理类
 */
public class JDKProxy implements InvocationHandler {
    //声明要代理的接口
    private UserDao userDao;
    //代理方法，给外界调用来获取代理后的对象
    public Object createProxy(UserDao userDao){
        this.userDao = userDao;
        //获取类加载器
        ClassLoader classLoader = JDKProxy.class.getClassLoader();
        //被代理对象实现的所有接口
        Class[] classes = userDao.getClass().getInterfaces();
        //调用newProxyInstance方法，返回代理后的对象
        return Proxy.newProxyInstance(classLoader,classes,this);
    }

    /**
     * 实现InvocationHandler的invoke方法。动态代理后的对象，其所有方法的调用都会交由invoke方法处理，在这里可以对其方法进行增强
     * @param proxy 被代理后的对象
     * @param method 代理对象要执行的方法，invoke里通过反射机制调用
     * @param args 要执行的方法的参数
     */
    public Object invoke(Object proxy, Method method,Object[] args) throws Throwable{
        //声明切面
        MyAspect myAspect = new MyAspect();
        //前置通知
        myAspect.addStart();
        Object obj = method.invoke(userDao,args);
        //后置通知
        myAspect.addLog();
        return obj;
    }
}
```

在createProxy()方法中，我们通过`newProxyInstance()`方法返回了一个代理对象，其对目标对象进行了代理。然后在invoke()方法中处理代理对象方法的调用，并进行增强。

**注意，在AOP的思想中，我们把程序执行流程的某个阶段称为连接点（Joinpoint），比如常见的连接点有：方法执行前、方法执行完后、方法抛出了异常。然后我们选择一个连接点作为“切入点”（Pointcut），将切面“插入”其中，从而实现了将“通知”织入（weaving）到目标对象的代码中。**

有不理解的地方可以参考上面AOP基本思想的图。

比如，在上面代码的invoke()方法中，我们选择了在方法执行前和方法执行完后织入通知。

最后在JDKProxy包下编写测试类。

``` java
package club.zunhuier.JDKProxy;

import club.zunhuier.UserDao;
import club.zunhuier.UserDaoImpl;

/**
 * 测试JDK动态代理
 */
public class JDKProxyTest {
    public static void main(String[] args) {
        //创建代理类的对象
        JDKProxy jdkProxy = new JDKProxy();
        //创建要代理的目标对象
        UserDao userDao = new UserDaoImpl();
        //对目标对象进行动态代理，获得代理后的增强对象。需要进行强转
        UserDao userDaoP = (UserDao) jdkProxy.createProxy(userDao);
        userDaoP.addUser();
        System.out.println("**************************************");
        userDaoP.deleteUser();
    }
}
```

运行结果如下：

![JDK动态代理](https://cdn.jsdelivr.net/gh/shallowhui/cdn/img/springAOP/JDKProxy.png)

### CGLIB代理——Code Generation Library

CGLIB代理通过非常底层的字节码技术，直接对要代理的目标类生成一个子类，并对子类增强来进行代理。所以具有代理实例运行时的高性能。

并且CGLIB代理解决了JDK代理的一个痛点，也就是由于JDK代理是对接口进行代理，所以实际要代理的目标类（对象）必须要实现一个或多个接口，这样JDK代理才能通过代理接口，来实现代理具体的接口实现类（对象）。而CGLIB代理可以直接对目标类进行代理，没有接口的限制。

CGLIB代理的具体代码实现就不演示了，实际过程跟JDK代理差不多。注意，要用CGLIB代理的话，就要引入`org.springframework.cglib`包，也就是要导入spring框架的核心包。JDK代理就不用导入第三方包。

>这里再稍微扩展一下什么是静态代理，相对于动态代理，静态代理就很容易理解和实现了，也能解释了为什么JDK动态代理是代理接口。比如说有一个接口Person，然后接口有一个实现类Baby，现在要对Baby类进行代理怎么实现呢？就是再创建一个BabyProxy类，这个类也实现了Person接口。然后在这个代理类中，有一个私有的成员变量：private Baby baby = new Baby()。接着在下面对目标类（Baby类）中的各个方法进行重写，对方法进行增强，怎么增强就看个人，至于原先的方法就可以直接调用baby.方法名(参数)。在外部需要调用Baby类的对象时，就只需创建代理后的对象就行了：Person baby = new BabyProxy()，这样就可以调用跟原Baby类中方法同名的增强方法。
>
>这种模式就是静态代理的实现。静态代理的缺点也很明显：针对不同的目标类要编写不同的代理类，代码量巨大。动态代理就可以实现一个代理类代理多个目标类，只要目标类实现同一个接口就行了。

## Spring中的AOP

上面介绍了两种动态代理的方式，而Spring中的AOP代理方式默认就是以JDK代理的方式实现。

Spring中的AOP具体实现有下面两种方式。

### 基于代理类的实现——ProxyFactoryBean

原理其实跟上面一样，都是通过编写代理类来代理目标类（对象）。只不过Spring对代理类的编写、代理对象的获取进行了封装，Spring给我们提供了一个叫`ProxyFactoryBean`的工厂实例，这个工厂就专门负责为其它的Bean创建代理对象的实例。

我们就不用自己编写代理类了，只需在Spring配置文件中编写：

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!-- 目标类的Bean -->
    <bean id="userDao" class="club.zunhuier.UserDaoImpl"/>
    <!-- 切面类的Bean -->
    <bean id="myAspect" class="club.zunhuier.aspect.MyAspect"/>
    <!-- 指定代理工厂 -->
    <bean id="userDaoProxy" class="org.springframework.aop.framework.ProxyFactoryBean">
        <!-- 指定要代理的接口 -->
        <property name="proxyInterfaces" value="club.zunhuier.UserDao"/>
        <!-- 指定目标对象 -->
        <property name="target" ref="userDao"/>
        <!-- 指定返回的代理对象是否是单例模式，默认为true -->
        <property name="singleton" value="true"/>
        <!-- 指定代理方式， true使用cglib，false(默认)使用JDK动态代理 -->
        <property name="proxyTargetClass" value="false"/>
        <!-- 指定切面 -->
        <property name="interceptorNames" value="myAspect"/>
    </bean>
</beans>
```

需要注意的是，我们在配置中只指定了切面，还没有指定切入点。我们需要在切面类中通过实现特定切入点的接口，来指定切入点：

``` java
package club.zunhuier.aspect;

import org.springframework.aop.AfterReturningAdvice;
import org.springframework.aop.MethodBeforeAdvice;

import java.lang.reflect.Method;

/**
 * 切面类，增强被代理对象
 * 要完成Spring的代理工厂配置，切面类还要实现特定的切入点的接口
 */
public class MyAspect implements MethodBeforeAdvice, AfterReturningAdvice {
    //模拟应用启动前的增强
    public void addStart(){
        System.out.println("应用正在启动中...");
    }
    //模拟日志记录增强
    public void addLog(){
        System.out.println("开启日志记录");
        System.out.println("记录应用日志中...");
    }
    //前置通知的方法
    public void before(Method method, Object[] objects, Object o) throws Throwable {
        addStart();
    }
    //后置通知的方法
    public void afterReturning(Object o, Method method, Object[] objects, Object o1) throws Throwable {
        addLog();
    }
}
```

特定的切入点接口定义在`org.springframework.aop`中，具体有哪些切入点可以自行查阅Spring文档。需要注意的是，实现环绕通知的切入点接口定义在`org.aopalliance.intercept.MethodInterceptor`中，所以还要导入第三方jar包。

然后编写测试类，直接从Spring工厂中获取代理对象的实例：

``` java
package club.zunhuier.JDKProxy;

import club.zunhuier.UserDao;
import club.zunhuier.UserDaoImpl;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

/**
 * 测试JDK动态代理
 */
public class JDKProxyTest {
    public static void main(String[] args) {
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");
        UserDao userDao = (UserDao)applicationContext.getBean("userDaoProxy");
        userDao.addUser();
        System.out.println("**************************************");
        userDao.deleteUser();
    }
}
```

运行结果：

![ProxyFactoryBean代理工厂](https://cdn.jsdelivr.net/gh/shallowhui/cdn/img/springAOP/ProxyFactorybean.png)

### 基于AspectJ的实现

AspectJ是一个基于Java的AOP框架，提供了强大的AOP功能。Spring2.0之后，AOP模块就对AspectJ进行了支持，官方也鼓励使用AspectJ来实现AOP，上面的代理工厂反倒很少使用。使用AspectJ需要额外导入一个包：`aspectjweaver.jar`。这个依赖包可以通过下面的网址进行下载：

[http://mvnrepository.com/artifact/org.aspectj/aspectjweaver](http://mvnrepository.com/artifact/org.aspectj/aspectjweaver)

AspectJ框架实现AOP的具体方式有下面两种。

#### 基于XML配置

这个就不多讲了，毕竟XML文件配置还是比较繁琐的。其实也跟上面的配置代理工厂差不多，不过是在`<aop:config>`这个标签下配置，具体实现方式可以自行查找。

需要注意的是要掌握“切入点表达式”。

#### 基于注解配置

首先在Spring的配置文件中开启包的注解扫描，以及AspectJ的注解支持：

``` xml
<!-- 扫描指定的包，使Spring注解生效，注意需要在<beans>标签中引入<context>和<aop>标签的约束信息 -->
<context:component-scan base-package="club.zunhuier"/>
<!-- 启动基于注解的AspectJ支持 -->
<aop:aspectj-autoproxy/>
```

接着在UserDaoImpl类上加一个注解：

``` java
@Repository("userDao") //告诉Spring这是一个Bean
```

然后在aspect包下新建一个切面类MyAspectAspectJ，里面的通知方法跟之前的那个切面类一样，接着用注解的形式在类中定义切入点、通知类型，以实现AOP：

``` java
package club.zunhuier.aspect;

import org.aspectj.lang.annotation.After;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.Pointcut;
import org.springframework.stereotype.Component;

/**
 * 基于注解定义的切面类
 */
@Aspect //声明这是一个切面类
@Component //还要告诉Spring这是一个Bean
public class MyAspectAspectJ {
    /**
     * 定义切入点表达式，这里的表达式的意思是匹配club.zunhuier包下的任意类的任意返回类型、任意参数的方法的执行
     * @Poincut这个注解要加到一个返回类型为void，且方法体为空的方法上。方法名就可以代表这个切入点
     */
    @Pointcut("execution(* club.zunhuier.*.*(..))")
    private void pointCut(){};

    @Before(value = "pointCut()") //声明这是一个前置通知，value参数的值为定义切入点的方法
    public void addStart(){
        System.out.println("应用正在启动中...");
    }
    @After("pointCut()") //声明这是一个后置通知，value可以省略
    public void addLog(){
        System.out.println("开启日志记录");
        System.out.println("记录应用日志中...");
    }
}
```

最后在测试类中测试，运行结果如下：

![基于注解的AspectJ实现AOP](https://cdn.jsdelivr.net/gh/shallowhui/cdn/img/springAOP/AspectAnnotation.png)

## 总结

本篇文章主要介绍了动态代理和Spring中AOP实现方式。要深入理解AOP的本质就是动态代理，并熟练掌握基于注解的AspectJ实现AOP的方式。






