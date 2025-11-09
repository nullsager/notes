创建一个行向量：

```python
x = torch.arange(12)
x
```

```
tensor([ 0,  1,  2,  3,  4,  5,  6,  7,  8,  9, 10, 11])
```

访问张量的形状：

```python
x.shape
```

```
torch.Size([12])
```

获取张量元素总数：

```python
x.numel()
```

```bash
12
```

改变张量的形状：

```python
X = x.reshape(3, 4)
# 也可以通过-1来调用此自动计算出维度的功能
# X = x.reshape(-1, 4)
# 或 X = x.reshape(3, -1)
X
```

```
tensor([[ 0,  1,  2,  3],
        [ 4,  5,  6,  7],
        [ 8,  9, 10, 11]])
```

创建一个全0的张量：

```python
x = torch.zeros((2, 3, 4))
x
```

```
tensor([[[0., 0., 0., 0.],
         [0., 0., 0., 0.],
         [0., 0., 0., 0.]],

        [[0., 0., 0., 0.],
         [0., 0., 0., 0.],
         [0., 0., 0., 0.]]])
```

创建一个全1的张量：

```python
x = torch.ones((2, 3, 4))
```

创建一个形状为（3,4）的张量。其中的每个元素都从均值为0、标准差为1的标准高斯分布（正态分布）中随机采样：

```python
torch.randn(3, 4)
```

可以通过提供包含数值的Python列表（或嵌套列表），来为所需张量中的每个元素赋予确定值。在这里，最外层的列表对应于轴0，内层的列表对应于轴1：

```python
x = torch.tensor([[2, 1, 4, 3], [1, 2, 3, 4], [4, 3, 2, 1]])
x
```

```
tensor([[2, 1, 4, 3],
        [1, 2, 3, 4],
        [4, 3, 2, 1]])
```

tensor运算符操作：

```python
x = torch.tensor([1.0, 2, 4, 8])
y = torch.tensor([2, 2, 2, 2])
x + y, x - y, x * y, x / y, x ** y
```

```
tensor([ 3.,  4.,  6., 10.])
tensor([-1.,  0.,  2.,  6.])
tensor([ 2.,  4.,  8., 16.])
tensor([0.5000, 1.0000, 2.0000, 4.0000])
tensor([ 1.,  4., 16., 64.])
```

也可以按元素方式计算：

```python
torch.exp(x)
```

```
tensor([2.7183e+00, 7.3891e+00, 5.4598e+01, 2.9810e+03])
```

可以把多个张量连结（concatenate）在一起，把它们端对端地叠起来形成一个更大的张量。只需要提供张量列表，并给出沿哪个轴连结：

```python
x = torch.arange(12, dtype=torch.float).reshape((3, 4))
y = torch.tensor([[2.0, 1, 4, 3], [1, 2, 3, 4], [4, 3, 2, 1]])
z = torch.cat((x, y), dim=0)
print(z)
```

```
tensor([[ 0.,  1.,  2.,  3.],
        [ 4.,  5.,  6.,  7.],
        [ 8.,  9., 10., 11.],
        [ 2.,  1.,  4.,  3.],
        [ 1.,  2.,  3.,  4.],
        [ 4.,  3.,  2.,  1.]])
```

```python
z = torch.cat((x, y), dim=1)
print(z)
```

```
tensor([[ 0.,  1.,  2.,  3.,  2.,  1.,  4.,  3.],
        [ 4.,  5.,  6.,  7.,  1.,  2.,  3.,  4.],
        [ 8.,  9., 10., 11.,  4.,  3.,  2.,  1.]])
```

也可以使用逻辑描述符构建二维张量：

```python
x == y
```

```
tensor([[False,  True, False,  True],
        [False, False, False, False],
        [False, False, False, False]])
```

对张量元素求和：

```python
x.sum()
```

```
tensor(66.)
```

广播机制：

```python
x = torch.arange(3).reshape((3, 1))
y = torch.arange(2).reshape((1, 2))
print(x)
print(y)
print(x + y)
```

```
tensor([[0],
        [1],
        [2]])
tensor([[0, 1]])
tensor([[0, 1],
        [1, 2],
        [2, 3]])
```

索引和切片：

```python
x = torch.arange(12).reshape(3, 4)

print(x)
print(x[-1])
print(x[1:3])  # 左闭右开，实际范围为[1, 2]
```

```
tensor([[ 0,  1,  2,  3],
        [ 4,  5,  6,  7],
        [ 8,  9, 10, 11]])
tensor([ 8,  9, 10, 11])
tensor([[ 4,  5,  6,  7],
        [ 8,  9, 10, 11]])
```

还可以通过指定索引来将元素写入矩阵。

```python
x[1, 2] = 9  # 将第一行第二列的数据设置为9
```

同时对多个元素赋值相同的值：

```python
x[0:2, :] = 12
```

```
tensor([[12, 12, 12, 12],
        [12, 12, 12, 12],
        [ 8,  9, 10, 11]])
```

运行一些操作可能会导致为新结果分配内存。例如，如果我们用`Y = X + Y`，我们将取消引用Y指向的张量，而是指向新分配的内存处的张量：

```python
x = torch.arange(12).reshape(3, 4)
y = torch.ones((3, 4))
a = id(y)
y = x + y
b = id(y)
print(a == b)
```

```
False
```

如果修改为如下代码，则可进行原地更新：

```python
y[:] = x + y
# 或者是y += x
```

将张量转换为其他python对象：

```python
y = x.numpy()
```

要将大小为1的张量转换为Python标量，我们可以调用item函数或Python的内置函数：

```python
a = torch.tensor([3.5])
a, a.item(), float(a), int(a)
```

```
(tensor([3.5000]), 3.5, 3.5, 3)
```


