---
title: Lambda表达式
date: 2020-04-05 20:32:52
tags: Lambda表达式
categories: Python
cover: https://cdn.jsdelivr.net/gh/shallowhui/cdn/top_img/lambda.jpeg
description: 这篇文章简单地介绍了Python中的匿名函数表达式及其用法。
---
## lambda表达式

通常我们在写程序的时候都要写很多函数，这些函数都有自己的名字，我们称为普通函数。但有时候我们可能只需要使用一个函数一次，在这种情况下，就无需正式定义一个普通函数，可以用lambda表达式定义一个匿名函数。lambda表达式不仅仅在Python中有，在其它很多编程语言，如Java，C#中都是存在的。

## lambda表达式的格式

``` python
lambda 形参 : 返回值表达式
```
**lambda表达式就是将接收进来的参数经过一个表达式的运算后，运算结果作为返回值返回。**

特别地，为了函数定义的方便，也可以将lambda表达式变成命名函数，格式如下：

``` python
函数名 = lambda 形参 : 返回值表达式
```

这样就可以像普通函数一样通过函数名去多次调用函数。

**注意，lambda表达式形式简单，只支持单条语句，不支持选择和循环结构。**

## 示例

我们写一个简单的排序程序，希望通过列表里的元素的平方值进行排序。

通常我们先定义一个普通函数：

``` python
from random import randint,seed #引入随机数模块

def square(x):
    return x**2

seed(1) #将随机数种子设为1，这样每次随机出来的数相同
a=[randint(-10,10) for _ in range(10)] #列表生成式生成随机数列表
print('排序前:',a)
a.sort(key=square) #将排序的依据设为数据的平方值
print('排序后:',a)
```
结果为：

    排序前:[-6,8,-8,-2,-7,5,4,5,10,2]
    排序后:[-2,2,4,5,5,-6,-7,8,-8,10]

square这个函数只使用了一次，我们可以通过lambda表达式改写程序：

``` python
a.sort(key=lambda x:x**2) #lambda表达式让程序更加简洁
```

结果一样：

    排序前:[-6,8,-8,-2,-7,5,4,5,10,2]
    排序后:[-2,2,4,5,5,-6,-7,8,-8,10]