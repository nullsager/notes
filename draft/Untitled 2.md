可以。下面用 **Qiskit** 实现四套两量子比特 Grover 电路，分别标记：

$$
|00\rangle,\quad |01\rangle,\quad |10\rangle,\quad |11\rangle
$$

每套算法都包含：

1. 使用 $H$ 门制备均匀叠加态；
2. 使用 $X$ 和 CZ 构造 Oracle；
3. 使用相同的 diffusion；
4. 测量并统计结果；
5. 输出各阶段的量子态，方便检查目标态是否被正确加上负号。

---

# 一、安装运行环境

在终端中安装 Qiskit：

```bash
pip install qiskit qiskit-aer matplotlib pylatexenc
```

其中：

- `qiskit`：构建量子电路；
- `qiskit-aer`：运行模拟器；
- `matplotlib`：绘制电路图；
- `pylatexenc`：支持 Qiskit 的电路绘图。

---

# 二、完整代码

将下面代码保存为 `two_qubit_grover.py`：

```python
from qiskit import QuantumCircuit, transpile
from qiskit.quantum_info import Statevector
from qiskit_aer import AerSimulator


# ============================================================
# 基本设置
# ============================================================

# 如果设置为 False：
#     diffusion 中直接使用 CZ
#
# 如果设置为 True：
#     diffusion 中使用 H-CNOT-H 分解 CZ
#
# 设置为 True 后，同一套作业中就会出现：
# H、X、Z、CZ、CNOT 五种门。
DECOMPOSE_DIFFUSION_CZ = False


# ============================================================
# 逻辑 CZ
# ============================================================

def apply_logical_cz(
    circuit,
    q0,
    q1,
    use_cnot_decomposition=False
):
    """
    在 q0 和 q1 之间实现逻辑 CZ。

    如果 use_cnot_decomposition=False，直接使用 CZ。

    如果 use_cnot_decomposition=True，使用：
        H(q1) -> CNOT(q0, q1) -> H(q1)

    因为：
        CZ = (I ⊗ H) CNOT (I ⊗ H)
    """

    if use_cnot_decomposition:
        circuit.h(q1)
        circuit.cx(q0, q1)
        circuit.h(q1)
    else:
        circuit.cz(q0, q1)


# ============================================================
# 均匀叠加态
# ============================================================

def apply_superposition(circuit):
    """
    从 |00> 制备均匀叠加态：

        1/2 (|00> + |01> + |10> + |11>)
    """

    circuit.h(0)
    circuit.h(1)


# ============================================================
# Oracle
# ============================================================

def apply_oracle(circuit, target):
    """
    给指定目标态增加负号相位。

    参数 target 按照 |q0 q1> 的顺序书写，例如：
        "00"
        "01"
        "10"
        "11"

    实现方法：
    1. 对目标态中为 0 的量子位执行 X；
    2. 此时目标态被映射成 |11>；
    3. 使用 CZ 给 |11> 加负号；
    4. 再次执行相同的 X，恢复状态标签。
    """

    if target not in {"00", "01", "10", "11"}:
        raise ValueError(
            "target 必须是 '00'、'01'、'10' 或 '11'"
        )

    # 将目标态映射到 |11>
    if target[0] == "0":
        circuit.x(0)

    if target[1] == "0":
        circuit.x(1)

    # CZ 只给 |11> 加负号
    circuit.cz(0, 1)

    # 反计算：将状态标签恢复
    if target[0] == "0":
        circuit.x(0)

    if target[1] == "0":
        circuit.x(1)


# ============================================================
# Diffusion 扩散算子
# ============================================================

def apply_diffusion(
    circuit,
    decompose_cz=False
):
    """
    两量子比特 diffusion：

        D = (H⊗H)(Z⊗Z)CZ(H⊗H)

    对应门序列：

        H(q0), H(q1)
        Z(q0), Z(q1)
        CZ(q0, q1)
        H(q0), H(q1)

    它实现：

        D = 2|s><s| - I

    即概率幅关于平均值翻转：

        a_i' = 2*a_average - a_i
    """

    # 第一组 H：转换坐标系
    circuit.h(0)
    circuit.h(1)

    # 关于 |00> 的相位反射
    circuit.z(0)
    circuit.z(1)

    apply_logical_cz(
        circuit,
        0,
        1,
        use_cnot_decomposition=decompose_cz
    )

    # 第二组 H：转换回原坐标系
    circuit.h(0)
    circuit.h(1)


# ============================================================
# 构建一套 Grover 电路
# ============================================================

def build_grover_circuit(
    target,
    decompose_diffusion_cz=False
):
    """
    构建完整的两量子比特 Grover 电路。

    返回：
        state_circuit：测量前的量子电路
        measured_circuit：包含测量的完整电路
        stages：各阶段的 Statevector
    """

    # 先创建不带测量的量子电路
    state_circuit = QuantumCircuit(2)

    stages = {}

    # 初始状态
    stages["initial"] = Statevector.from_instruction(
        state_circuit
    )

    # 第一步：制备均匀叠加态
    apply_superposition(state_circuit)

    stages["after_superposition"] = (
        Statevector.from_instruction(state_circuit)
    )

    state_circuit.barrier()

    # 第二步：Oracle 相位标记
    apply_oracle(state_circuit, target)

    stages["after_oracle"] = (
        Statevector. from_instruction (state_circuit)
    )

    state_circuit.barrier ()

    # 第三步：Diffusion
    apply_diffusion (
        state_circuit,
        decompose_cz=decompose_diffusion_cz
    )

    stages["after_diffusion"] = (
        Statevector. from_instruction (state_circuit)
    )

    # 创建带两个经典比特的测量电路
    measured_circuit = QuantumCircuit (2, 2)

    measured_circuit.compose (
        state_circuit,
        qubits=[0, 1],
        inplace=True
    )

    measured_circuit.barrier ()

    # Qiskit 的测量字符串默认显示为 c 1 c 0。
    #
    # 为了让最终显示顺序是 q 0 q 1：
    #     q 0 -> c 1
    #     q 1 -> c 0
    measured_circuit.measure (0, 1)
    measured_circuit.measure (1, 0)

    return state_circuit, measured_circuit, stages


# ============================================================
# 按照 |q 0 q 1> 顺序输出 Statevector
# ============================================================

def print_statevector_q 0 q 1 (statevector, title):
    """
    Qiskit 的内部基态顺序一般是 |q 1 q 0>。

    本作业统一使用 |q 0 q 1>，所以这里手动转换并输出。
    """

    print (title)

    found_nonzero = False

    for q 0 in [0, 1]:
        for q 1 in [0, 1]:

            # Qiskit 中 q 0 是最低有效位
            index = q 0 + 2 * q 1

            amplitude = statevector. data[index]

            if abs (amplitude) > 1 e-10:
                found_nonzero = True

                # 清除非常小的浮点误差
                real_part = (
                    0.0
                    if abs (amplitude. real) < 1 e-10
                    else amplitude. real
                )

                imag_part = (
                    0.0
                    if abs (amplitude. imag) < 1 e-10
                    else amplitude. imag
                )

                cleaned_amplitude = complex (
                    real_part,
                    imag_part
                )

                probability = abs (amplitude) ** 2

                print (
                    f"  |{q 0}{q 1}>: "
                    f"概率幅 = {cleaned_amplitude}, "
                    f"概率 = {probability:. 6 f}"
                )

    if not found_nonzero:
        print ("  没有找到非零概率幅。")

    print ()


# ============================================================
# 在模拟器上执行电路
# ============================================================

def run_circuit (circuit, shots=1024):
    """
    使用 AerSimulator 执行测量电路。
    """

    simulator = AerSimulator ()

    compiled_circuit = transpile (
        circuit,
        simulator
    )

    job = simulator.run (
        compiled_circuit,
        shots=shots
    )

    result = job.result ()

    return result. get_counts ()


# ============================================================
# 执行四套算法
# ============================================================

def main ():
    targets = ["00", "01", "10", "11"]

    shots = 1024

    all_results = {}

    for target in targets:
        print ("=" * 70)
        print (f"当前目标态：|{target}>")
        print ("=" * 70)
        print ()

        (
            state_circuit,
            measured_circuit,
            stages
        ) = build_grover_circuit (
            target,
            decompose_diffusion_cz=DECOMPOSE_DIFFUSION_CZ
        )

        # 输出各阶段状态
        print_statevector_q 0 q 1 (
            stages["initial"],
            "1. 初始状态："
        )

        print_statevector_q 0 q 1 (
            stages["after_superposition"],
            "2. H 门制备均匀叠加态后："
        )

        print_statevector_q 0 q 1 (
            stages["after_oracle"],
            f"3. Oracle 标记 |{target}> 后："
        )

        print_statevector_q 0 q 1 (
            stages["after_diffusion"],
            "4. Diffusion 后："
        )

        # 输出电路
        print ("5. 完整测量电路：")
        print (measured_circuit.draw (output="text"))
        print ()

        # 模拟测量
        counts = run_circuit (
            measured_circuit,
            shots=shots
        )

        all_results[target] = counts

        print (f"6. 测量次数：{shots}")
        print (f"测量统计：{counts}")
        print ()

    # 最终汇总
    print ("=" * 70)
    print ("四套算法测量结果汇总")
    print ("=" * 70)

    for target, counts in all_results.items ():
        print (
            f"目标态 |{target}> -> 测量统计 {counts}"
        )


if __name__ == "__main__":
    main ()
```

