#python

Python 的列表（List）是 Python 中最常用、最灵活的数据结构之一。它是一个**有序**、**可变**、**允许重复元素**的集合。

以下是对 Python 列表知识点的详细讲解，涵盖从基础操作到进阶用法的方方面面。

---

### 1. 列表的定义与创建

列表使用方括号 `[]` 表示，元素之间用逗号 `,` 分隔。

*   **特点：**
    *   **有序性**：元素的位置是固定的（除非你改变它）。
    *   **可变性**：你可以修改、添加或删除列表中的元素。
    *   **异构性**：一个列表中可以包含不同类型的数据（整数、字符串、甚至其他列表）。

*   **创建方式：**

```python
# 1. 创建空列表
empty_list = []
empty_list_2 = list()

# 2. 创建包含元素的列表
num_list = [1, 2, 3, 4, 5]

# 3. 创建包含不同类型的列表
mixed_list = [1, "Hello", 3.14, True]

# 4. 使用 list() 将其他类型（如字符串、元组）转换为列表
str_list = list("Python")  # 结果: ['P', 'y', 't', 'h', 'o', 'n']
```

---

### 2. 访问列表元素 (Indexing & Slicing)

#### 索引 (Indexing)
列表的索引从 `0` 开始。

```python
fruits = ["apple", "banana", "cherry", "date"]

print(fruits[0])   # "apple" (第一个元素)
print(fruits[2])   # "cherry" (第三个元素)

# 负索引：从末尾开始数，-1 表示最后一个
print(fruits[-1])  # "date"
print(fruits[-2])  # "cherry"
```

#### 切片 (Slicing)
获取列表的一部分。语法：`list[start:stop:step]`
*   **注意**：包含 `start`，**不包含** `stop`。

```python
nums = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

print(nums[2:5])   # [2, 3, 4] (从索引2到4)
print(nums[:3])    # [0, 1, 2] (从开头到索引2)
print(nums[5:])    # [5, 6, 7, 8, 9] (从索引5到末尾)
print(nums[::2])   # [0, 2, 4, 6, 8] (步长为2)
print(nums[::-1])  # [9, 8, ..., 0] (列表反转的常用技巧)
```

---

### 3. 修改列表 (增、删、改)

#### 添加元素 (Add)

1.  **`append(item)`**: 在列表**末尾**添加一个元素。
2.  **`extend(iterable)`**: 将另一个序列的所有元素添加到末尾（合并）。
3.  **`insert(index, item)`**: 在**指定位置**插入一个元素。

```python
my_list = [1, 2]

my_list.append(3)          # [1, 2, 3]
my_list.extend([4, 5])     # [1, 2, 3, 4, 5]
my_list.insert(0, "Start") # ["Start", 1, 2, 3, 4, 5]

# 注意 append 和 extend 的区别：
a = [1, 2]
b = [3, 4]
a.append(b)  # 结果: [1, 2, [3, 4]] -> 把列表当做一个整体元素添加
```

#### 删除元素 (Remove)

1.  **`pop(index)`**: 删除并**返回**指定索引的元素（默认删除最后一个）。
2.  **`remove(value)`**: 删除列表中**第一个**出现的指定值（如果没有会报错）。
3.  **`del` 语句**: 删除指定索引或切片，或者删除整个列表变量。
4.  **`clear()`**: 清空列表，变为空列表 `[]`。

```python
letters = ['a', 'b', 'c', 'd', 'b']

last = letters.pop()     # 删除 'b' (最后一个)，last 变量变为 'b'
removed = letters.pop(0) # 删除 'a' (索引0)

letters.remove('c')      # 删除第一次出现的 'c'
# letters.remove('z')    # 报错 ValueError，因为 'z' 不在列表中

del letters[0]           # 删除当前索引 0 的元素
letters.clear()          # 列表变为 []
```

#### 修改元素 (Update)
直接通过索引赋值。

