# NumPy基础

NumPy(Numerical Python)是Python科学计算的基础库，提供了高性能的多维数组对象和处理这些数组的工具。本文详细介绍NumPy的核心概念和使用方法。

## NumPy简介

### 什么是NumPy

NumPy是Python中用于科学计算的核心库，它提供了：

- 强大的N维数组对象
- 复杂的广播功能
- 用于集成C/C++和Fortran代码的工具
- 实用的线性代数、傅里叶变换和随机数功能
- 高效的数值计算操作

NumPy的核心是`ndarray`(多维数组)对象，它封装了Python原生的同数据类型的多维数组，提供了有效的存储和操作大型数据集所需的接口。

### 为什么使用NumPy

相比于Python原生的列表，NumPy数组具有以下优势：

1. **内存效率更高**：NumPy数组在存储同类型数据时比Python列表更紧凑
2. **计算效率更高**：NumPy使用预编译的C代码执行操作，比纯Python代码更快
3. **便捷的数组操作**：NumPy提供了丰富的函数和操作符，使数组操作更加简单高效
4. **基于数组的数学计算**：可以直接对整个数组进行数学运算，无需显式循环
5. **集成了线性代数功能**：包含丰富的线性代数、傅里叶变换等功能

### 安装NumPy

使用pip安装NumPy：

```bash
pip install numpy
```

使用conda安装NumPy：

```bash
conda install numpy
```

## NumPy数组基础

### 创建数组

NumPy提供了多种创建数组的方法：

```python
import numpy as np

# 从Python列表创建数组
arr1 = np.array([1, 2, 3, 4, 5])
print(arr1)  # [1 2 3 4 5]

# 创建二维数组
arr2d = np.array([[1, 2, 3], [4, 5, 6]])
print(arr2d)
# [[1 2 3]
#  [4 5 6]]

# 创建指定类型的数组
arr_float = np.array([1, 2, 3], dtype=np.float64)
print(arr_float)  # [1. 2. 3.]

# 创建全零数组
zeros = np.zeros((3, 4))  # 3行4列的全零数组
print(zeros)
# [[0. 0. 0. 0.]
#  [0. 0. 0. 0.]
#  [0. 0. 0. 0.]]

# 创建全一数组
ones = np.ones((2, 3, 4))  # 2x3x4的全一数组
print(ones)

# 创建指定值数组
full = np.full((2, 2), 7)  # 2x2数组，所有元素都是7
print(full)
# [[7 7]
#  [7 7]]

# 创建单位矩阵
eye = np.eye(3)  # 3x3单位矩阵
print(eye)
# [[1. 0. 0.]
#  [0. 1. 0.]
#  [0. 0. 1.]]

# 创建等间隔数组
arange = np.arange(0, 10, 2)  # 从0到10，步长为2
print(arange)  # [0 2 4 6 8]

# 创建均匀分布的数组
linspace = np.linspace(0, 1, 5)  # 0到1之间的5个均匀分布的点
print(linspace)  # [0.   0.25 0.5  0.75 1.  ]

# 创建随机数数组
random = np.random.random((2, 2))  # 2x2随机数数组(0到1之间)
print(random)
# [[0.42 0.72]
#  [0.31 0.44]] (随机值，每次运行结果不同)

# 创建正态分布随机数
normal = np.random.normal(0, 1, (2, 2))  # 均值0，标准差1的2x2正态分布随机数
print(normal)

# 创建随机整数
randint = np.random.randint(0, 10, (3, 3))  # 0到10之间的3x3随机整数
print(randint)
```

### 数组属性

NumPy数组具有多种有用的属性：

```python
arr = np.array([[1, 2, 3], [4, 5, 6]])

# 数组形状
print(arr.shape)  # (2, 3)

# 数组维度
print(arr.ndim)  # 2

# 数组元素总数
print(arr.size)  # 6

# 数组元素类型
print(arr.dtype)  # int64

# 每个元素的字节大小
print(arr.itemsize)  # 8 (64位整数=8字节)

# 数组的总字节大小
print(arr.nbytes)  # 48 (6个元素 * 8字节)
```

### 数组索引与切片

