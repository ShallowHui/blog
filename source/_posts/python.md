---
title: Python的高级特性
date: 2020-04-06 12:03:45
tags: Python高级
categories: Python
cover: https://cdn.jsdelivr.net/gh/shallowhui/cdn/top_img/python.jpg
top_img: https://cdn.jsdelivr.net/gh/shallowhui/cdn/top_img/python.jpg
description: Python的高级语法、特性都会在这篇文章中介绍，比如Python的函数嵌套，Python的装饰器、迭代器，等等。
---
## 函数的嵌套定义

Python函数可以嵌套定义，即在函数体内再定义另一个函数。嵌套函数的定义形式与普通函数一样，只是要位于一个函数的函数体内。

### 示例

``` python
def outer():
    def inner():
        print("inner function")
    print("outer function")
```

**被嵌套函数不会随着外部函数的执行而自动执行，例如：**

``` python
outer()
```

输出：

    outer function

**要执行被嵌套函数，需要在外部函数的函数体内显式地调用被嵌套函数：**

``` python
def outer():
    def inner():
        print("inner function")
    inner() #显式地调用被嵌套函数
    print("outer function")
```

调用外部函数：

``` python
outer()
```

输出：

    inner function
    outer function

## Python的装饰器

### 定义

嵌套函数的一个重要应用就是装饰器。装饰器也是一个函数，它接收其它函数作为参数，在此函数的基础上增加一些功能作为装饰，最后返回一个新函数。

### 示例

例如，我们有一个普通函数，它负责完成某项功能：

``` python
def main():
    print("主要功能")
```

直接去调用它：

``` python
main()
```

只会输出：

    主要功能

我们可以写一个装饰器函数，来为main函数添加一些新的功能，比如main函数的调用时间：

``` python
import time # 导入时间模块

def decorator(fun): #装饰器函数接收一个函数作为参数
    def wrapper(*args,**kwargs): # 不定长参数的写法，*表示接收若干个参数作为一个元组传给形参args，**表示接收若干个类似键值对的参数作为一个字典传给形参kwargs
        print("前装饰：",time.ctime()) #作日志记录
        result=fun(*args,**kwargs) #执行被装饰的函数，结果作为result保存
        print("后装饰")
        return result #返回被装饰函数的执行结果
    return wrapper #返回装饰后的新函数
```

用装饰器来装饰函数：

``` python
@decorator
def main():
    print("主要功能")
```

+ 在普通函数前面加上`@装饰器函数名`就可以使用该装饰器了，这样普通函数就变成了装饰后的新函数，函数名不变，调用函数的方法与原来一样。

调用新函数：

``` python
main()
```

结果：

    前装饰：Mon Apr 6 13:14:20 2020
    主要功能
    后装饰

### 应用

装饰器是一种高级编程方法，常见于框架/架构等大型系统内，可用于日志/授权/性能测试等方面。装饰器的作用就是为已经存在的函数对象添加额外的功能。一般出于安全性考虑，不允许修改原来存在的函数代码，就可以用装饰器来实现功能的拓展。

## Python的生成器

### 关键字——yield

生成器也是一种函数，是一种内含yield语句的特殊函数，用于创建生成器对象。

### 生成器应用

例如，如果需要产生一亿个数据，普通函数就需要将这一亿个数据都生成出来并返回，这样内存消耗巨大。而使用生成器对象生成数据，可以每次只生成一个数据，处理完后通过迭代再生成下一个数据，这样就节省内存。

### 生成器对象的特点

1. 惰性机制：生成器对象创建出来后本身并没有数据，需要通过迭代去一个一个地生成数据。
2. 只能向前：生成器对象只能向前迭代，例如已经生成了1，2，3这3个数据，就不能再重新生成1，2，3。
3. 节省内存。

### 示例一

我们写一个生成器函数并迭代输出它：

``` python
def g(): #生成器函数
    a=1
    while True:
        yield a
        a=a+2

c=g() #创建生成器对象
print(next(c))
print(next(c))
print(next(c))
```

+ yield语句的作用就是返回一个值并暂停生成器函数的执行，只有下次通过内置函数next或for循环来迭代生成器对象时再恢复执行，并且是从yield语句的下一条语句继续执行。由于这个示例的生成器函数里的循环条件为True，那么生成器对象可以无限迭代下去，生成无穷多个数据。

输出结果：

    1
    3
    5

### 示例二

我们经常使用的for循环语句`for x in range(n)`中的range函数也可以创建一个迭代对象，但请注意它不是生成器函数。range函数可以接收3个参数range(m,n,q)，作用是生成[m,n)范围内的数据，数据间隔或者说步长为整型q。我们可以使用生成器来重写range函数，使其具有浮点步长：

``` python
def range2(m,n,q): #生成器函数
    while m<n:
        yield m
        m=m+q

for x in range2(1,5,0.5): #通过for循环来迭代生成器对象
    print(x,end=' ')
```

输出结果：

    1 1.5 2.0 2.5 3.0 3.5 4.0 4.5

### 注意

生成器创建的生成器对象，一旦被迭代完毕，即这个对象所能生成的数据都生成完了，那么该对象就被消耗完了，继续迭代该对象不会再生成任何数据，就不会有任何输出了。如果还想继续使用这个生成器，就需要重新创建生成器对象。

## 列表生成式

列表生成式即List Comprehensions，是python内置的非常简单却强大的可以用来创建list的生成式。

例如，要生成list`[1,2,3,4,5,6]`，我们可以使用一个迭代对象来生成list：

``` python
list(range(1,7))
```

结果：

    [1,2,3,4,5,6]

但如果我们要生成list`[1,2,3,4,5,6]^2`，即列表里的每个元素变成平方值，我们该如何做？

我们可以使用强大的列表生成式：

``` python
x=[x**2 for x in range(1,7)]
print(x)
```

+ 写列表生成式时，把要生成的元素的表达式写在前面，后面跟个for循环，就可以把list创建出来了。

结果：

    [1,4,9,16,25,36]

我们还可以继续在列表生成式后面加上if判断：

``` python
x=[x**2 for x in range(1,7) if x%2==0]
print(x)
```

+ 这样就可以仅生成偶数的平方。if仅作为筛选条件，后面不能再跟else。

结果：

    [4,16,36]

还可以通过循环嵌套来进行排列组合，例如：

``` python
x=[m+n for m in 'ABC' for n in 'XYZ']
print(x)
```

结果：

    ['AX', 'AY', 'AZ', 'BX', 'BY', 'BZ', 'CX', 'CY', 'CZ']

for循环中还可以使用两个变量来生成list：

``` python
d={1:'A',2:'B',3:'C'} #先创建一个字典
x=[str(key)+'='+value for key,value in d.items()]
print(x)
```

结果：

    ['1=A', '2=B', '3=C']