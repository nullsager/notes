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













下面我从**零基础前置知识**开始，把你这个作业背后的概念系统讲一遍。你可以把它理解成：我们要学习一个最小版本的 **Grover 搜索算法**。

你的作业本质上是：

> 有 $2$ 个量子比特，初始态都是 $|0\rangle$，也就是 $|00\rangle$。  
> 先用 $H$ 门制备均匀叠加态。  
> 然后分别把 $|00\rangle$、$|01\rangle$、$|10\rangle$、$|11\rangle$ 中的一个状态加上负号。  
> 再用同一个 diffusion 扩散模块，把被加负号的目标态概率放大。  
> 最后测量，得到目标态。

---

# 1. 量子比特是什么？

经典计算机里的比特只能是：

$$
0
$$

或者

$$
1
$$

但量子比特可以是 $|0\rangle$ 和 $|1\rangle$ 的叠加：

$$
|\psi\rangle = a|0\rangle + b|1\rangle
$$

其中：

- $a$ 是 $|0\rangle$ 的概率幅；
- $b$ 是 $|1\rangle$ 的概率幅；
- $a$ 和 $b$ 可以是正数、负数，甚至复数；
- 测量到 $|0\rangle$ 的概率是 $|a|^2$；
- 测量到 $|1\rangle$ 的概率是 $|b|^2$。

必须满足归一化条件：

$$
|a|^2 + |b|^2 = 1
$$

例如：

$$
|\psi\rangle = \frac{1}{\sqrt{2}}|0\rangle + \frac{1}{\sqrt{2}}|1\rangle
$$

测量结果：

$$
P(0)=\left|\frac{1}{\sqrt{2}}\right|^2=\frac{1}{2}
$$

$$
P(1)=\left|\frac{1}{\sqrt{2}}\right|^2=\frac{1}{2}
$$

所以它有一半概率测到 $0$，一半概率测到 $1$。

---

# 2. 什么是 $|0\rangle$、$|1\rangle$？

$|0\rangle$ 和 $|1\rangle$ 是量子比特的两个基本状态，也叫计算基态。

可以用向量表示：

$$
|0\rangle =
\begin{bmatrix}
1\\
0
\end{bmatrix}
$$

$$
|1\rangle =
\begin{bmatrix}
0\\
1
\end{bmatrix}
$$

一个一般的一比特状态：

$$
|\psi\rangle = a|0\rangle + b|1\rangle
$$

写成向量就是：

$$
|\psi\rangle =
\begin{bmatrix}
a\\
b
\end{bmatrix}
$$

其中第一项是 $|0\rangle$ 的概率幅，第二项是 $|1\rangle$ 的概率幅。

---

# 3. 什么是概率幅？

量子态里面的系数叫做**概率幅**。

例如：

$$
|\psi\rangle = \frac{1}{2}|00\rangle + \frac{1}{2}|01\rangle - \frac{1}{2}|10\rangle + \frac{1}{2}|11\rangle
$$

这里：

| 状态 | 概率幅 |
|---|---|
| $ |00\rangle$ | $\frac{1}{2}$ |
| $ |01\rangle$ | $\frac{1}{2}$ |
| $ |10\rangle$ | $-\frac{1}{2}$ |
| $ |11\rangle$ | $\frac{1}{2}$ |

测量概率是概率幅的平方：

$$
P(00)=\left|\frac{1}{2}\right|^2=\frac{1}{4}
$$

$$
P(10)=\left|-\frac{1}{2}\right|^2=\frac{1}{4}
$$

注意，正的 $\frac{1}{2}$ 和负的 $-\frac{1}{2}$ 测量概率一样，都是 $\frac{1}{4}$。

那么负号有什么用？

负号本身不会直接改变测量概率，但它会影响后面的**干涉**。

---

# 4. 什么是相位？

在量子计算中，概率幅可以有正负号，也可以有复数相位。

比如：

$$
\frac{1}{2}
$$

和

$$
-\frac{1}{2}
$$

概率一样，但相位不同。

我们可以说：

$$
-\frac{1}{2}
$$

相比

$$
\frac{1}{2}
$$

多了一个 $\pi$ 相位。