NumPy数组支持灵活的索引和切片操作：

```python
# 一维数组索引
arr1d = np.array([1, 2, 3, 4, 5])
print(arr1d[0])    # 1 (第一个元素)
print(arr1d[-1])   # 5 (最后一个元素)

# 一维数组切片
print(arr1d[1:4])  # [2 3 4] (索引1到索引3)
print(arr1d[:3])   # [1 2 3] (开始到索引2)
print(arr1d[2:])   # [3 4 5] (索引2到结束)
print(arr1d[::2])  # [1 3 5] (每隔一个元素)

# 二维数组索引
arr2d = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])
print(arr2d[0, 0])  # 1 (第一行第一列)
print(arr2d[2, 1])  # 8 (第三行第二列)

# 二维数组切片
print(arr2d[0, :])     # [1 2 3] (第一行所有列)
print(arr2d[:, 1])     # [2 5 8] (所有行的第二列)
print(arr2d[1:3, 0:2]) # [[4 5], [7 8]] (第2-3行，第1-2列)

# 布尔索引
mask = arr1d > 3
print(mask)          # [False False False True True]
print(arr1d[mask])   # [4 5] (只选择大于3的元素)
print(arr1d[arr1d > 3])  # [4 5] (简化写法)

# 花式索引
indices = np.array([0, 2, 4])
print(arr1d[indices])  # [1 3 5] (选择特定索引的元素)
```

### 数组变形与转置

NumPy提供了多种数组形状变换的方法：

```python
arr = np.arange(12)
print(arr)  # [ 0  1  2  3  4  5  6  7  8  9 10 11]

# 改变数组形状
arr_reshaped = arr.reshape(3, 4)
print(arr_reshaped)
# [[ 0  1  2  3]
#  [ 4  5  6  7]
#  [ 8  9 10 11]]

# 使用-1自动计算维度
arr_auto = arr.reshape(2, -1)  # 2行，列数自动计算
print(arr_auto)
# [[ 0  1  2  3  4  5]
#  [ 6  7  8  9 10 11]]

# 展平数组
flattened = arr_reshaped.flatten()  # 返回拷贝
print(flattened)  # [ 0  1  2  3  4  5  6  7  8  9 10 11]
raveled = arr_reshaped.ravel()      # 返回视图
print(raveled)    # [ 0  1  2  3  4  5  6  7  8  9 10 11]

# 转置数组
transposed = arr_reshaped.T
print(transposed)
# [[ 0  4  8]
#  [ 1  5  9]
#  [ 2  6 10]
#  [ 3  7 11]]

# 交换轴
arr3d = np.arange(24).reshape(2, 3, 4)
print(arr3d)
# [[[ 0  1  2  3]
#   [ 4  5  6  7]
#   [ 8  9 10 11]]
#  [[12 13 14 15]
#   [16 17 18 19]
#   [20 21 22 23]]]

swapped = np.swapaxes(arr3d, 1, 2)
print(swapped.shape)  # (2, 4, 3)
```

## 数组操作

### 基本运算

NumPy支持数组之间的元素级运算，以及数组与标量之间的运算：

```python
a = np.array([1, 2, 3])
b = np.array([4, 5, 6])

# 数组加法
print(a + b)  # [5 7 9]
print(np.add(a, b))  # [5 7 9]

# 数组减法
print(a - b)  # [-3 -3 -3]
print(np.subtract(a, b))  # [-3 -3 -3]

# 数组乘法(元素级)
print(a * b)  # [4 10 18]
print(np.multiply(a, b))  # [4 10 18]

# 数组除法
print(a / b)  # [0.25 0.4 0.5]
print(np.divide(a, b))  # [0.25 0.4 0.5]

# 数组幂运算
print(a ** 2)  # [1 4 9]
print(np.power(a, 2))  # [1 4 9]

# 数组与标量运算
print(a + 2)  # [3 4 5]
print(a * 2)  # [2 4 6]

# 数组比较
print(a > 2)  # [False False True]
print(a == b)  # [False False False]

# 数学函数
print(np.sqrt(a))  # [1. 1.41421356 1.73205081]
print(np.exp(a))   # [ 2.71828183  7.3890561  20.08553692]
print(np.sin(a))   # [0.84147098 0.90929743 0.14112001]
print(np.log(a))   # [0. 0.69314718 1.09861229]
```

