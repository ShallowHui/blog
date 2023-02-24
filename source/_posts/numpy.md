---
title: Python第三方库一NumPy
date: 2023-02-24T12:06:36.000Z
tags:
  - NumPy
  - Python高级
categories: Python
cover: https://cdn.jsdelivr.net/gh/shallowhui/cdn/top_img/python.jpg
top_img: https://cdn.jsdelivr.net/gh/shallowhui/cdn/top_img/python.jpg
description: 这个系列的文章简单介绍一下像NumPy、Matplotlib这样功能强大的一些Python第三方库。
---
## NumPy

NumPy是Python中进行科学计算的基础包，它提供了一个多维的数组对象，这比Python基本的列表数据结构强大的多，以及可以对其执行各种科学计算的方法，包括矩阵、逻辑、排序、线性代数、基本统计等等运算。

NumPy已经成为Python中处理数值数据的通用标准，几乎用于科学和工程的每个领域。NumPy的数据结构和API广泛应用于科学Python包。

### 安装NumPy并导入

安装：

```bash
$ pip install numpy
```

导入

```python
import numpy as np # 简写为np比较方便
```

### n维数组

NumPy中多维数组对象由`ndarray`类提供，它描述了`相同类型`元素的集合，即数组中每个元素占用的内存大小是相同的，数据类型由`dtype`属性定义。数组通常是固定大小的容器，数组中的维数和项数由其形状决定，数组的形状是一个指定了`每个维度的大小`的`非负整数元组`，由`shape`属性定义。

在NumPy中，维度称为`轴`，例如下面这个二维数组：

    [[0, 0, 0], 
     [1, 1, 1]]
    
这个数组具有2个轴，第一个轴的长度大小为2，第二个轴的长度大小为3，形状可以用一个元组表示为：`(2, 3)`。

### 创建数组

要创建`ndarray`数组对象，可以使用`np.array()`函数。

可以将一个列表或元组传递给这个函数，返回一个转换后的数组对象：

```python
>>> np.array((1,2,3))
array([1, 2, 3])

>>> np.array([[0,0,0],[1,1,1]])
array([[0, 0, 0],
       [1, 1, 1]])
```

除此之外，还有很多高级函数可以用来创建数组对象，下面简单介绍两个。

指定创建一个内容是给定区间`[start, stop)`内均匀分布的值的一维数组：

```python
>>> np.arange(10)
array([0, 1, 2, 3, 4, 5, 6, 7, 8, 9])
```

`np.arange()`函数的区间start默认为0，步长默认为1，可以自定义：

```python
>>> np.arange(1, 5, step=0.5)
array([1. , 1.5, 2. , 2.5, 3. , 3.5, 4. , 4.5])
```

`np.linspace`函数可以生成在给定区间`[start, stop]`内，均匀分布的若干个样本值，`num`默认为50：

```python
>>> np.linspace(0, 10, num=3)
array([ 0.,  5., 10.])
```

NumPy的数组对象中默认数据类型是浮点型（`np.float64`），但创建数组时可以通过`dtype`指定数据类型：

```python
>>> np.linspace(0, 10, num=3, dtype=np.int64)
array([ 0,  5, 10], dtype=int64)
```

NumPy数据类型有一套自己的定义，与C语言的类型紧密联系，可以参考：

[basics.types](https://numpy.org/doc/stable/user/basics.types.html)

### 索引和切片

数组对象中的元素可以通过索引和切片来访问。

正如Python列表可以通过形如`list[index]、list[m:n]`的方式进行索引和切片一样，`ndarray`也可以通过这样的方式进行索引和切片。

比如，有这样一个一维数组：

```python
>>> a = np.array([1, 2, 3, 4, 5, 6])
```

访问数字6：

```python
>>> a[5]
6
```

访问数字3及后面的数字：

```python
>>> a[2:]
array([3, 4, 5, 6])
```

**NumPy扩展了`[]`语法的能力，使得支持多维数组的索引和切片**。比如有这样一个二维数组：

```python
>>> b = np.array([[1,2,3], [4,5,6], [7,8,9]])
```

把它当成一个矩阵，如果想访问第二行第三列的元素：

```python
>>> b[1,2]
6
```

访问第二行所有元素：

```python
>>> b[1,:]
array([4, 5, 6])
```

不止如此，NmuPy还扩展出了高级索引的能力，功能非常强大。

比如有这样一个数组：

```python
>>> x = np.arange(10, 0, -1)
>>> x
array([10,  9,  8,  7,  6,  5,  4,  3,  2,  1])
```

可以通过一个整数数组去索引访问x，这样可以一次性返回多个元素：

```python
>>> x[np.array([3,4,9])]
array([7, 6, 1])
```

还可以通过布尔数组去索引访问x：

```python
>>> x[x>6]
array([10,  9,  8,  7])
```

通过比较运算符，数组对象的逻辑运算会返回一个相同形状的布尔数组：

```python
>>> x>6
array([ True,  True,  True,  True, False, False, False, False, False, False])
```

所以，根据布尔数组中为`True`的元素，对应位置上的数组`x`的元素就会被索引到。

### 基本数组操作

数组对象之间可以进行加、减、乘、除等运算。

例如：

```python
>>> a = np.array([1,2])
>>> b = np.array([3,4])

>>> a + b
array([4, 6])

>>> a - b
array([-2, -2])

>>> a * b
array([3, 8])

>>> a / b
array([0.33333333, 0.5       ])
```

数组还可以与单个数字之间进行运算：

```python
>>> a + 1
array([2, 3])

>>> a * 3
array([3, 6])
```