因为：

$$
-1 = e^{i\pi}
$$

所以把某个状态的概率幅乘以 $-1$，也可以说是给它加了一个 $\pi$ 相位。

你的作业中说的“给 $|00\rangle$、$|01\rangle$、$|10\rangle$、$|11\rangle$ 分别增加 $-$ 相位”，意思就是：

$$
|目标态\rangle \rightarrow -|目标态\rangle
$$

例如标记 $|10\rangle$：

$$
|10\rangle \rightarrow -|10\rangle
$$

其它状态不变。

---

# 5. 什么是量子干涉？

量子算法最重要的思想之一就是**干涉**。

概率幅可以相加，也可以相消。

例如：

$$
\frac{1}{2} + \frac{1}{2} = 1
$$

这叫相长干涉，概率幅变大。

而：

$$
\frac{1}{2} - \frac{1}{2} = 0
$$

这叫相消干涉，概率幅消失。

量子算法经常做的事情就是：

1. 先把所有可能答案放进叠加态；
2. 给目标答案加特殊相位，例如加负号；
3. 通过量子门让错误答案相消；
4. 让正确答案相长；
5. 最后测量时更容易得到正确答案。

Grover 算法就是这样。

---

# 6. 两个量子比特的状态

一个量子比特有两个基态：

$$
|0\rangle,\ |1\rangle
$$

两个量子比特就有四个基态：

$$
|00\rangle,\ |01\rangle,\ |10\rangle,\ |11\rangle
$$

一般的两比特状态可以写成：

$$
|\psi\rangle = a_{00}|00\rangle + a_{01}|01\rangle + a_{10}|10\rangle + a_{11}|11\rangle
$$

其中：

- $a_{00}$ 是 $|00\rangle$ 的概率幅；
- $a_{01}$ 是 $|01\rangle$ 的概率幅；
- $a_{10}$ 是 $|10\rangle$ 的概率幅；
- $a_{11}$ 是 $|11\rangle$ 的概率幅。

归一化条件是：

$$
|a_{00}|^2 + |a_{01}|^2 + |a_{10}|^2 + |a_{11}|^2 = 1
$$

如果四个概率幅都是 $\frac{1}{2}$，那么：

$$
|\psi\rangle =
\frac{1}{2}|00\rangle+
\frac{1}{2}|01\rangle+
\frac{1}{2}|10\rangle+
\frac{1}{2}|11\rangle
$$

测量每个状态的概率都是：

$$
\left|\frac{1}{2}\right|^2 = \frac{1}{4}
$$

所以四个状态均匀分布。

---

# 7. 什么是均匀叠加态？

对于两个量子比特，均匀叠加态是：

$$
|s\rangle =
\frac{1}{2}
\left(
|00\rangle + |01\rangle + |10\rangle + |11\rangle
\right)
$$

它的意思是，系统同时包含四种可能状态：

$$
|00\rangle,\ |01\rangle,\ |10\rangle,\ |11\rangle
$$

而且四个状态概率相等。

因为每个状态的概率幅是 $\frac{1}{2}$，所以每个状态的测量概率是：

$$
\frac{1}{4}
$$

---

# 8. $H$ 门是什么？

$H$ 门叫 Hadamard 门，它最常用的作用就是制造叠加态。

它对 $|0\rangle$ 的作用是：

$$
H|0\rangle = \frac{|0\rangle + |1\rangle}{\sqrt{2}}
$$

它对 $|1\rangle$ 的作用是：

$$
H|1\rangle = \frac{|0\rangle - |1\rangle}{\sqrt{2}}
$$

我们常把下面两个状态记作：

$$
|+\rangle = \frac{|0\rangle + |1\rangle}{\sqrt{2}}
$$

$$
|-\rangle = \frac{|0\rangle - |1\rangle}{\sqrt{2}}
$$

所以：

$$
H|0\rangle = |+\rangle
$$

$$
H|1\rangle = |-\rangle
$$

---

# 9. 对两个量子比特都加 $H$ 门会怎样？

初始状态是：

$$
|00\rangle
$$

对第一个量子比特加 $H$：