### 聚合函数

NumPy提供了多种聚合和统计函数：

```python
arr = np.array([[1, 2, 3], [4, 5, 6]])

# 求和
print(np.sum(arr))       # 21 (所有元素的和)
print(arr.sum())         # 21 (方法形式)
print(np.sum(arr, axis=0))  # [5 7 9] (沿着行方向求和)
print(np.sum(arr, axis=1))  # [6 15] (沿着列方向求和)

# 求平均值
print(np.mean(arr))      # 3.5
print(arr.mean(axis=0))  # [2.5 3.5 4.5]

# 求标准差
print(np.std(arr))       # 1.707825127659933
print(arr.std(axis=1))   # [0.81649658 0.81649658]

# 最小值和最大值
print(np.min(arr))       # 1
print(arr.max(axis=0))   # [4 5 6]

# 最小值和最大值的索引
print(np.argmin(arr))    # 0
print(np.argmax(arr, axis=1))  # [2 2]

# 求乘积
print(np.prod(arr))      # 720 (所有元素的乘积)
print(arr.prod(axis=0))  # [4 10 18]

# 累积和与累积积
print(np.cumsum(arr))    # [ 1  3  6 10 15 21]
print(np.cumprod(arr))   # [  1   2   6  24 120 720]

# 百分位数
print(np.percentile(arr, 50))  # 3.5 (中位数，即50%百分位数)
print(np.percentile(arr, [25, 50, 75]))  # [2.25 3.5  4.75]

# 中位数
print(np.median(arr))    # 3.5
```

### 广播机制

广播(Broadcasting)是NumPy的一种强大功能，允许在算术运算期间自动处理不同形状的数组：

```python
# 广播标量
arr = np.array([[1, 2, 3], [4, 5, 6]])
print(arr + 10)
# [[11 12 13]
#  [14 15 16]]

# 广播向量
row_vector = np.array([10, 20, 30])
print(arr + row_vector)
# [[11 22 33]
#  [14 25 36]]

col_vector = np.array([[100], [200]])
print(arr + col_vector)
# [[101 102 103]
#  [204 205 206]]

# 广播规则示例
a = np.array([[0, 0, 0], [10, 10, 10], [20, 20, 20]])  # 3x3
b = np.array([1, 2, 3])  # 1D数组长度为3
print(a + b)
# [[ 1  2  3]
#  [11 12 13]
#  [21 22 23]]

# 更复杂的广播
x = np.ones((3, 4))
y = np.arange(4)
print(x + y)
# [[1. 2. 3. 4.]
#  [1. 2. 3. 4.]
#  [1. 2. 3. 4.]]
```

广播机制遵循以下规则：
1. 如果两个数组的维度不同，形状较小的数组会在前面添加维度1
2. 如果两个数组的形状在任何维度上不匹配，但其中一个维度为1，则该维度会被扩展以匹配另一个数组的形状
3. 如果两个数组的形状在任何维度上不匹配，且没有一个维度为1，则会引发错误

### 数组堆叠与拆分

NumPy提供了多种方法来堆叠和拆分数组：

```python
a = np.array([[1, 2], [3, 4]])
b = np.array([[5, 6], [7, 8]])

# 垂直堆叠
vertical = np.vstack((a, b))
print(vertical)
# [[1 2]
#  [3 4]
#  [5 6]
#  [7 8]]

# 水平堆叠
horizontal = np.hstack((a, b))
print(horizontal)
# [[1 2 5 6]
#  [3 4 7 8]]

# 深度堆叠(增加新维度)
depth = np.dstack((a, b))
print(depth)
# [[[1 5]
#   [2 6]]
#  [[3 7]
#   [4 8]]]

# 通用堆叠函数
stacked = np.concatenate((a, b), axis=0)  # 等同于vstack
print(stacked)
# [[1 2]
#  [3 4]
#  [5 6]
#  [7 8]]

# 增加维度再堆叠
newaxis = np.stack((a, b), axis=0)
print(newaxis.shape)  # (2, 2, 2)

# 数组拆分
arr = np.arange(16).reshape(4, 4)
print(arr)
# [[ 0  1  2  3]
#  [ 4  5  6  7]
#  [ 8  9 10 11]
#  [12 13 14 15]]

# 水平拆分
h1, h2 = np.hsplit(arr, 2)
print(h1)
# [[ 0  1]
#  [ 4  5]
#  [ 8  9]
#  [12 13]]

# 垂直拆分
v1, v2, v3, v4 = np.vsplit(arr, 4)
print(v1)
# [[0 1 2 3]]

# 通用拆分函数
splits = np.split(arr, 2, axis=1)
print(splits[0])
# [[ 0  1]
#  [ 4  5]
#  [ 8  9]
#  [12 13]]
```

