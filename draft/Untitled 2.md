下面给出 **4 套两量子比特 Grover 型量子算法电路**，分别把 $|00\rangle$、$|01\rangle$、$|10\rangle$、$|11\rangle$ 加上负号相位，然后使用 **同一个 diffusion 扩散算子**，最后测量得到对应目标态。

下面默认量子比特顺序为：

$$
|q_0q_1\rangle
$$

即 $q_0$ 是左边第一位，$q_1$ 是右边第二位。

---

## 一、量子算法总体结构

每一套算法都遵循相同的四步：

1. **制备均匀叠加态**

$$
|00\rangle \xrightarrow{H\otimes H} |s\rangle
$$

其中

$$
|s\rangle=\frac{1}{2}\left(|00\rangle+|01\rangle+|10\rangle+|11\rangle\right)
$$

2. **Oracle 相位标记**

把目标态的概率幅乘以 $-1$。

例如标记 $|11\rangle$：

$$
|11\rangle \rightarrow -|11\rangle
$$

其它态不变。

3. **Diffusion 扩散放大**

使用相同的 diffusion 算子，把被标记的负相位转化为目标态概率幅放大。

4. **测量**

测量后以最大概率得到目标态。对于两比特、一个目标态的情况，一次 Grover 迭代后理论上可以得到目标态概率 $1$。

---

## 二、统一使用的 Diffusion 扩散模块

本题四套算法使用完全相同的 diffusion 模块。

### Diffusion 电路顺序

```text
q0: ──H──Z──●──H──
             │
q1: ──H──Z──●──H──
```

也就是：

```text
H(q0), H(q1)
Z(q0), Z(q1)
CZ(q0, q1)
H(q0), H(q1)
```

数学形式为：

$$
D=(H\otimes H)(Z\otimes Z)CZ(H\otimes H)
$$

它等价于：

$$
D=2|s\rangle\langle s|-I
$$

其中：

$$
|s\rangle=\frac{1}{2}
\begin{bmatrix}
1\\
1\\
1\\
1
\end{bmatrix}
$$

所以 diffusion 矩阵为：

$$
D=
\frac{1}{2}
\begin{bmatrix}
-1 & 1 & 1 & 1\\
1 & -1 & 1 & 1\\
1 & 1 & -1 & 1\\
1 & 1 & 1 & -1
\end{bmatrix}
$$

它的作用是“关于平均值翻转”，即：

$$
a_i' = 2\bar{a}-a_i
$$

其中 $\bar{a}$ 是所有概率幅的平均值。

---

## 三、相位标记的基本思想

CZ 门的作用是：

$$
CZ|00\rangle=|00\rangle
$$

$$
CZ|01\rangle=|01\rangle
$$

$$
CZ|10\rangle=|10\rangle
$$

$$
CZ|11\rangle=-|11\rangle
$$

所以 CZ 可以直接给 $|11\rangle$ 加上负号相位。

如果我们想给其它态加负号，可以先用 $X$ 门把目标态变成 $|11\rangle$，使用 CZ 加负号，再用 $X$ 门变回去。

例如，要标记 $|00\rangle$：

$$
|00\rangle \xrightarrow{X\otimes X} |11\rangle
$$

然后 CZ 给它加负号：

$$
|11\rangle \xrightarrow{CZ} -|11\rangle
$$

最后再变回：

$$
-|11\rangle \xrightarrow{X\otimes X} -|00\rangle
$$

因此：

$$
|00\rangle \rightarrow -|00\rangle
$$

---

# 四、四套量子算法

---

## 算法一：标记 $|00\rangle$

### 目标

让 $|00\rangle$ 获得负号相位：

$$
|00\rangle \rightarrow -|00\rangle
$$

### Oracle 电路

因为 CZ 默认标记 $|11\rangle$，所以先用 $X$ 把 $|00\rangle$ 映射到 $|11\rangle$。

```text
q0: ──X──●──X──
         │
q1: ──X──●──X──
```

### 完整算法电路

```text
q0: ──H──X──●──X──H──Z──●──H──M
             │           │
q1: ──H──X──●──X──H──Z──●──H──M
```

### 门序列

```text
H(q0), H(q1)

X(q0), X(q1)
CZ(q0, q1)
X(q0), X(q1)

H(q0), H(q1)
Z(q0), Z(q1)
CZ(q0, q1)
H(q0), H(q1)

Measure(q0, q1)
```