$$
H|0\rangle = \frac{|0\rangle + |1\rangle}{\sqrt{2}}
$$

对第二个量子比特也加 $H$：

$$
H|0\rangle = \frac{|0\rangle + |1\rangle}{\sqrt{2}}
$$

所以整体变成：

$$
H\otimes H |00\rangle
=
\frac{|0\rangle + |1\rangle}{\sqrt{2}}
\otimes
\frac{|0\rangle + |1\rangle}{\sqrt{2}}
$$

展开：

$$
=
\frac{1}{2}
\left(
|00\rangle + |01\rangle + |10\rangle + |11\rangle
\right)
$$

这就是均匀叠加态。

也就是说：

```text
q0: ──H──
q1: ──H──
```

可以把：

$$
|00\rangle
$$

变成：

$$
\frac{1}{2}
\left(
|00\rangle + |01\rangle + |10\rangle + |11\rangle
\right)
$$

---

# 10. $X$ 门是什么？

$X$ 门类似经典计算里的 NOT 门。

它的作用是翻转 $0$ 和 $1$：

$$
X|0\rangle = |1\rangle
$$

$$
X|1\rangle = |0\rangle
$$

所以：

```text
q: ──X──
```

会把 $|0\rangle$ 变成 $|1\rangle$，把 $|1\rangle$ 变成 $|0\rangle$。

对于两个量子比特，如果对第一个量子比特 $q_0$ 加 $X$：

$$
X(q_0)|00\rangle = |10\rangle
$$

$$
X(q_0)|01\rangle = |11\rangle
$$

$$
X(q_0)|10\rangle = |00\rangle
$$

$$
X(q_0)|11\rangle = |01\rangle
$$

如果对第二个量子比特 $q_1$ 加 $X$：

$$
X(q_1)|00\rangle = |01\rangle
$$

$$
X(q_1)|01\rangle = |00\rangle
$$

$$
X(q_1)|10\rangle = |11\rangle
$$

$$
X(q_1)|11\rangle = |10\rangle
$$

---

# 11. $Z$ 门是什么？

$Z$ 门不改变 $|0\rangle$，但会让 $|1\rangle$ 多一个负号：

$$
Z|0\rangle = |0\rangle
$$

$$
Z|1\rangle = -|1\rangle
$$

所以如果：

$$
|\psi\rangle = a|0\rangle + b|1\rangle
$$

那么：

$$
Z|\psi\rangle = a|0\rangle - b|1\rangle
$$

它改变的是相位，不是把 $0$ 变成 $1$。

比如：

$$
Z|+\rangle
=
Z\frac{|0\rangle + |1\rangle}{\sqrt{2}}
=
\frac{|0\rangle - |1\rangle}{\sqrt{2}}
=
|-\rangle
$$

所以 $Z$ 门可以把 $|+\rangle$ 变成 $|-\rangle$。

---

# 12. CNOT 门是什么？

CNOT 是受控非门。

它有两个量子比特：

- 控制位 control；
- 目标位 target。

如果控制位是 $0$，目标位不变。

如果控制位是 $1$，目标位执行 $X$ 翻转。

假设 $q_0$ 是控制位，$q_1$ 是目标位：

```text
q 0: ──●──
      │
q 1: ──X──
```

作用如下：

$$
CNOT|00\rangle = |00\rangle
$$

$$
CNOT|01\rangle = |01\rangle
$$

$$
CNOT|10\rangle = |11\rangle
$$

$$
CNOT|11\rangle = |10\rangle
$$

简单说：

> 只有当控制位是 $1$ 时，目标位才翻转。

---

# 13. CZ 门是什么？

CZ 是受控 $Z$ 门。

它也有两个量子比特。

如果控制位是 $0$，不做事。

如果控制位是 $1$，对目标位执行 $Z$。

但是 CZ 的最终效果非常简单：

$$
CZ|00\rangle = |00\rangle
$$

$$
CZ|01\rangle = |01\rangle
$$

$$
CZ|10\rangle = |10\rangle
$$

$$
CZ|11\rangle = -|11\rangle
$$

也就是说：

> CZ 只给 $|11\rangle$ 加负号，其它状态不变。

电路写作：