```python
colors = ["red", "blue", "green"]
colors[1] = "yellow"  # ["red", "yellow", "green"]
```

---

### 4. 常用操作符与内置函数

```python
lst = [10, 20, 30]

# 1. 获取长度
print(len(lst))  # 3

# 2. 成员运算 (检查元素是否存在)
print(20 in lst)     # True
print(50 not in lst) # True

# 3. 列表拼接 (+)
new_lst = lst + [40, 50] # [10, 20, 30, 40, 50]

# 4. 列表重复 (*)
zeros = [0] * 5  # [0, 0, 0, 0, 0]

# 5. 最大值、最小值、求和
print(max(lst))  # 30
print(min(lst))  # 10
print(sum(lst))  # 60
```

---

### 5. 排序与反转

这里有两个容易混淆的概念：**原地修改** vs **返回新列表**。

1.  **`sort()` 方法**: 对列表进行**原地**排序（修改了原列表，无返回值）。
2.  **`sorted()` 函数**: 返回一个新的排序后的列表（原列表不变）。
3.  **`reverse()` 方法**: **原地**反转列表。

```python
nums = [3, 1, 4, 1, 5]

# 使用 sorted() - 原列表不变
sorted_nums = sorted(nums) 
print(sorted_nums) # [1, 1, 3, 4, 5]
print(nums)        # [3, 1, 4, 1, 5]

# 使用 sort() - 原列表改变
nums.sort()
print(nums)        # [1, 1, 3, 4, 5]

# 降序排序
nums.sort(reverse=True) # [5, 4, 3, 1, 1]

# 反转
nums.reverse()
```

---

### 6. 列表推导式 (List Comprehensions) —— **Python 高阶技巧**

这是 Python 最优雅的功能之一，用一行代码生成列表。
**语法**：`[expression for item in iterable if condition]`

```python
# 任务：生成 0 到 9 的平方列表
# 普通写法：
squares = []
for x in range(10):
    squares.append(x**2)

# 列表推导式写法：
squares = [x**2 for x in range(10)]

# 任务：筛选出偶数
evens = [x for x in range(10) if x % 2 == 0] 
# 结果: [0, 2, 4, 6, 8]
```

---

### 7. 常见陷阱：列表的复制

因为列表是**可变对象**，直接赋值只是复制了**引用**（内存地址），而不是复制数据。

```python
a = [1, 2, 3]
b = a          # b 只是指向了 a 的同一个内存地址

b[0] = 99
print(a)       # [99, 2, 3] -> a 也变了！因为它们是同一个对象

# 正确的复制方法（浅拷贝）：
c = a.copy()   # 方法1
d = a[:]       # 方法2：切片
e = list(a)    # 方法3：构造函数

c[0] = 100
print(a)       # [99, 2, 3] -> a 不受影响
```

*注意：如果列表中包含子列表（嵌套列表），需要使用 `copy.deepcopy()` 进行深拷贝。*

---

### 8. 嵌套列表 (Nested Lists)

列表中的元素也可以是列表，常用于表示矩阵。

```python
matrix = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
]

print(matrix[1])    # [4, 5, 6] (第二行)
print(matrix[1][2]) # 6 (第二行，第三列)
```

---

### 总结速查表

| 方法/操作 | 描述 |
| :--- | :--- |
| `append(x)` | 末尾添加元素 x |
| `extend(L)` | 末尾添加列表 L 中的所有元素 |
| `insert(i, x)` | 在索引 i 处插入 x |
| `remove(x)` | 删除第一个值为 x 的元素 |
| `pop(i)` | 删除并返回索引 i 处的元素（默认删最后一个） |
| `clear()` | 清空列表 |
| `index(x)` | 返回第一个值为 x 的索引 |
| `count(x)` | 返回 x 出现的次数 |
| `sort()` | 原地排序 |
| `reverse()` | 原地反转 |
| `copy()` | 返回列表的浅拷贝 |