## 数组函数

### 通用函数(ufuncs)

NumPy的通用函数(Universal Functions，简称ufuncs)是对数组中每个元素执行操作的函数：

```python
arr = np.array([-5, -3, 0, 3, 5])

# 基本数学函数
print(np.abs(arr))    # [5 3 0 3 5] (绝对值)
print(np.sqrt(np.abs(arr)))  # [2.23606798 1.73205081 0.         1.73205081 2.23606798]
print(np.square(arr))  # [25  9  0  9 25] (平方)

# 指数和对数
print(np.exp(arr))    # [6.73794700e-03 4.97870684e-02 1.00000000e+00 2.00855369e+01 1.48413159e+02]
print(np.log(np.abs(arr) + 1))  # [1.79175947 1.38629436 0.         1.38629436 1.79175947]

# 三角函数
angles = np.array([0, np.pi/4, np.pi/2])
print(np.sin(angles))  # [0.         0.70710678 1.        ]
print(np.cos(angles))  # [1.         0.70710678 0.        ]
print(np.tan(angles))  # [0.         1.         1.63312394e+16]

# 取整函数
decimals = np.array([1.2, 3.7, -2.8])
print(np.floor(decimals))  # [ 1.  3. -3.]
print(np.ceil(decimals))   # [ 2.  4. -2.]
print(np.round(decimals))  # [ 1.  4. -3.]
print(np.trunc(decimals))  # [ 1.  3. -2.]

# 复数操作
complex_arr = np.array([1+2j, 3-4j])
print(np.real(complex_arr))  # [1. 3.]
print(np.imag(complex_arr))  # [ 2. -4.]
print(np.conjugate(complex_arr))  # [1.-2.j 3.+4.j]

# 双操作数ufuncs
a = np.array([1, 2, 3])
b = np.array([4, 5, 6])
print(np.maximum(a, b))  # [4 5 6]
print(np.minimum(a, b))  # [1 2 3]
print(np.mod(b, a))      # [0 1 0]
```

### 随机数生成

NumPy提供了强大的随机数生成功能：

```python
# 设置随机数种子
np.random.seed(42)

# 均匀分布
print(np.random.rand(3, 2))  # 0到1之间的均匀分布
print(np.random.uniform(1, 10, (2, 3)))  # 1到10之间的均匀分布

# 正态分布
print(np.random.randn(3, 2))  # 标准正态分布
print(np.random.normal(5, 2, (2, 3)))  # 均值5，标准差2的正态分布

# 整数随机数
print(np.random.randint(1, 11, 5))  # 1到10之间的随机整数
print(np.random.choice([1, 3, 5, 7, 9], 3))  # 从给定数组中随机选择

# 随机排列
arr = np.arange(10)
np.random.shuffle(arr)  # 原地洗牌
print(arr)
print(np.random.permutation(10))  # 返回洗牌后的新数组

# 随机样本
print(np.random.choice(10, 5, replace=False))  # 不放回抽样
print(np.random.choice(10, 5, replace=True, p=[0.1]*9+[0.1]))  # 带权重的抽样
```

### 线性代数

NumPy提供了丰富的线性代数函数：