```text
q 0: ──●──
      │
q 1: ──●──
```

有些教材会画成一个实心点和一个 $Z$，也有些会画成两个实心点。

---

# 14. 为什么 CZ 可以用来“标记”目标态？

假设我们有均匀叠加态：

$$
|s\rangle =
\frac{1}{2}
\left (
|00\rangle + |01\rangle + |10\rangle + |11\rangle
\right)
$$

现在加一个 CZ：

$$
CZ|s\rangle
=
\frac{1}{2}
\left (
|00\rangle + |01\rangle + |10\rangle - |11\rangle
\right)
$$

可以看到，只有 $|11\rangle$ 前面的符号变成了负号。

所以 CZ 天然可以标记：

$$
|11\rangle
$$

如果目标态刚好是 $|11\rangle$，最简单：

```text
q 0: ──●──
      │
q 1: ──●──
```

---

# 15. 如果想标记 $|00\rangle$、$|01\rangle$、$|10\rangle$ 怎么办？

CZ 默认只能标记 $|11\rangle$。

但是我们可以用 $X$ 门先把想标记的目标态临时变成 $|11\rangle$，然后用 CZ 加负号，再用 $X$ 门变回去。

这个过程叫做：

> 映射到 $|11\rangle$，加相位，再映射回来。

---

## 15.1 标记 $|00\rangle$

目标：

$$
|00\rangle \rightarrow -|00\rangle
$$

因为 CZ 标记 $|11\rangle$，所以先把 $|00\rangle$ 变成 $|11\rangle$。

对两个比特都加 $X$：

$$
|00\rangle \xrightarrow{X (q_0), X (q_1)} |11\rangle
$$

然后 CZ：

$$
|11\rangle \xrightarrow{CZ} -|11\rangle
$$

再对两个比特加 $X$ 变回：

$$
-|11\rangle \xrightarrow{X (q_0), X (q_1)} -|00\rangle
$$

所以整体就是：

```text
q 0: ──X──●──X──
         │
q 1: ──X──●──X──
```

---

## 15.2 标记 $|01\rangle$

目标：

$$
|01\rangle \rightarrow -|01\rangle
$$

$|01\rangle$ 想变成 $|11\rangle$，只需要把第一位 $0$ 变成 $1$。

所以只对 $q_0$ 加 $X$：

$$
|01\rangle \xrightarrow{X (q_0)} |11\rangle
$$

然后 CZ：

$$
|11\rangle \xrightarrow{CZ} -|11\rangle
$$

最后再对 $q_0$ 加 $X$ 变回来：

$$
-|11\rangle \xrightarrow{X (q_0)} -|01\rangle
$$

电路：

```text
q 0: ──X──●──X──
         │
q 1: ─────●─────
```

---

## 15.3 标记 $|10\rangle$

目标：

$$
|10\rangle \rightarrow -|10\rangle
$$

$|10\rangle$ 想变成 $|11\rangle$，只需要把第二位 $0$ 变成 $1$。

所以只对 $q_1$ 加 $X$：

$$
|10\rangle \xrightarrow{X (q_1)} |11\rangle
$$

然后 CZ：

$$
|11\rangle \xrightarrow{CZ} -|11\rangle
$$

最后再对 $q_1$ 加 $X$ 变回来：

$$
-|11\rangle \xrightarrow{X (q_1)} -|10\rangle
$$

电路：

```text
q 0: ─────●─────
         │
q 1: ──X──●──X──
```

---

## 15.4 标记 $|11\rangle$

目标：

$$
|11\rangle \rightarrow -|11\rangle
$$

CZ 本来就是标记 $|11\rangle$，所以直接用 CZ：

```text
q 0: ──●──
      │
q 1: ──●──
```

---

# 16. 什么是 Oracle？

在 Grover 算法里，Oracle 可以理解成一个“黑盒判断器”。

它不直接告诉你答案是什么，而是把正确答案做一个相位标记。

如果目标态是 $|x\rangle$，Oracle 的作用是：

$$
O|x\rangle = -|x\rangle
$$

对于非目标态 $|y\rangle$：

$$
O|y\rangle = |y\rangle
$$

