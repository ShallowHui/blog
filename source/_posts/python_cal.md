---
title: Python第三方库
date: 2023-02-24T12:06:36.000Z
tags:
  - NumPy
  - Matplotlib
  - Python高级
categories: Python
mathjax: true
cover: https://cdn.jsdelivr.net/gh/shallowhui/cdn/top_img/python.jpg
top_img: https://cdn.jsdelivr.net/gh/shallowhui/cdn/top_img/python.jpg
description: 这篇文章主要用于简单介绍一下，一些功能强大的Python第三方库。
---
## NumPy

NumPy是Python中进行科学计算的基础包，它提供了一个多维的数组对象，这比Python基本的列表数据结构强大的多，以及可以对其执行各种科学计算的方法，包括矩阵、逻辑、排序、线性代数、基本统计等等运算。

NumPy已经成为Python中处理数值数据的通用标准，几乎用于科学和工程的每个领域。NumPy的数据结构和API广泛应用于科学Python包。

### 安装NumPy并导入

安装：

```bash
$ pip install numpy
```

导入：

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

>>> a = np.array([[1,2], [3,4]])
>>> a.dtype
dtype('int64')
>>> a.shape
(2, 2)
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
>>> b
array([[1, 2, 3],
       [4, 5, 6],
       [7, 8, 9]])
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

这是因为通过比较运算符，数组对象的逻辑运算会返回一个相同形状的布尔数组：

```python
>>> x>6
array([ True,  True,  True,  True, False, False, False, False, False, False])
```

然后，根据布尔数组中为`True`的元素，对应位置上的数组`x`的元素就会被索引到。

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

数组还可以与单个数字之间进行运算，这在NumPy中被称为向量与标量的运算：

```python
>>> a + 1
array([2, 3])

>>> a * 3
array([3, 6])
```

### 广播

上面数组对象之间运算，是发生在两个形状相同的数组之间的。如果想要两个不同形状的数组之间进行运算，需要用到NumPy的`广播`概念，实际上，上面的数组与一个数字进行运算，正是用到了广播。

**NumPy认为对数组的运算操作应该发生在数组中的每一个元素上，即每一个元素都需要有另一个数组上对应位置上的元素与它进行运算，这个概念叫做`广播`。广播是一种允许NumPy对不同形状的数组进行操作的机制，但是，数组的维度必须兼容，即两个数组从最小(最右)的维度开始比较大小，要么相等要么其中一个为1，否则报错。**

例如，下面这个形状为(2, 2)的数组a：

```python
>>> a = np.array([[1,2], [3,4]])
>>> a * np.array([[3], [2]])
array([[3, 6],
       [6, 8]])

>>> a * np.array([3,6])
array([[ 3, 12],
       [ 9, 24]])
```

(2, 2)和(2, 1)形状的数组相乘，通过广播，相当于进行了下面这样的一个运算：

```
[1, 2] * [3] = [1, 2] * [3, 3] = [3, 6]
[3, 4]   [2]   [3, 4]   [2, 2]   [6, 8]
```

可以理解为(2, 1)的数组被横向拉伸为了(2, 2)的数组。另外，可以自己想象一下下面那个形状为(2,)的一维数组是如何广播到与(2, 2)形状的二维数组相乘的。

而这个例子就会报错：

```python
>>> a * np.array([[3], [2], [1]])
ValueError: operands could not be broadcast together with shapes (2,2) (3,1)
```

这是广播的一般规则，广播还有一个简单的规则，也就是上面提到的一个数组与一个数字相乘，这是最简单的一个广播。**一个数字标量可以看成一个形状为(1，)的数组，可以被拉伸到任何形状与其他数组进行运算。**

### 数组方法

NumPy还提供了很多便利的函数对数组进行操作，下面进行一些简单的介绍：

#### 求和

```python
>>> a = np.array([1,2,3,4])
>>> a.sum()
10
```

不仅可以求全部元素的和，还可以按指定维度进行求和：

```python
>>> a = np.array([[1,2,3], [4,5,6]])
>>> a.sum(axis=0)
array([5, 7, 9])

>>> a.sum(axis=1)
array([6, 15])

>>> a.sum()
21
```

#### 排序

```python
>>> a = np.array([9,8,7,6,5,4,3,2,1])
>>> np.sort(a)
array([1, 2, 3, 4, 5, 6, 7, 8, 9])
```

#### 求平均值和最值

```python
>>> a = np.array([[1,2,3], [4,5,6]])
>>> a.max()
6

>>> a.min()
1

>>> a.mean()
3.5
```

同求和一样，也可以指定维度进行运算：

```python
>>> a.max(axis=1)
array([3, 6])

>>> a.min(axis=1)
array([1, 4])

>>> a.mean(axis=1)
array([2., 5.])
```

### NumPy的数学

前面提到过，NumPy是Python中科学计算的基础，使用非常广泛。这是因为NumPy提供了很多适用于数组的简便方法，使得其实现数学公式的计算非常容易。比如下面这些：

#### 均方误差公式