```python
# 创建矩阵
a = np.array([[1, 2], [3, 4]])
b = np.array([[5, 6], [7, 8]])

# 矩阵乘法
print(np.dot(a, b))  # 点积
# [[19 22]
#  [43 50]]
print(a @ b)  # 使用@运算符(Python 3.5+)
# [[19 22]
#  [43 50]]

# 矩阵特征值和特征向量
eigenvalues, eigenvectors = np.linalg.eig(a)
print(eigenvalues)  # [-0.37228132  5.37228132]
print(eigenvectors)
# [[-0.82456484 -0.41597356]
#  [ 0.56576746 -0.90937671]]

# 矩阵求逆
inv_a = np.linalg.inv(a)
print(inv_a)
# [[-2.   1. ]
#  [ 1.5 -0.5]]
print(np.dot(a, inv_a))  # 近似单位矩阵
# [[1.00000000e+00 0.00000000e+00]
#  [8.88178420e-16 1.00000000e+00]]

# 矩阵行列式
det_a = np.linalg.det(a)
print(det_a)  # -2.0

# 矩阵分解
u, s, vh = np.linalg.svd(a)  # 奇异值分解
print(u)
print(s)
print(vh)

# 解线性方程组 Ax = b
A = np.array([[3, 1], [1, 2]])
b = np.array([9, 8])
x = np.linalg.solve(A, b)
print(x)  # [2. 3.]
print(np.allclose(np.dot(A, x), b))  # True

# 矩阵范数
print(np.linalg.norm(a))  # Frobenius范数
print(np.linalg.norm(a, ord=1))  # 1-范数
print(np.linalg.norm(a, ord=np.inf))  # 无穷范数

# 矩阵的秩
print(np.linalg.matrix_rank(a))  # 2
```

## 高级应用

### 广播和向量化

理解广播并使用向量化操作可以显著提高代码效率：

```python
# 低效的循环方式
def add_slow(a, b):
    result = np.zeros(a.shape)
    for i in range(a.shape[0]):
        for j in range(a.shape[1]):
            result[i, j] = a[i, j] + b[j]
    return result

# 高效的向量化方式
def add_fast(a, b):
    return a + b  # 利用广播

a = np.random.rand(1000, 1000)
b = np.random.rand(1000)

# %time add_slow(a, b)  # 慢
# %time add_fast(a, b)  # 快，可能快100倍以上

# 向量化示例：计算欧氏距离
def dist_slow(x, y):
    result = 0
    for i in range(len(x)):
        result += (x[i] - y[i]) ** 2
    return np.sqrt(result)

def dist_fast(x, y):
    return np.sqrt(np.sum((x - y) ** 2))  # 或使用 np.linalg.norm(x - y)

x = np.random.rand(1000)
y = np.random.rand(1000)

# %time dist_slow(x, y)  # 慢
# %time dist_fast(x, y)  # 快
```

### 数组的视图与拷贝

理解NumPy中视图和拷贝的区别非常重要：

```python
# 创建原始数组
original = np.array([[1, 2, 3], [4, 5, 6]])
print(original)

# 创建视图(浅拷贝)
view = original.view()
print(view)

# 视图与原始数组共享数据
view[0, 0] = 99
print(original)  # 原始数组也被修改
# [[99  2  3]
#  [ 4  5  6]]

# 切片也创建视图
slice_view = original[:, 1:]
slice_view[0, 0] = 88
print(original)
# [[99 88  3]
#  [ 4  5  6]]

# 创建深拷贝
copy = original.copy()
copy[0, 0] = 77
print(original)  # 原始数组不变
# [[99 88  3]
#  [ 4  5  6]]
print(copy)
# [[77 88  3]
#  [ 4  5  6]]

# 注意reshape的行为
reshaped_view = original.reshape(3, 2)  # 返回视图
reshaped_view[0, 0] = 100
print(original)  # 原始数组被修改

# 某些操作返回拷贝
transposed = original.T  # 返回视图
transposed[0, 0] = 200
print(original)  # 原始数组被修改
# [[200  88  3]
#  [  4   5  6]]

# 而其他操作返回拷贝
filtered = original[original > 5]  # 返回拷贝
filtered[0] = 999
print(original)  # 原始数组不变
```

### 结构化数组

NumPy支持结构化数组，类似于C语言中的结构体：