所以可以写成：

$$
O|y\rangle =
\begin{cases}
-|y\rangle, & y=x\\
|y\rangle, & y\neq x
\end{cases}
$$

在你的作业里，Oracle 分别是：

| 目标态 | Oracle |
|---|---|
| $|00\rangle$ | $X (q_0), X (q_1), CZ,X (q_0), X (q_1)$ |
| $|01\rangle$ | $X (q_0), CZ,X (q_0)$ |
| $|10\rangle$ | $X (q_1), CZ,X (q_1)$ |
| $|11\rangle$ | $CZ$ |

Oracle 的目的不是直接让目标态概率变大。

Oracle 做的事情只是：

> 给目标态加负号。

真正把目标态概率放大的，是后面的 diffusion。

---

# 17. 什么是 diffusion？

Diffusion 也叫扩散算子，或者关于平均值的翻转。

它的作用是：

> 把被 Oracle 加了负号的目标态，通过干涉变成概率幅最大的状态。

对于两个量子比特，均匀叠加态是：

$$
|s\rangle =
\frac{1}{2}
\left (
|00\rangle+|01\rangle+|10\rangle+|11\rangle
\right)
$$

Diffusion 算子可以写成：

$$
D=2|s\rangle\langle s|-I
$$

你现在不用太纠结这个矩阵表达。它的直观意思是：

> 所有概率幅关于它们的平均值做一次翻转。

如果某个状态的概率幅低于平均值，翻转后会变高。

如果某个状态的概率幅高于平均值，翻转后会变低。

---

# 18. diffusion 的核心公式

假设四个状态的概率幅分别是：

$$
a_{00}, a_{01}, a_{10}, a_{11}
$$

它们的平均值是：

$$
\bar{a} = \frac{a_{00}+a_{01}+a_{10}+a_{11}}{4}
$$

Diffusion 后，每个概率幅变成：

$$
a_i' = 2\bar{a} - a_i
$$

这叫做关于平均值翻转。

---

# 19. 用一个例子理解 diffusion

假设目标态是 $|11\rangle$。

第一步，用 $H$ 门制备均匀叠加态：

$$
|\psi_1\rangle =
\frac{1}{2}
\left (
|00\rangle+|01\rangle+|10\rangle+|11\rangle
\right)
$$

概率幅表：

| 状态 | 概率幅 |
|---|---|
| $|00\rangle$ | $\frac{1}{2}$ |
| $|01\rangle$ | $\frac{1}{2}$ |
| $|10\rangle$ | $\frac{1}{2}$ |
| $|11\rangle$ | $\frac{1}{2}$ |

Oracle 给 $|11\rangle$ 加负号：

$$
|\psi_2\rangle =
\frac{1}{2}
\left (
|00\rangle+|01\rangle+|10\rangle-|11\rangle
\right)
$$

概率幅表变成：

| 状态 | 概率幅 |
|---|---|
| $|00\rangle$ | $\frac{1}{2}$ |
| $|01\rangle$ | $\frac{1}{2}$ |
| $|10\rangle$ | $\frac{1}{2}$ |
| $|11\rangle$ | $-\frac{1}{2}$ |

现在求平均值：

$$
\bar{a}
=
\frac{
\frac{1}{2}+\frac{1}{2}+\frac{1}{2}-\frac{1}{2}
}{4}
$$

分子是：

$$
\frac{1}{2}+\frac{1}{2}+\frac{1}{2}-\frac{1}{2}=1
$$

所以：

$$
\bar{a}=\frac{1}{4}
$$

现在对每个概率幅使用 diffusion 公式：

$$
a_i'=2\bar{a}-a_i
$$

对于非目标态，比如 $|00\rangle$：

$$
a_{00}'=2\times\frac{1}{4}-\frac{1}{2}
$$

$$
a_{00}'=\frac{1}{2}-\frac{1}{2}=0
$$

同理：

$$
a_{01}'=0
$$

$$
a_{10}'=0
$$

对于目标态 $|11\rangle$：

$$
a_{11}'=2\times\frac{1}{4}-\left (-\frac{1}{2}\right)
$$