---

# 三、运行代码

在终端进入代码所在目录，执行：

```bash
python two_qubit_grover. py
```

理想模拟器中的结果应该类似：

```text
目标态 |00> -> 测量统计 {'00': 1024}
目标态 |01> -> 测量统计 {'01': 1024}
目标态 |10> -> 测量统计 {'10': 1024}
目标态 |11> -> 测量统计 {'11': 1024}
```

也就是：

$$
P (00)=1
$$

$$
P (01)=1
$$

$$
P (10)=1
$$

$$
P (11)=1
$$

当然，这里四个公式分别对应四套不同电路，不是同一套电路同时得到四个概率为 $1$。

---

# 四、代码中的 Oracle 是怎样构造的？

核心函数是：

```python
def apply_oracle (circuit, target):
    if target[0] == "0":
        circuit.x (0)

    if target[1] == "0":
        circuit.x (1)

    circuit.cz (0, 1)

    if target[0] == "0":
        circuit.x (0)

    if target[1] == "0":
        circuit.x (1)
```

它采用的规则是：

> 目标态中哪一位是 $0$，就在对应量子比特上前后各添加一个 $X$。

---

## 标记 $|00\rangle$

两位都是 $0$，所以两位都需要 $X$：

```python
circuit.x (0)
circuit.x (1)

circuit.cz (0, 1)

circuit.x (0)
circuit.x (1)
```