```python
# 创建结构化数据类型
dt = np.dtype([('name', 'S20'), ('age', 'i4'), ('salary', 'f8')])

# 创建结构化数组
employees = np.array([
    ('John', 25, 55000.0),
    ('Jane', 30, 70000.0),
    ('Jim', 35, 85000.0)
], dtype=dt)

print(employees)
# [(b'John', 25, 55000.) (b'Jane', 30, 70000.) (b'Jim', 35, 85000.)]

# 访问字段
print(employees['name'])  # [b'John' b'Jane' b'Jim']
print(employees['age'])   # [25 30 35]
print(employees['salary'])  # [55000. 70000. 85000.]

# 筛选数据
senior_employees = employees[employees['age'] > 27]
print(senior_employees)
# [(b'Jane', 30, 70000.) (b'Jim', 35, 85000.)]

# 排序
sorted_by_age = np.sort(employees, order='age')
print(sorted_by_age)

# 添加新记录
new_employee = np.array([('Sarah', 28, 62000.0)], dtype=dt)
all_employees = np.concatenate((employees, new_employee))
print(all_employees)
```

### 内存优化

对于大型数据，内存优化非常重要：

```python
# 选择合适的数据类型
large_array_float64 = np.ones(1000000, dtype=np.float64)  # 8字节每元素
large_array_float32 = np.ones(1000000, dtype=np.float32)  # 4字节每元素
large_array_int8 = np.ones(1000000, dtype=np.int8)        # 1字节每元素

print(large_array_float64.nbytes)  # 8000000
print(large_array_float32.nbytes)  # 4000000
print(large_array_int8.nbytes)     # 1000000

# 使用内存映射文件
mmap_array = np.memmap('mmap_file.dat', dtype=np.float64, mode='w+', shape=(1000, 1000))
mmap_array[0, 0] = 1.0
mmap_array.flush()  # 写入磁盘

# 重新加载内存映射文件
mmap_array_reload = np.memmap('mmap_file.dat', dtype=np.float64, mode='r+', shape=(1000, 1000))
print(mmap_array_reload[0, 0])  # 1.0

# 使用稀疏矩阵(来自scipy.sparse)
import scipy.sparse as sp
dense_matrix = np.zeros((1000, 1000))
dense_matrix[0, 0] = 1
dense_matrix[500, 500] = 2

# 转换为稀疏矩阵
sparse_matrix = sp.csr_matrix(dense_matrix)
print(dense_matrix.nbytes)  # 8000000
print(sparse_matrix.data.nbytes + sparse_matrix.indptr.nbytes + sparse_matrix.indices.nbytes)  # 远小于8MB
```

### 性能优化技巧

使用NumPy时的一些性能优化技巧：

```python
# 1. 预分配数组
n = 1000000
result = np.zeros(n)  # 预分配
for i in range(n):
    result[i] = i ** 2  # 填充预分配的数组

# 2. 使用内置函数代替循环
result = np.arange(n) ** 2  # 比循环快得多

# 3. 避免复制大型数组
a = np.random.rand(10000, 10000)
b = a  # 不复制数据
c = a.view()  # 不复制数据
d = a.copy()  # 复制数据(占用更多内存)

# 4. 使用numexpr进行复杂表达式计算
import numexpr as ne
x = np.random.rand(1000000)
y = np.random.rand(1000000)
z = np.random.rand(1000000)

# 标准NumPy计算
result1 = 2*x + 3*y**2 - 4*z

# 使用numexpr(通常更快，特别是对于复杂表达式)
result2 = ne.evaluate("2*x + 3*y**2 - 4*z")

# 5. 利用NumPy的缓存友好性
# 按行主序遍历(更快，因为NumPy默认是行主序)
arr = np.random.rand(1000, 1000)
for i in range(1000):
    for j in range(1000):
        arr[i, j] = i + j  # 缓存友好

# 6. 使用where避免条件循环
conditions = np.random.randint(0, 2, 1000000, dtype=bool)
values = np.random.rand(1000000)

# 不好的方式(循环)
result = np.zeros_like(values)
for i in range(len(values)):
    if conditions[i]:
        result[i] = values[i]
    else:
        result[i] = -values[i]

# 好的方式(向量化)
result = np.where(conditions, values, -values)
```