$$
a_{11}'=\frac{1}{2}+\frac{1}{2}=1
$$

所以最后变成：

$$
|\psi_3\rangle = |11\rangle
$$

测量一定得到：

$$
11
$$

这就是 Grover 算法在 $2$ 个量子比特、$1$ 个目标答案时的效果。

---

# 20. diffusion 电路为什么是这个样子？

你的课件中给出的 diffusion 是：

```text
q 0: ──H──Z──●──H──
             │
q 1: ──H──Z──●──H──
```

也就是：

$$
D=(H\otimes H)(Z\otimes Z) CZ (H\otimes H)
$$

按照门执行顺序，从左到右是：

1. 对 $q_0, q_1$ 加 $H$；
2. 对 $q_0, q_1$ 加 $Z$；
3. 加 CZ；
4. 对 $q_0, q_1$ 再加 $H$。

写成门序列是：

```text
H (q 0), H (q 1)
Z (q 0), Z (q 1)
CZ (q 0, q 1)
H (q 0), H (q 1)
```

这个模块的作用就是实现：

$$
D=2|s\rangle\langle s|-I
$$

也就是关于均匀叠加态 $|s\rangle$ 的反射。

你可以暂时不用深究它怎么推导出来，只要记住：

> 这个固定模块就是两比特 Grover 算法里的 diffusion，它负责把被加负号的目标态概率放大。

---

# 21. 为什么“加负号”之后 diffusion 能把目标态放大？

这是整个算法最核心的地方。

假设我们有 $4$ 个状态，初始概率幅都是：

$$
\frac{1}{2}
$$

Oracle 把目标态变成：

$$
-\frac{1}{2}
$$

于是整体变成类似：

$$
\left[
\frac{1}{2},
\frac{1}{2},
\frac{1}{2},
-\frac{1}{2}
\right]
$$

平均值变成：

$$
\frac{1}{4}
$$

然后 diffusion 做关于平均值的翻转。

非目标态：

$$
\frac{1}{2}
$$

关于平均值 $\frac{1}{4}$ 翻转，会变成：

$$
0
$$

目标态：

$$
-\frac{1}{2}
$$

关于平均值 $\frac{1}{4}$ 翻转，会变成：

$$
1
$$

所以：

$$
\left[
\frac{1}{2},
\frac{1}{2},
\frac{1}{2},
-\frac{1}{2}
\right]
\xrightarrow{D}
\left[
0,
0,
0,
1
\right]
$$

最终测量就一定得到目标态。

---

# 22. 什么是 Phase Kickback？

Phase Kickback 中文常翻译为**相位回移**或者**相位反冲**。

它的核心思想是：

> 一个受控门表面上好像作用在目标位上，但如果目标位处在特殊本征态中，产生的相位会“回到”控制位上。

我们先看 CNOT。

CNOT 的作用是：

如果控制位是 $1$，目标位执行 $X$。

如果控制位是 $0$，目标位不变。

也就是：

$$
CNOT|0\rangle|\phi\rangle = |0\rangle|\phi\rangle
$$

$$
CNOT|1\rangle|\phi\rangle = |1\rangle X|\phi\rangle
$$

现在关键来了。

$X$ 门有两个特殊本征态：

$$
|+\rangle = \frac{|0\rangle+|1\rangle}{\sqrt{2}}
$$

$$
|-\rangle = \frac{|0\rangle-|1\rangle}{\sqrt{2}}
$$

它们满足：

$$
X|+\rangle=|+\rangle
$$

$$
X|-\rangle=-|-\rangle
$$

这说明：

- $|+\rangle$ 被 $X$ 作用后不变；
- $|-\rangle$ 被 $X$ 作用后会多一个负号。

---

## 22.1 CNOT 对 $|+\rangle$ 的作用

如果目标位是 $|+\rangle$：

$$
CNOT|0\rangle|+\rangle = |0\rangle|+\rangle
$$

$$
CNOT|1\rangle|+\rangle = |1\rangle X|+\rangle = |1\rangle|+\rangle
$$

所以没有产生负号。

---

## 22.2 CNOT 对 $|-\rangle$ 的作用

如果目标位是 $|-\rangle$：