对应：

$$
O_{00}=(X\otimes X) CZ (X\otimes X)
$$

作用结果：

$$
|00\rangle\rightarrow-|00\rangle
$$

---

## 标记 $|01\rangle$

$q_0=0$，所以只对 $q_0$ 添加 $X$：

```python
circuit.x (0)
circuit.cz (0, 1)
circuit.x (0)
```

作用结果：

$$
|01\rangle\rightarrow-|01\rangle
$$

---

## 标记 $|10\rangle$

$q_1=0$，所以只对 $q_1$ 添加 $X$：

```python
circuit.x (1)
circuit.cz (0, 1)
circuit.x (1)
```

作用结果：

$$
|10\rangle\rightarrow-|10\rangle
$$

---

## 标记 $|11\rangle$

目标态已经是 $|11\rangle$，直接使用 CZ：

```python
circuit.cz (0, 1)
```

作用结果：

$$
|11\rangle\rightarrow-|11\rangle
$$

---

# 五、代码中的 Diffusion

Diffusion 函数是：

```python
def apply_diffusion (circuit, decompose_cz=False):
    circuit.h (0)
    circuit.h (1)

    circuit.z (0)
    circuit.z (1)

    apply_logical_cz (
        circuit,
        0,
        1,
        use_cnot_decomposition=decompose_cz
    )

    circuit.h (0)
    circuit.h (1)
```

对应门序列：

```text
H (q 0), H (q 1)
Z (q 0), Z (q 1)
CZ (q 0, q 1)
H (q 0), H (q 1)
```

对应数学表达式：

$$
D=(H\otimes H)(Z\otimes Z) CZ (H\otimes H)
$$

它实现：

$$
D=2|s\rangle\langle s|-I
$$

其中：

$$
|s\rangle=
\frac{1}{2}
\left (
|00\rangle+|01\rangle+|10\rangle+|11\rangle
\right)
$$

作用在概率幅上就是：

$$
a_i'=2\bar a-a_i
$$

---

# 六、怎样在作业中同时展示 CNOT？

默认设置为：

```python
DECOMPOSE_DIFFUSION_CZ = False
```

此时 diffusion 直接使用 CZ，与课件电路完全一致。

如果老师要求电路中展示 CNOT，可以改成：

```python
DECOMPOSE_DIFFUSION_CZ = True
```