## 与其他库的集成

### 与Pandas的集成

NumPy与Pandas可以无缝集成：

```python
import pandas as pd

# NumPy数组转换为DataFrame
arr = np.random.rand(5, 3)
df = pd.DataFrame(arr, columns=['A', 'B', 'C'])
print(df)

# DataFrame转换为NumPy数组
arr_back = df.to_numpy()
print(arr_back)

# 在Pandas中使用NumPy函数
df['D'] = np.sqrt(df['A']) + np.log(df['B'] + 1)
print(df)

# 对DataFrame应用NumPy通用函数
print(np.sin(df))
```

### 与Matplotlib的集成

NumPy与Matplotlib配合使用可以创建强大的可视化：

```python
import matplotlib.pyplot as plt

# 创建数据
x = np.linspace(0, 2*np.pi, 100)
y_sin = np.sin(x)
y_cos = np.cos(x)

# 绘制图形
plt.figure(figsize=(10, 6))
plt.plot(x, y_sin, 'b-', label='sin(x)')
plt.plot(x, y_cos, 'r--', label='cos(x)')
plt.legend()
plt.grid(True)
plt.title('Sine and Cosine Functions')
plt.xlabel('x')
plt.ylabel('y')
plt.show()

# 创建2D图像
matrix = np.random.rand(10, 10)
plt.figure(figsize=(8, 8))
plt.imshow(matrix, cmap='viridis')
plt.colorbar()
plt.title('Random 2D Matrix')
plt.show()

# 创建3D表面图
from mpl_toolkits.mplot3d import Axes3D
x = np.linspace(-5, 5, 50)
y = np.linspace(-5, 5, 50)
X, Y = np.meshgrid(x, y)
Z = np.sin(np.sqrt(X**2 + Y**2))

fig = plt.figure(figsize=(10, 8))
ax = fig.add_subplot(111, projection='3d')
surface = ax.plot_surface(X, Y, Z, cmap='plasma')
fig.colorbar(surface)
plt.title('3D Surface Plot')
plt.show()
```

### 与SciPy的集成

NumPy是SciPy的基础，两者无缝协作：

```python
import scipy.linalg as la
import scipy.stats as stats
import scipy.optimize as opt

# 线性代数扩展
A = np.array([[1, 2], [3, 4]])
# LU分解
P, L, U = la.lu(A)
print(P)
print(L)
print(U)

# 计算矩阵幂
A_power = la.fractional_matrix_power(A, 0.5)  # 矩阵平方根
print(A_power)

# 统计功能
data = np.random.normal(size=1000)
print(stats.describe(data))  # 统计描述
print(stats.normaltest(data))  # 正态性检验

# 优化功能
def f(x):
    return x**2 + 10*np.sin(x)

x = np.linspace(-10, 10, 1000)
plt.plot(x, f(x))
plt.grid(True)
plt.show()

result = opt.minimize(f, x0=0)
print(result)
```

## 最佳实践

### 代码风格和约定

编写NumPy代码时的最佳实践：

```python
# 导入约定
import numpy as np

# 使用显式的数组创建
arr = np.array([1, 2, 3])  # 好
# arr = np.r_[1, 2, 3]  # 避免使用

# 使用数组属性而非函数
n = len(arr)  # 可以，但不是最NumPy的方式
n = arr.size  # 更好的NumPy风格

# 使用布尔索引而非循环
mask = arr > 1
filtered = arr[mask]  # 好
# for i in range(len(arr)):  # 避免使用循环
#     if arr[i] > 1:
#         ...

# 处理NaN值
data = np.array([1, 2, np.nan, 4])
mean = np.nanmean(data)  # 好，忽略NaN
# mean = np.mean(data[~np.isnan(data)])  # 也可以，但更啰嗦

# 避免隐式类型转换
a = np.array([1, 2, 3])
b = np.array([4, 5, 6], dtype=np.float64)
c = a.astype(np.float64) + b  # 显式转换
# c = a + b  # 可行，但隐式转换

# 使用函数式编程风格
def process_array(arr):
    return np.sqrt(arr) + 2

result = process_array(arr)  # 易于理解和测试
```