$$
CNOT|0\rangle|-\rangle = |0\rangle|-\rangle
$$

因为控制位是 $0$，不翻转。

而：

$$
CNOT|1\rangle|-\rangle = |1\rangle X|-\rangle
$$

由于：

$$
X|-\rangle=-|-\rangle
$$

所以：

$$
CNOT|1\rangle|-\rangle = -|1\rangle|-\rangle
$$

现在看，如果控制位是一般叠加态：

$$
a|0\rangle+b|1\rangle
$$

目标位是：

$$
|-\rangle
$$

那么：

$$
CNOT\left (a|0\rangle+b|1\rangle\right)|-\rangle
=
a|0\rangle|-\rangle-b|1\rangle|-\rangle
$$

提取公共的目标位：

$$
=
\left (a|0\rangle-b|1\rangle\right)|-\rangle
$$

这说明：

$$
a|0\rangle+b|1\rangle
$$

变成了：

$$
a|0\rangle-b|1\rangle
$$

也就是控制位上发生了 $Z$ 相位翻转。

所以虽然 CNOT 表面上对目标位做 $X$，但由于目标位是 $|-\rangle$，负号“回移”到了控制位上。

这就是 Phase Kickback。

---

# 23. Phase Kickback 和 CZ 的关系

CZ 可以看成：

> 如果控制位是 $1$，就对目标位执行 $Z$。

也就是：

$$
CZ = |0\rangle\langle 0|\otimes I + |1\rangle\langle 1|\otimes Z
$$

由于：

$$
Z|0\rangle = |0\rangle
$$

$$
Z|1\rangle = -|1\rangle
$$

所以当两个量子比特都是 $1$ 时：

$$
CZ|11\rangle=-|11\rangle
$$

这就是直接的相位标记。

在 Grover 算法中，我们用 CZ 给某个状态加负号，本质上就是在做一种受控相位操作。

如果目标不是 $|11\rangle$，就先用 $X$ 门把目标态变成 $|11\rangle$，再使用 CZ。

---

# 24. 本作业完整逻辑重新梳理

你的作业分成四个算法。

四个算法的共同结构都是：

```text
初始 |00>
↓
H (q 0), H (q 1)
↓
Oracle 标记某个目标态
↓
Diffusion
↓
Measure
```

数学上：

$$
|00\rangle
\xrightarrow{H\otimes H}
\frac{1}{2}
\left (
|00\rangle+|01\rangle+|10\rangle+|11\rangle
\right)
$$

然后 Oracle 把某一个目标态加负号。

例如目标是 $|10\rangle$：

$$
\frac{1}{2}
\left (
|00\rangle+|01\rangle+|10\rangle+|11\rangle
\right)
\xrightarrow{Oracle}
\frac{1}{2}
\left (
|00\rangle+|01\rangle-|10\rangle+|11\rangle
\right)
$$

然后 diffusion：

$$
\frac{1}{2}
\left (
|00\rangle+|01\rangle-|10\rangle+|11\rangle
\right)
\xrightarrow{D}
|10\rangle
$$

最后测量得到：

$$
10
$$

---

# 25. 四个目标态分别怎么做？

## 25.1 目标态 $|00\rangle$

Oracle：

```text
q 0: ──X──●──X──
         │
q 1: ──X──●──X──
```

完整电路：

```text
q 0: ──H──X──●──X──H──Z──●──H──M
             │           │
q 1: ──H──X──●──X──H──Z──●──H──M
```

状态变化：

$$
|00\rangle
\xrightarrow{H, H}
\frac{1}{2}
\left (
|00\rangle+|01\rangle+|10\rangle+|11\rangle
\right)
$$

Oracle：

$$
\frac{1}{2}
\left (
-|00\rangle+|01\rangle+|10\rangle+|11\rangle
\right)
$$

Diffusion：

$$
|00\rangle
$$

测量得到：

$$
00
$$

---

## 25.2 目标态 $|01\rangle$

Oracle：

```text
q 0: ──X──●──X──
         │
q 1: ─────●─────
```

完整电路：

```text
q 0: ──H──X──●──X──H──Z──●──H──M
             │           │
q 1: ──H─────●─────H──Z──●──H──M
```

