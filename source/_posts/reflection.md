---
title: Java反射和注解
date: 2021-05-31 21:30:26
tags:
    - 反射
    - 注解
categories: Java
cover: https://cdn.jsdelivr.net/gh/shallowhui/cdn/top_img/java.jpg
description: 在程序运行的时候，可以获取某个类，某个对象的全部信息，称为反射。注解技术基于反射。
---
## 什么是反射

反射，就是在程序运行的时候，可以获取某个类，某个对象的全部信息。

## 反射的原理

我们知道，Java源代码是先编译为字节码(.class)文件后，再交由JVM去解释执行。

在Java中，除了int、float等基本数据类型之外，一切皆对象，都有自己的class，即类，代表这一类抽象数据类型。那么，在Java中，有这么一个类：`Class`，代表了Java中的其它类，你可以把`Class`理解为“类的类”。

JVM在执行Java程序的时候是动态加载的，并不是一次性把所有用到的class全部加载到内存中，而是第一次需要用到class时才加载。

**当JVM把某个类加载进内存的时候，就会为其创建一个`Class`类的实例对象，并把这个对象与加载进来的类关联起来。所以，JVM中所有的Class实例对象，都指向一个类，这个类可以是自己编写的，也可以是JDK的。**

既然JVM为每个类都创建了一个Class实例对象，那么通过这个对象，就可以访问到对应的类的全部信息了。

这种通过Class实例对象获取class信息的方法就称为反射(Reflection)。

## 反射API

### 获取具体的类对应的Class实例对象

有三种方式获取。

+ 每个类都有一个静态变量class，就是其对应的Class实例对象：

```java
Class cls = String.class;
```

+ 可以通过一个类的实例对象，获取这个类的Class实例对象：

```java
String s = "Hello，world！";
Class cls = s.getClass();
```

+ 如果知道一个类的完整类名，可以通过Class类的静态方法获取：

```java
Class cls = Class.forName("java.lang.String");
```

### 访问成员属性

通过Class实例对象获取某个类的成员属性：

```java
Field f = String.class.getDeclaredField("value");
f.getName(); //"value"
f.getType(); //class [B    //表示byte[]类型
```

可以看出需要知道具体的属性名才能访问到相应的属性。`getDeclaredField`方法返回了一个属性的封装对象，通过`Field`对象就可以获取这个属性的名字和类型了。

注意，`getDeclaredField`方法只能获取这个类自己声明的属性，还有其它方法可以获取从父类继承下来的属性，可以自行去查阅资料。

#### 获取属性值

上面，我们只是获取了某个类的成员属性，那我们想获取这个类的一个具体实例对象的属性值呢？

可以通过上面提到的`Field`对象：

```java
f.get(object); //传进去一个具体的对象就行了
```

既然可以通过`Filed`获取值，那当然也可以设置值：

```java
f.set(object, value);
```

注意，如果`Field`对象所代表的属性是`private`权限的话，直接获取、设置值会报错，需要先调用：

```java
f.setAccessible(true); //允许访问
```

### 访问成员方法

```java
Method method = cls.getDeclaredMethod(name, Class...);
```

同样返回方法的封装对象，`getDeclaredMethod`方法的参数是不定长的，第一个是指定要访问的方法名，之后是这个方法的各个参数的类型，需要传入Class实例对象。比如某个参数是String类型的，就需要传入String.class。

#### 调用方法

获取方法的封装后，我们可以在具体的实例对象上调用这个方法：

```java
String s = "Hello，world！";
// 获取String类的substring(int)方法，参数为int类型
Method m = String.class.getMethod("substring", int.class);
// 在s对象上调用该方法并获取结果，需要强制转型
String r = (String) m.invoke(s, 6);
```

对了，虽然Java中的基本数据类型没有class，但也可以通过int.class这样的方式指明。

如果方法是静态方法，那么`invoke`方法的第一个参数传入null就可以了。

### 获取构造函数

常规的实例化一个对象是通过new，那非常规的就是通过反射机制。

```java
Constructor constructor = cls.getDeclaredConstructor(Class...);
```

`getDeclaredConstructor`方法获取某个类自己声明的构造方法，参数传入构造方法的参数类型，需要传入Class实例对象。

获取了构造方法的封装对象后，就可以实例化这个类的一个具体对象了：

```java
Object object = (Object) constructor.newInstance(parameters...); //需要强制转型
```

`newInstance`方法的参数传入实际构造方法的参数就行了。

## 什么是注解

注解，可以看成是一种特殊的注释，都是用来对源代码来进行说明解释的。只不过注释会被编译器忽略，而注解，则可以被一起编译到.class文件中。

注解作为一种标记，可以放在类、属性、方法、方法参数前面，注解本身对代码的逻辑没有影响，标记了之后起什么作用，完全由谁使用这个注解决定。

注解有如下几类：

+ **由编译器使用的注解。**

例如经常见到的@Override注解，是加在方法上面的，这是告诉编译器在编译的时候检查这个方法是否正确地实现了覆写。

这类注解就不会被一起编译到.class文件中，在编译完成后就丢掉了。

+ **由一些特殊工具处理.class文件时使用的注解。**