### 常见陷阱和如何避免

使用NumPy时要注意的常见问题：

```python
# 1. 数组维度不匹配
a = np.array([1, 2, 3])
b = np.array([[1], [2], [3]])
# c = a + b  # 广播可能导致意外结果
c = a + b.flatten()  # 明确表达意图

# 2. 视图与拷贝的混淆
arr = np.array([[1, 2, 3], [4, 5, 6]])
slice_view = arr[:, :2]
slice_view[0, 0] = 99  # 修改原始数组
# 如果不希望修改原始数组
slice_copy = arr[:, :2].copy()

# 3. 类型溢出
small_int = np.array([1, 2, 3], dtype=np.int8)
big_int = small_int * 100  # 可能溢出
# 避免溢出
safe_int = small_int.astype(np.int32) * 100

# 4. 忽略NaN传播
data = np.array([1, 2, np.nan, 4])
mean = np.mean(data)  # 结果是NaN
# 正确处理NaN
mean = np.nanmean(data)

# 5. 低效的循环
large_arr = np.random.rand(1000000)
# 低效：
# squared = np.zeros_like(large_arr)
# for i in range(large_arr.size):
#     squared[i] = large_arr[i] ** 2
# 高效：
squared = large_arr ** 2

# 6. 忽略维度顺序
matrix = np.random.rand(1000, 10)  # 1000行，10列
# 低效：按列遍历
# for j in range(10):
#     for i in range(1000):
#         process(matrix[i, j])
# 高效：按行遍历
# for i in range(1000):
#     for j in range(10):
#         process(matrix[i, j])
```

### 调试NumPy代码

调试NumPy代码的技巧：

```python
# 检查数组属性
arr = np.random.rand(3, 4, 5)
print(f"Shape: {arr.shape}, Dtype: {arr.dtype}, Size: {arr.size}")

# 检查数值范围
print(f"Min: {np.min(arr)}, Max: {np.max(arr)}, Mean: {np.mean(arr)}")

# 检查NaN和Inf
print(f"NaNs: {np.isnan(arr).sum()}, Infs: {np.isinf(arr).sum()}")

# 显示数组内容摘要
print("Array preview:")
np.set_printoptions(threshold=10, edgeitems=3)  # 限制输出
print(arr)

# 使用断言进行验证
assert arr.shape == (3, 4, 5), f"Unexpected shape: {arr.shape}"
assert not np.isnan(arr).any(), "Array contains NaN values"

# 打印操作结果
result = arr * 2
print(f"Operation result shape: {result.shape}")
print(f"Sample values: {result[0, 0, :3]}")

# 在Jupyter中进行可视化调试
# %matplotlib inline
# plt.figure(figsize=(10, 6))
# plt.imshow(arr[0], cmap='viridis')
# plt.colorbar()
# plt.title('First slice of the array')
# plt.show()
```

## NumPy生态系统

NumPy是科学计算生态系统的核心，与许多其他库协同工作：

- **SciPy**：建立在NumPy之上，提供更专业的科学计算功能
- **Pandas**：提供数据分析的数据结构和工具
- **Matplotlib**：用于数据可视化
- **Scikit-learn**：机器学习库，建立在NumPy和SciPy之上
- **TensorFlow**和**PyTorch**：深度学习框架，使用NumPy风格的数组
- **Numba**：JIT编译器，加速NumPy代码
- **Dask**：并行计算库，扩展NumPy到更大的数据集
- **Xarray**：处理带标签的多维数组
- **CuPy**：在NVIDIA GPU上实现NumPy API

## 结论

NumPy是Python科学计算的基础，它提供了高效的数组操作和数学函数。通过掌握NumPy，你可以更有效地处理数值计算问题，并与Python科学计算生态系统中的其他库无缝集成。

本文介绍了NumPy的核心概念和功能，从基础的数组创建和操作，到高级的线性代数和优化技巧。随着你对NumPy的深入理解，你将能够编写更高效、更简洁的数据处理代码。