这样 diffusion 中的逻辑 CZ 将被替换为：

```text
q 0: ─────●─────
         │
q 1: ──H──X──H──
```

对应代码：

```python
circuit.h (q 1)
circuit.cx (q 0, q 1)
circuit.h (q 1)
```

原因是：

$$
CZ=(I\otimes H) CNOT (I\otimes H)
$$

设置为 `True` 后：

- Oracle 使用 CZ；
- diffusion 中使用 CNOT 实现逻辑 CZ；
- 四套算法整体会用到 $H$、$X$、$Z$、CZ、CNOT；
- 实现的数学功能没有改变；
- 四套算法仍然使用相同的 diffusion。

---

# 七、需要特别注意 Qiskit 的比特顺序

本作业约定状态写作：

$$
|q_0 q_1\rangle
$$

但 Qiskit 内部通常按照：

$$
|q_1 q_0\rangle
$$

显示状态和测量结果。

为了避免混乱，代码中特意使用了：

```python
measured_circuit.measure (0, 1)
measured_circuit.measure (1, 0)
```

即：

```text
q 0 -> c 1
q 1 -> c 0
```

这样 Qiskit 输出的测量字符串就可以直接按照本作业的：

$$
q_0 q_1
$$

来阅读。

例如目标态是 $|01\rangle$，输出就是：

```text
{'01': 1024}
```

而不会变成容易混淆的：

```text
{'10': 1024}
```

---

# 八、只需要电路、不需要中间状态的简化版本

如果完整代码对你来说太长，也可以先使用下面这个简化版本：

```python
from qiskit import QuantumCircuit, transpile
from qiskit_aer import AerSimulator


def oracle (qc, target):
    # 把目标态映射到 |11>
    if target[0] == "0":
        qc.x (0)

    if target[1] == "0":
        qc.x (1)

    # 给 |11> 加负号
    qc.cz (0, 1)

    # 恢复原状态标签
    if target[0] == "0":
        qc.x (0)

    if target[1] == "0":
        qc.x (1)


def diffusion (qc):
    qc.h (0)
    qc.h (1)

    qc.z (0)
    qc.z (1)

    qc.cz (0, 1)

    qc.h (0)
    qc.h (1)


def build_circuit (target):
    qc = QuantumCircuit (2, 2)

    # 第一步：均匀叠加
    qc.h (0)
    qc.h (1)

    qc.barrier ()

    # 第二步：Oracle
    oracle (qc, target)

    qc.barrier ()

    # 第三步：Diffusion
    diffusion (qc)

    qc.barrier ()

    # 第四步：测量
    # 使输出字符串按照 q 0 q 1 显示
    qc.measure (0, 1)
    qc.measure (1, 0)

    return qc


simulator = AerSimulator ()

for target in ["00", "01", "10", "11"]:
    circuit = build_circuit (target)

    compiled = transpile (circuit, simulator)

    result = simulator.run (
        compiled,
        shots=1024
    ). result ()

    counts = result. get_counts ()

    print (f"目标态 |{target}>")
    print (circuit.draw (output="text"))
    print ("测量结果：", counts)
    print ("-" * 60)
```

这个版本最适合直接演示四套电路。

---

# 九、实验报告中可以写的结论

可以在作业报告中写：

> 本实验使用两个 $H$ 门将初始态 $|00\rangle$ 制备为四个计算基态的均匀叠加态。随后，根据目标态中 $0$ 所处的位置添加 $X$ 门，将目标态临时映射为 $|11\rangle$，再利用 CZ 对 $|11\rangle$ 分支进行相位翻转，最后使用相同的 $X$ 门完成反计算，从而保证只有目标态的概率幅被乘以 $-1$。
>
> 四套算法使用相同的 diffusion 模块。该模块由 $H\otimes H$、$Z\otimes Z$、CZ 和 $H\otimes H$ 构成，实现 $D=2|s\rangle\langle s|-I$，即概率幅关于平均值翻转。在 Oracle 执行后，目标态概率幅为 $-\frac 12$，其余状态概率幅为 $\frac 12$；经过 diffusion 后，三个非目标态的概率幅变为 $0$，目标态概率幅变为 $1$。
>
> 在理想模拟器中分别运行四套电路，最终以 $100\%$ 的理论概率测量得到 $00$、$01$、$10$ 和 $11$，验证了相位标记、量子干涉以及 Grover diffusion 概率幅放大的作用。