### 状态变化

初始：

$$
|\psi_0\rangle=|00\rangle
$$

制备均匀叠加态：

$$
|\psi_1\rangle=
\frac{1}{2}
\left(
|00\rangle+|01\rangle+|10\rangle+|11\rangle
\right)
$$

Oracle 标记 $|00\rangle$：

$$
|\psi_2\rangle=
\frac{1}{2}
\left(
-|00\rangle+|01\rangle+|10\rangle+|11\rangle
\right)
$$

经过 diffusion：

$$
|\psi_3\rangle=|00\rangle
$$

测量结果：

$$
P(00)=1
$$

---

## 算法二：标记 $|01\rangle$

### 目标

让 $|01\rangle$ 获得负号相位：

$$
|01\rangle \rightarrow -|01\rangle
$$

### Oracle 电路

$|01\rangle$ 的第一位是 $0$，第二位是 $1$。

只需要对 $q_0$ 加 $X$，把 $|01\rangle$ 映射成 $|11\rangle$。

```text
q0: ──X──●──X──
         │
q1: ─────●─────
```

### 完整算法电路

```text
q0: ──H──X──●──X──H──Z──●──H──M
             │           │
q1: ──H─────●─────H──Z──●──H──M
```

### 门序列

```text
H(q0), H(q1)

X(q0)
CZ(q0, q1)
X(q0)

H(q0), H(q1)
Z(q0), Z(q1)
CZ(q0, q1)
H(q0), H(q1)

Measure(q0, q1)
```

### 状态变化

制备均匀叠加态：

$$
|\psi_1\rangle=
\frac{1}{2}
\left(
|00\rangle+|01\rangle+|10\rangle+|11\rangle
\right)
$$

Oracle 标记 $|01\rangle$：

$$
|\psi_2\rangle=
\frac{1}{2}
\left(
|00\rangle-|01\rangle+|10\rangle+|11\rangle
\right)
$$

经过 diffusion：

$$
|\psi_3\rangle=|01\rangle
$$

测量结果：

$$
P(01)=1
$$

---

## 算法三：标记 $|10\rangle$

### 目标

让 $|10\rangle$ 获得负号相位：

$$
|10\rangle \rightarrow -|10\rangle
$$

### Oracle 电路

$|10\rangle$ 的第一位是 $1$，第二位是 $0$。

只需要对 $q_1$ 加 $X$，把 $|10\rangle$ 映射成 $|11\rangle$。

```text
q0: ─────●─────
         │
q1: ──X──●──X──
```

### 完整算法电路

```text
q0: ──H─────●─────H──Z──●──H──M
             │           │
q1: ──H──X──●──X──H──Z──●──H──M
```

### 门序列

```text
H(q0), H(q1)

X(q1)
CZ(q0, q1)
X(q1)

H(q0), H(q1)
Z(q0), Z(q1)
CZ(q0, q1)
H(q0), H(q1)

Measure(q0, q1)
```

### 状态变化

制备均匀叠加态：

$$
|\psi_1\rangle=
\frac{1}{2}
\left(
|00\rangle+|01\rangle+|10\rangle+|11\rangle
\right)
$$

Oracle 标记 $|10\rangle$：

$$
|\psi_2\rangle=
\frac{1}{2}
\left(
|00\rangle+|01\rangle-|10\rangle+|11\rangle
\right)
$$

经过 diffusion：

$$
|\psi_3\rangle=|10\rangle
$$

测量结果：

$$
P (10)=1
$$

---

## 算法四：标记 $|11\rangle$

### 目标

让 $|11\rangle$ 获得负号相位：

$$
|11\rangle \rightarrow -|11\rangle
$$

### Oracle 电路

CZ 本身就会给 $|11\rangle$ 加负号，因此不需要额外的 $X$ 门。

```text
q 0: ──●──
      │
q 1: ──●──
```

### 完整算法电路

```text
q 0: ──H──●──H──Z──●──H──M
         │        │
q 1: ──H──●──H──Z──●──H──M
```

### 门序列

```text
H (q 0), H (q 1)

CZ (q 0, q 1)

H (q 0), H (q 1)
Z (q 0), Z (q 1)
CZ (q 0, q 1)
H (q 0), H (q 1)

Measure (q 0, q 1)
```

