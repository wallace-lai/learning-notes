# 【机器学习】科学计算库numpy

作者：wallace-lai <br/>
发布：2023-11-28 <br/>
更新：2023-11-29 <br/>

## 一、简介
NumPy是一个用于**大规模矩阵和数组运算**的**高性能**Python计算库，广泛应用于Python矩阵运算和数据处理，在机器学习中亦有大量应用。

NumPy中有两种基本的对象，分别是多维数组ndarray（N-dimensional Array Object）和通用函数对象ufunc（Universal Function Object）。多维数组是相同类型元素的一个多维存储容器，而通用函数则是一种以逐元素方式操作ndarray的函数，它支持数组广播、类型转换和其他一些标准功能。

## 二、多维数组
一个ndarray是具有相同类型和大小（一般是固定大小）元素的多维容器。容器尺寸和数组中的元素数量是由它的`shape`决定的，数组中的元素类型则由`dtype`指定。

```python
>>> x = np.array([[1, 2, 3], [4, 5, 6]])
>>> type(x)
<class 'numpy.ndarray'>
>>> x.shape
(2, 3)
>>> x.dtype
dtype('int32')
```

### 2.1 创建数组

#### array方法

- 将原始的list或者tuple转换成数组

```python
# 将列表转换成numpy数组
>>> a = np.array([1, 2, 3])
>>> print(a)
[1 2 3]
# 将元祖转换成numpy数组
>>> a = np.array((1.2, 2.3, 3.4))
>>> print(a)
[1.2 2.3 3.4]
```

- 将列表的列表转换成多维数组

```python
>>> a = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])
>>> print(a)
[[1 2 3]
 [4 5 6]
 [7 8 9]]
```

#### 生成数组的特殊方法

- zeros生成全0数组
```python
# 元素默认是float64类型
>>> a = np.zeros((2, 3))
>>> print(a)
[[0. 0. 0.]
 [0. 0. 0.]]
>>> a.dtype
dtype('float64')

# 可以指定元素类型
>>> a = np.zeros((2, 3), np.int32)
>>> print(a)
[[0 0 0]
 [0 0 0]]
>>> a.dtype
dtype('int32')
```

- ones生成全1数组
```python
>>> a = np.ones((2, 3), np.int16)
>>> print(a)
[[1 1 1]
 [1 1 1]]
>>> a.dtype
dtype('int16')
```

- empty创建未初始化的随机数组
```python
>>> a = np.empty([3, 3])
>>> print(a)
[[0.00000000e+000 0.00000000e+000 0.00000000e+000]
 [0.00000000e+000 0.00000000e+000 3.16202013e-321]
 [8.90104238e-307 1.24610383e-306 3.17320232e-317]]
```

- arange生成给定范围内的数组

```python
>>> a = np.arange(10, 30, 5)
>>> print(a)
[10 15 20 25]
>>> a.dtype
dtype('int32')
```

注意arange只生成一维数组，若想生成多维数组需要多次调用arange

#### random模块生成随机数组

### 2.2 数组的索引与切片

### 2.3 数组基本运算

### 2.4 数组维度变换

### 2.5 数组合并与切分


## 三、通用函数

## 四、常用操作