$$ MSE(y,y^{’}) = {\sum_{i=1}^{n} (y_i-y’_i)^2 \over n} $$

求方差在数学计算中非常普遍，在NumPy中，实现这个公式简单明了：

```python
var = (1/n) * np.sum(np.square(y2 - y1))
```

简单演示一下：

```python
>>> y1 = np.arange(6)
>>> y1
array([0, 1, 2, 3, 4, 5])

>>> y2 = y1 + np.array([-0.1, 0.2, -0.8, 1, -0.1, 0])
>>> y2
array([-0.1,  1.2,  1.2,  4. ,  3.9,  5. ])

>>> (1/6) * np.sum(np.square(y2 - y1))
0.2833333333333333
```

其中，`y1`相当于函数`y=x`，`y2`相当于一些有误差的样本数据，最后的值就是两者之间的方差。

## Pandas

Pandas是Python的核心数据分析支持库，提供了快速、灵活、明确的数据结构，旨在简单、直观地处理**关系型、标记型**数据。

Pandas是基于NumPy开发的，可以与其它第三方科学计算支持库完美集成。Pandas适合处理以下类型的数据：

+ 与SQL或者Excel表类似的，含异构（不同数据类型）列的表格数据。
+ 有序或无序（非固定频率）的时间序列数据。
+ 带行列标签的矩阵数据，包括同构或异构型数据。
+ 任意其它形式的观测、统计数据集。

### 安装和导入Pandas

安装：

```bash
$ pip install pandas
```

导入：

```python
import numpy as np # 一般连同numpy一起导入
import pandas as pd
```

### 数据结构

与NumPy的n维数组结构不同，Pandas的主要数据结构就是`Series`（一维数据）和`DataFrame`（二维数据），因为这两种数据结构已经足以处理金融、统计、社会科学、工程等领域内的大部分典型数据了。

#### Series

Series是带标签的一维数组，可以存储整数、浮点数、字符串、Python对象等类型的数据。Series的标签被称为索引（`index`），一个标签对应一个数据。

调用`pd.Series`函数即可创建Series：

```python
>>> s = pd.Series(data, index=index)
```

`data`参数支持以下几种方式传入数据：

+ Python字典：

```python
>>> d = {'b': 1, 'a': 0, 'c': 2}
>>> s = pd.Series(d)
>>> s
b    1
a    0
c    2
dtype: int64
```

如果以字典的方式传入数据，并且`index`参数缺省，则默认会以字典中相应的key作为数据的标签，可以通过`s.index`查看Series对象的标签：

```python
>>> s.index
Index(['b', 'a', 'c'], dtype='object')
```

如果设置了`index`参数，则会把`index`参数当作key，去提取字典中的值：

```python
>>> d = {'b': 1, 'a': 0, 'c': 2}
>>> s = pd.Series(d, index=['b', 'c', 'd', 'a'])
>>> s
b    1.0
c    2.0
d    NaN
a    0.0
dtype: float64
```

由于字典中没有`d`这个key，所以Series中`d`标签对应的数据为`NaN`。**注意，Pandas中用NaN（Not a Number）表示数据缺失。**

+ numpy对象：

当`data`参数是一个numpy对象时，标签长度必须与numpy对象的长度一致：

```python
>>> s = pd.Series(np.random.randn(5), index=['a', 'b', 'c', 'd', 'e'])
>>> s
a    0.085665
b    0.339841
c   -1.257418
d    0.706560
e    1.096948
dtype: float64
```

如果没有指定`index`参数，则默认会创建一个整数索引：

```python
>>> s = pd.Series(np.random.randn(5))
>>> s
0   -0.094684
1    0.528470
2    0.202291
3    0.919904
4   -0.309003
dtype: float64
>>> s.index
RangeIndex(start=0, stop=5, step=1)
```

+ 列表：

```python
>>> s = pd.Series([1, 3, 5, np.nan, 6, 8])
>>> s
0    1.0
1    3.0
2    5.0
3    NaN
4    6.0
5    8.0
dtype: float64
```

+ 标量值：

如果是标量值的话，必须提供索引，Pandas会自动根据索引长度重复改标量值：

```python
>>> s = pd.Series(5, index=['a', 'b', 'c', 'd', 'e'])
>>> s
a    5
b    5
c    5
d    5
e    5
dtype: int64
```

Series的操作与NumPy的ndarrary对象类似，支持大多数NumPy函数，还可以进行各种花式索引和切片：

```python
>>> s = pd.Series(np.random.randn(5))
>>> s
0   -0.713951
1    1.830373
2   -0.481400
3   -2.423198
4    0.556725
dtype: float64
# 索引
>>> s[0]
-0.7139514490641597
# 切片
>>> s[:3]
0   -0.713951
1    1.830373
2   -0.481400
dtype: float64
# 布尔索引，返回大于中位数的数据
>>> s[s > s.median()]
1    1.830373
4    0.556725
dtype: float64
# 列表索引
>>> s[[4, 3, 1]]
4    0.556725
3   -2.423198
1    1.830373
dtype: float64
```