状态变化：

$$
|00\rangle
\xrightarrow{H, H}
\frac{1}{2}
\left (
|00\rangle+|01\rangle+|10\rangle+|11\rangle
\right)
$$

Oracle：

$$
\frac{1}{2}
\left (
|00\rangle-|01\rangle+|10\rangle+|11\rangle
\right)
$$

Diffusion：

$$
|01\rangle
$$

测量得到：

$$
01
$$

---

## 25.3 目标态 $|10\rangle$

Oracle：

```text
q 0: ─────●─────
         │
q 1: ──X──●──X──
```

完整电路：

```text
q 0: ──H─────●─────H──Z──●──H──M
             │           │
q 1: ──H──X──●──X──H──Z──●──H──M
```

状态变化：

$$
|00\rangle
\xrightarrow{H, H}
\frac{1}{2}
\left (
|00\rangle+|01\rangle+|10\rangle+|11\rangle
\right)
$$

Oracle：

$$
\frac{1}{2}
\left (
|00\rangle+|01\rangle-|10\rangle+|11\rangle
\right)
$$

Diffusion：

$$
|10\rangle
$$

测量得到：

$$
10
$$

---

## 25.4 目标态 $|11\rangle$

Oracle：

```text
q 0: ──●──
      │
q 1: ──●──
```

完整电路：

```text
q 0: ──H──●──H──Z──●──H──M
         │        │
q 1: ──H──●──H──Z──●──H──M
```

状态变化：

$$
|00\rangle
\xrightarrow{H, H}
\frac{1}{2}
\left (
|00\rangle+|01\rangle+|10\rangle+|11\rangle
\right)
$$

Oracle：

$$
\frac{1}{2}
\left (
|00\rangle+|01\rangle+|10\rangle-|11\rangle
\right)
$$

Diffusion：

$$
|11\rangle
$$

测量得到：

$$
11
$$

---

# 26. 四套算法总结表

| 目标态 | 目标态如何变成 $|11\rangle$ | Oracle 门序列 | Oracle 后状态 | Diffusion 后 |
|---|---|---|---|---|
| $|00\rangle$ | 两位 $0$ 都翻成 $1$ | $X (q_0), X (q_1), CZ,X (q_0), X (q_1)$ | $\frac{1}{2}\left (-|00\rangle+|01\rangle+|10\rangle+|11\rangle\right)$ | $|00\rangle$ |
| $|01\rangle$ | 第一位 $0$ 翻成 $1$ | $X (q_0), CZ,X (q_0)$ | $\frac{1}{2}\left (|00\rangle-|01\rangle+|10\rangle+|11\rangle\right)$ | $|01\rangle$ |
| $|10\rangle$ | 第二位 $0$ 翻成 $1$ | $X (q_1), CZ,X (q_1)$ | $\frac{1}{2}\left (|00\rangle+|01\rangle-|10\rangle+|11\rangle\right)$ | $|10\rangle$ |
| $|11\rangle$ | 已经是 $|11\rangle$ | $CZ$ | $\frac{1}{2}\left (|00\rangle+|01\rangle+|10\rangle-|11\rangle\right)$ | $|11\rangle$ |

---

# 27. 最后用一句话理解整个作业

这四个电路做的都是同一件事：

> 先把 $|00\rangle$ 变成四个状态的均匀叠加；  
> 再用 Oracle 给某一个目标状态加负号；  
> 然后用 diffusion 把这个负号变成概率放大；  
> 最后测量得到目标态。

可以总结为：

$$
|00\rangle
\rightarrow
\frac{1}{2}
\left (
|00\rangle+|01\rangle+|10\rangle+|11\rangle
\right)
\rightarrow
\text{给目标态加负号}
\rightarrow
|目标态\rangle
\rightarrow
\text{测量得到目标态}
$$

所以，Grover 算法不是一开始就“知道答案”，而是通过：

1. **叠加态**：同时包含所有可能答案；
2. **Oracle**：给正确答案加负号；
3. **Diffusion**：通过干涉放大正确答案；
4. **测量**：得到正确答案。

这就是你这次作业背后的完整原理。