### 状态变化

制备均匀叠加态：

$$
|\psi_1\rangle=
\frac{1}{2}
\left (
|00\rangle+|01\rangle+|10\rangle+|11\rangle
\right)
$$

Oracle 标记 $|11\rangle$：

$$
|\psi_2\rangle=
\frac{1}{2}
\left (
|00\rangle+|01\rangle+|10\rangle-|11\rangle
\right)
$$

经过 diffusion：

$$
|\psi_3\rangle=|11\rangle
$$

测量结果：

$$
P (11)=1
$$

---

# 五、四套算法汇总表

| 目标态 | Oracle 相位标记方法 | Oracle 门序列 | Diffusion 是否相同   | 理论测量结果           |                                         |      |      |
| --- | ------------- | ---------- | ---------------- | ---------------- | --------------------------------------- | ---- | ---- |
| $   | 00\rangle$    | 把 $        | 00\rangle$ 映射到 $ | 11\rangle$ 后用 CZ | $X (q_0), X (q_1), CZ,X (q_0), X (q_1)$ | 相同   | $00$ |
| $   | 01\rangle$    | 把 $        | 01\rangle$ 映射到 $ | 11\rangle$ 后用 CZ | $X (q_0), CZ,X (q_0)$                   | 相同   | $01$ |
| $   | 10\rangle$    | 把 $        | 10\rangle$ 映射到 $ | 11\rangle$ 后用 CZ | $X (q_1), CZ,X (q_1)$                   | 相同   | $10$ |
| $   | 11\rangle$    | CZ 直接标记 $  | 11\rangle$       | $CZ$             | 相同                                      | $11$ |      |

---

# 六、结合 Phase Kickback 的解释

CZ 可以看成受控的 $Z$ 门：

$$
CZ|q_0 q_1\rangle =
\begin{cases}
-|q_0 q_1\rangle, & q_0=1,\ q_1=1\\
|q_0 q_1\rangle, & \text{其它情况}
\end{cases}
$$

也就是说，当两个量子比特都处于 $|1\rangle$ 分支时，会发生相位翻转。

这就是一种相位回移思想：受控门表面上是对目标比特作用，但由于目标比特处在 $Z$ 门的本征态中，相位会回到整个分支的概率幅上。

对于 CNOT，也有类似关系：

如果目标量子比特为：

$$
|-\rangle=\frac{|0\rangle-|1\rangle}{\sqrt{2}}
$$

则：

$$
CNOT (|\psi\rangle|-\rangle)=Z|\psi\rangle|-\rangle
$$

也就是说，CNOT 可以通过 phase kickback 在控制位上产生 $Z$ 相位。

而 CZ 本质上就是直接实现这种受控相位标记：

$$
CZ = |0\rangle\langle 0|\otimes I + |1\rangle\langle 1|\otimes Z
$$

所以在本题中，Oracle 的核心就是：

1. 用 $X$ 门把目标态变成 $|11\rangle$；
2. 用 CZ 给 $|11\rangle$ 加负号；
3. 再用 $X$ 门恢复原来的基态。

---

# 七、如果需要用 CNOT 实现 CZ

CZ 可以由 CNOT 和 H 门分解：

```text
q 0: ─────●─────
         │
q 1: ──H──X──H──
```

即：

$$
CZ=(I\otimes H) CNOT (I\otimes H)
$$

所以如果实验平台没有 CZ 门，可以把所有 CZ 替换成：

```text
H (q 1)
CNOT (q 0, q 1)
H (q 1)
```

这样整套算法仍然只使用题目要求的：

```text
CZ, CNOT, H, X, Z
```

这些量子门。

---

# 八、最终结论

四套算法的共同结构是：

```text
初始化 |00>
↓
H (q 0), H (q 1) 制备均匀叠加态
↓
Oracle 给某一个目标态加负号
↓
统一 diffusion 扩散
↓
测量
```

四个目标态对应结果为：

$$
|00\rangle \rightarrow 测量得到\ 00
$$

$$
|01\rangle \rightarrow 测量得到\ 01
$$

$$
|10\rangle \rightarrow 测量得到\ 10
$$

$$
|11\rangle \rightarrow 测量得到\ 11
$$

理论上每一种情况最终目标态的测量概率都是：

$$
P=1
$$