有些工具会在加载.class文件的时候，对.class文件做动态修改，实现一些特殊的功能。

这类注解会被编译进入.class文件，但加载结束后并不会存在于内存中。这类注解只被一些底层库使用，一般开发不必理会。

+ **在程序运行期间可以读取的注解。**

这类注解会被编译进.class文件中，JVM加载.class后会存在于内存中，这时我们就可以读取这个注解了，怎么读取？

当然是上面讲的反射机制，读取之后就可以用代码来处理这个注解，实现它标记之后应有的功能。

**所以，看到这里应该可以理解了，Java的注解并不是什么黑魔法，看起来注解一标上去，就可以实现功能了。注解能实现的功能，背后都是有代码支撑的。**

## 定义注解

Java使用@interface语法来定义注解（Annotation），就像定义一个接口：

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target({ElementType.METHOD,ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
public @interface PersonalA {
    String name() default "";
}
```

注解里面可以定义参数，参数类型可以是基本数据类型、String、Class、枚举类型的数组。格式如上面的代码所示，default后面跟默认值。

**如果有一个参数在使用注解时是经常要配置的，可以将参数名设为value，这样在配置参数时，就可以直接写值，不用写成“参数名=值”这种形式了。**

### 元注解

上面的代码可以看到，定义的`PersonalA`注解上面，还标有其它注解。像这些可以修饰其它注解的注解，称为元注解。一般来说，元注解都是JDK自带的，我们不用编写元注解，只需使用就好了，元注解的使用对我们编写的自定义注解非常重要。

下面是一些常用的元注解：

+ **@Target：最常用的元注解。**

使用这个元注解去修饰其它注解，用来定义被修饰的注解可以被用在源代码的哪些位置。

下面是这个元注解的源代码：

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Target {
    /**
     * Returns an array of the kinds of elements an annotation type
     * can be applied to.
     * @return an array of the kinds of elements an annotation type
     * can be applied to
     */
    ElementType[] value();
}
```

可以看到参数是个ElementType枚举类型的数组，ElementType枚举源代码如下：

```java
public enum ElementType {
    /** Class, interface (including annotation type), or enum declaration */
    TYPE,

    /** Field declaration (includes enum constants) */
    FIELD,

    /** Method declaration */
    METHOD,

    /** Formal parameter declaration */
    PARAMETER,

    /** Constructor declaration */
    CONSTRUCTOR,

    /** Local variable declaration */
    LOCAL_VARIABLE,

    /** Annotation type declaration */
    ANNOTATION_TYPE,

    /** Package declaration */
    PACKAGE,

    /**
     * Type parameter declaration
     *
     * @since 1.8
     */
    TYPE_PARAMETER,

    /**
     * Use of a type
     *
     * @since 1.8
     */
    TYPE_USE
}
```

从注释可以看出，ElementType.METHOD说明可以被用在方法上，ElementType.TYPE说明可以被用在类或接口上。所以上面定义的`PersonalA`注解既可以被用在类或接口上，也可以被用到方法上。

+ **@Retention：定义其它注解的生命周期。**

这个元注解的参数也是枚举类型，有如下几种取值：

1. RetentionPolicy.SOURCE：仅编译器起作用
2. RetentionPolicy.CLASS：仅存在于.class文件中
3. RetentionPolicy.RUNTIME：运行期间起作用

如果一个自定义注解没有加@Retention，则默认是CLASS的，即这个自定义注解只会被编译进.class文件中，而不会被加载进内存中。

**因此，如果我们想要自定义的注解在程序运行期间可以被读取到并起作用，务必要用@Retention(RetentionPolicy.RUNTIME)去修饰它。

## 使用注解

上面我们定义了`PersonalA`注解，现在来使用它：

```java
@PersonalA(name = "zunhuier")
public class User {

    public String name;

    public User(String name) {
        this.name = name;
    }
}
```

## 处理注解

我们想要`PersonalA`注解实现这样一个功能：检查每个User对象的属性值name，是否与加在User类上面的注解参数指定的值一样。那我们编写如下处理注解的代码：

```java
public class Main {
    
    public static void main(String[] args) {
        User user1 = new User("zunhuier");
        checkUser(user1);
        User user2 = new User("zzz");
        checkUser(user2);
    }

    //根据注解检查的方法
    private static void checkUser(User user) {
        //获取User类的Class对象
        Class cls = user.getClass();
        //通过反射读取注解
        PersonalA personalA = (PersonalA) cls.getAnnotation(PersonalA.class);
        if (personalA != null) {
            String nameA = personalA.name();
            try {
                //通过反射获取实例对象的属性值
                Field name = cls.getDeclaredField("name");
                String realName = (String) name.get(user);
                if (!nameA.equals(realName)) {
                    System.out.println("成员属性值" + name.getName() + "不是" +  nameA);
                }else {
                    System.out.println("成员属性值" + name.getName() + "是" + realName);
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }

}
```

结果：

成员属性值name是zunhuier
成员属性值name不是zunhuier

## 总结

反射机制是Java的基本特性，而注解技术是基于反射的。Spring框架就大量采用了注解技术。