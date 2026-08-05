# KVFlow 论文详细解读

论文：**KVFlow: Efficient Prefix Caching for Accelerating LLM-Based Multi-Agent Workflows**

- 作者：Zaifeng Pan、Ajjkumar Patel、Zhengding Hu 等
- 单位：UCSD、AWS
- 发表：**NeurIPS 2025**
- 论文：[NeurIPS PDF](https://proceedings.neurips.cc/paper_files/paper/2025/file/b7971d31a7d5eb0f1eed2f8f6f368195-Paper-Conference.pdf)｜[arXiv HTML](https://arxiv.org/html/2507.07400v1)
- 代码：[GitHub：PanZaifeng/KVFlow](https://github.com/PanZaifeng/KVFlow)

---

## 一句话概括

KVFlow 的核心思想是：

> **既然多智能体工作流通常提前知道“接下来可能执行哪个 Agent”，就不应该再用只看历史访问的 LRU 管理 KV Cache，而应该根据未来执行顺序保留、淘汰和预取缓存。**

它主要做了两件事：

1. **工作流感知的 KV Cache 淘汰**：越快要执行的 Agent，其固定提示词缓存越应该留在 GPU。
2. **提前预取与状态感知调度**：在当前 Agent 计算时，把下一个 Agent 的 KV Cache 从 CPU 异步搬回 GPU；如果还没搬完，就先执行其他已经就绪的请求。

---

# 1. 论文要解决什么问题？

## 1.1 多智能体工作流为什么慢？

一个典型的多 Agent 系统可能是：

```text
Planner → Executor → Expresser → Reviewer
   ↑                                  |
   └──────────────────────────────────┘
```

每个 Agent 都有自己的固定提示词，例如：

```text
你是一个 Planner。
你的任务是分析用户需求、分解任务，并为 Executor 制定计划。
请遵循以下格式……
下面是若干 few-shot 示例……
```

一次 Agent 调用的输入通常可以拆成：

```text
[固定前缀] + [当前任务的动态内容]
```

其中：

- **固定前缀**：角色、规则、工具说明、few-shot 示例；
- **动态后缀**：用户问题、其他 Agent 的输出、当前中间状态；
- 固定前缀可能有几千甚至上万 token；
- 同一个 Agent 会在循环中被反复调用。

如果每次都重新计算固定前缀，就会重复执行昂贵的 prefill。

---

## 1.2 什么是 KV Cache？

Transformer 的每层自注意力都会为输入 token 计算 Key 和 Value：

$$
K_l = XW_l^K,\qquad V_l = XW_l^V
$$

自回归生成时，过去 token 的 $K$、$V$ 不会改变，因此可以把它们缓存起来。

如果请求具有相同前缀：

```text
请求 1：[固定 Agent 提示词] + [任务 A]
请求 2：[固定 Agent 提示词] + [任务 B]
```

那么第二次请求可以直接复用固定提示词对应的 KV Cache，只计算新的动态部分。

KV Cache 的近似大小为：

$$
M_{\mathrm{KV}}
=
T \times L \times 2 \times H_{\mathrm{KV}}
\times D_{\mathrm{head}} \times B
$$

其中：

- $T$：token 数；
- $L$：Transformer 层数；
- $2$：Key 和 Value 两份；
- $H_{\mathrm{KV}}$：KV head 数；
- $D_{\mathrm{head}}$：每个 head 的维度；
- $B$：每个元素的字节数。

以 Llama-3.1-8B 的 FP16 KV Cache 为例：

- $L=32$；
- $H_{\mathrm{KV}}=8$；
- $D_{\mathrm{head}}=128$；
- $B=2$ 字节。

每个 token 的 KV Cache 约为：

$$
32 \times 2 \times 8 \times 128 \times 2
=
131072\ \text{bytes}
\approx 128\ \text{KB}
$$

因此，一个 $8192$ token 的前缀大约需要：

$$
8192 \times 128\ \text{KB}
\approx 1\ \text{GB}
$$

如果有十几个不同 Agent，GPU 很快就装不下所有固定前缀。

---

# 2. 现有方法为什么不够好？

## 2.1 Prefix Cache 通常如何组织？

SGLang 等系统会使用 Radix Tree，也就是基数树，管理共享前缀：

```text
root
 └── 公共系统提示词
      ├── Planner 固定提示词
      │    └── Planner 动态内容
      ├── Executor 固定提示词
      │    └── Executor 动态内容
      └── Reviewer 固定提示词
           └── Reviewer 动态内容
```

不同 Agent 如果共享一部分提示词，就可以共享树上的公共节点。

当新请求到来时：

1. 从根节点开始匹配最长前缀；
2. 已匹配节点的 KV Cache 直接复用；
3. 只为未匹配部分执行 prefill；
4. GPU 显存不足时，淘汰部分缓存节点。

---

## 2.2 LRU 的问题

传统系统通常使用 LRU：

> 最久没被访问的缓存最先被淘汰。

但多 Agent 工作流中，**最近访问时间不等于未来复用价值**。

例如工作流按以下顺序循环：

```text
Planner → Executor → Expresser → Reviewer → Planner
```

当前正在执行 Executor 时：

- Expresser 已经很久没被访问；
- 按 LRU，它可能是首要淘汰对象；
- 但 Expresser 恰恰是下一个要执行的 Agent。

与此同时：

- Executor 刚被访问；
- LRU 认为它很重要；
- 但 Executor 可能要到下一轮循环才会再次使用。

因此 LRU 会出现一种反直觉行为：

```text
马上要用的缓存被淘汰；
刚刚用完、短期不会再用的缓存被保留。
```

这会导致：

1. **重新计算**：从头执行长前缀 prefill；
2. **响应式换入**：如果缓存已备份到 CPU，则等请求真正执行时再从 CPU 搬回 GPU；
3. GPU 因等待缓存而空闲。

---

## 2.3 CPU 二级缓存也没有完全解决问题

SGLang HiCache 可以把被淘汰的 KV Cache备份到 CPU：

```text
GPU KV Cache ↔ CPU KV Cache
```

CPU 换入通常比重新计算快，但传统方法是**响应式加载**：

```text
发现 Executor 要执行
        ↓
发现 KV Cache 不在 GPU
        ↓
从 CPU 加载到 GPU
        ↓
等待传输结束
        ↓
开始生成
```

问题是 CPU 到 GPU 的 PCIe 传输仍然会增加请求延迟。

因此，KVFlow 不只问：

> 淘汰谁？

它还进一步问：

> 能不能在请求真正到来前，就把缓存提前搬回来？

---

# 3. KVFlow 的整体设计

KVFlow 的逻辑可以概括为：

```text
Agent 工作流结构
        ↓
Agent Step Graph
        ↓
计算每个 Agent 距离执行还有多少步
        ↓
映射到 Radix Tree 的缓存节点
        ↓
决定：
  1. 哪些节点应保留
  2. 哪些节点应淘汰
  3. 哪些节点应提前从 CPU 加载
        ↓
状态感知的请求调度
```

系统由三个关键部分构成：

| 模块 | 作用 |
|---|---|
| Agent Step Graph | 描述 Agent 之间的执行依赖 |
| Workflow-aware eviction | 根据未来执行距离淘汰 KV 节点 |
| Overlapped prefetching | 提前把即将使用的 KV Cache 搬回 GPU |

---

# 4. Agent Step Graph

## 4.1 为什么普通 DAG 不够？

多智能体工作流不一定只是简单的串行 DAG，它可能包含：

- 条件分支；
- 多路并行；
- 同步屏障；
- 任意一个前驱完成即可触发；
- 所有前驱都完成后才能触发；
- 循环。

例如：

```text
          ┌→ Executor 1 ─┐
Planner ──┤              ├→ Expresser
          └→ Executor 2 ─┘
```

这里存在两种不同语义。

### 情况一：两个 Executor 都完成才能执行 Expresser

那么 Expresser 的最早执行时间取决于较慢的一条路径：

$$
S_{\mathrm{Expresser}}
=
\max
\left(
S_{\mathrm{Executor1}},
S_{\mathrm{Executor2}}
\right)+1
$$

### 情况二：任意一个 Executor 完成即可执行

那么取较快的一条路径：

$$
S_{\mathrm{Expresser}}
=
\min
\left(
S_{\mathrm{Executor1}},
S_{\mathrm{Executor2}}
\right)+1
$$

所以 KVFlow 不是简单地在图上计算普通拓扑距离，而是给每个 Agent 节点配置一个**步骤聚合函数**。

---

## 4.2 Steps-to-execution

KVFlow 为每个 Agent 计算一个 `steps-to-execution`：

> 从当前状态开始，最早还需要经过多少个逻辑步骤，该 Agent 才可能被执行。

例如当前正在运行 Planner：

| Agent | Steps-to-execution |
|---|---:|
| Planner | $0$ |
| Executor | $1$ |
| Expresser | $2$ |
| Reviewer | $3$ |

这个值越小，表示越快会被使用。

因此缓存管理策略是：

- 小值：近期需要，尽量保留；
- 大值：短期不需要，优先淘汰。

这与 Belady 最优缓存算法的思想类似：理想情况下，应淘汰“距离下一次访问最远”的缓存。KVFlow 无法知道精确未来请求，但 Agent 工作流提供了比 LRU 更有价值的未来信息。

---

# 5. 工作流感知的缓存淘汰

## 5.1 为什么不能只给 Agent 分配优先级？

因为 Prefix Cache 是一棵树，而不是“一个 Agent 对应一个独立缓存对象”。

例如：

```text
root
 └── 共同的系统提示词
      ├── 共同工具说明
      │    ├── Executor 1 专属提示词
      │    └── Executor 2 专属提示词
      └── Reviewer 专属提示词
```

一个缓存节点可能被多个 Agent 共享。

如果简单地按 Agent 淘汰：

- 可能错误删除其他 Agent 仍需使用的共享前缀；
- 也可能只能整体删除一个 Agent 的缓存，粒度太粗。

因此 KVFlow 把 Agent 级别的步骤信息映射为**KV Cache 节点级别的优先级**。

---

## 5.2 固定前缀和动态后缀采用不同策略

KVFlow 首先区分：

```text
[固定前缀] + [动态后缀]
```

固定前缀：

- 未来调用大概率还会复用；
- 根据 Agent 的 `steps-to-execution` 决定保留优先级。

动态后缀：

- 与当前问题和中间结果相关；
- 下次调用通常会变化；
- 复用概率低；
- 始终设置为最高淘汰优先级。

因此在显存不足时，KVFlow 的淘汰顺序大致是：

```text
先淘汰动态后缀
      ↓
再淘汰短期内不会执行的 Agent 固定前缀
      ↓
尽量保留即将执行的 Agent 固定前缀
```

这是一个非常重要但容易忽略的设计：**刚刚生成出来的动态上下文虽然“最新”，却不一定值得缓存。** LRU 恰恰容易把它留下。

---

## 5.3 优先级如何传播到 Radix Tree？

假设每个 Agent 固定前缀末尾节点的执行距离分别为：

```text
Executor 1：1
Executor 2：3
Reviewer：4
```

KVFlow 将这个值标在对应固定前缀的末尾节点上，然后沿树向根节点传播。

当某个节点被多个 Agent 共享时，使用：

$$
P(v)=\min_{c\in\operatorname{children}(v)}P(c)
$$

也就是说，共享节点取所有使用者中最小的执行距离。

原因是：

> 只要有一个 Agent 很快要使用这个共享节点，就不应该过早淘汰它。

例如：

```text
共享前缀：被 Executor 1 和 Executor 2 共同使用
Executor 1：1 步后执行
Executor 2：4 步后执行
```

则共享前缀优先级为：

$$
\min(1,4)=1
$$

所以共享节点会按照 Executor 1 的紧迫程度被保护。

---

## 5.4 淘汰规则

在 KVFlow 的定义中：

- `steps-to-execution` 越小，越不应该淘汰；
- `steps-to-execution` 越大，越应该淘汰。

显存紧张时：

1. 首先淘汰动态后缀；
2. 然后按照执行距离从大到小淘汰固定前缀节点；
3. 树上共享节点采用保守的最小值；
4. 多个并发工作流同时引用某个节点时，同样取最保守的优先级。

可以抽象为：

$$
v^*
=
\arg\max_{v\in\mathcal{E}}P(v)
$$

其中：

- $\mathcal{E}$ 是当前可淘汰节点集合；
- $P(v)$ 是节点距离未来使用的优先级；
- 值最大的节点最先淘汰。

---

# 6. KV Cache 提前预取

仅仅更聪明地淘汰还不够，因为 GPU 内存有限，不可能保留所有缓存。

KVFlow 因此把 CPU 内存作为二级缓存：

```text
GPU：近期马上要用的 KV Cache
CPU：已被淘汰、但未来可能重新使用的固定前缀
```

---

## 6.1 传统响应式加载

传统 HiCache 的逻辑是：

```text
请求到达
  ↓
发现缓存位于 CPU
  ↓
CPU → GPU
  ↓
传输完成
  ↓
执行请求
```

总延迟近似为：

$$
T_{\mathrm{total}}
=
T_{\mathrm{load}}
+
T_{\mathrm{compute}}
$$

即使 CPU 加载比重新计算快，加载时间仍然暴露在关键路径上。

---

## 6.2 KVFlow 的主动预取

KVFlow 根据 Agent Step Graph 知道下一步可能执行哪些 Agent。

例如：

```text
Planner 正在执行
下一步：Executor 1
```

那么在 Planner 还没执行完时，KVFlow 就启动后台线程：

```text
CPU 中 Executor 1 的 KV Cache
              ↓
           异步传输
              ↓
GPU 中 Executor 1 的 KV Cache
```

理想情况下：

$$
T_{\mathrm{total}}
\approx
\max
\left(
T_{\mathrm{current\ compute}},
T_{\mathrm{prefetch}}
\right)
$$

而不是两者相加。

如果当前 Agent 计算时间足够长，使得：

$$
T_{\mathrm{current\ compute}}
\geq
T_{\mathrm{prefetch}}
$$

那么预取开销可以完全被隐藏。

---

## 6.3 分支工作流怎么办？

例如：

```text
          ┌→ Executor 1
Planner ──┤
          └→ Executor 2
```

如果运行时还不知道会走哪条分支，KVFlow 会保守地预取所有可能的下一步 Agent：

```text
预取 Executor 1
预取 Executor 2
```

但会设置同时预取数量的上限，避免：

- 无效分支占满 GPU；
- 大量预取占满 PCIe；
- 预取反而挤掉真正有用的缓存。

这一机制提高了命中概率，但也构成了 KVFlow 的潜在局限：分支越多、预测越不确定，投机预取浪费越严重。

---

# 7. 状态感知调度

## 7.1 只预取还不够

假设：

- Planner 的计算时间是 $10$ ms；
- Executor 的 KV Cache 加载需要 $20$ ms。

即使提前预取，Planner 完成时，缓存仍然没加载完：

```text
Planner 结束
    ↓
Executor 缓存仍在加载
    ↓
GPU 等待
```

所以 KVFlow 加入了状态感知调度。

---

## 7.2 KV 节点的四种状态

每个缓存节点带有状态字段：

| 状态 | 含义 |
|---|---|
| `in GPU` | KV Cache 已位于 GPU |
| `backup in CPU` | GPU 中不存在，但 CPU 有备份 |
| `loading` | 正在从 CPU 加载到 GPU |
| `offloading` | 正在从 GPU 写回 CPU |

调度器在选择下一个请求时，会检查其所需的所有 KV 节点。

如果某个请求依赖的节点仍是 `loading`：

- 不重复发起加载；
- 暂时跳过这个请求；
- 先调度其他缓存已经就绪的请求。

例如：

```text
Executor 1：缓存加载中
Executor 2：缓存已就绪
Workflow B：也有就绪请求
```

调度器会先执行 Executor 2 或 Workflow B，而不是让 GPU 等待 Executor 1。

---

## 7.3 为什么能实现“Fully Overlapped”？

完整重叠依赖三层机制：

1. **提前知道下一步是谁**；
2. **异步执行 CPU 到 GPU 传输**；
3. **当预取没完成时，先运行其他就绪任务**。

时间线可以理解为：

```text
GPU：    Planner 计算 | 其他就绪请求 | Executor 1 计算
PCIe：      Executor 1 KV 预取
```

这样，数据搬运尽量藏在其他请求的 GPU 计算后面。

不过论文中的“fully overlapped”更适合理解为一个系统目标，而不是无条件保证。要完全隐藏传输，必须有足够的并行计算或其他就绪请求。如果系统只有一个串行工作流，且当前计算比传输更短，仍可能出现部分等待。

---

# 8. 系统实现

KVFlow 原型基于 **SGLang v0.4.4**。

## 8.1 前端修改

前端需要向后端传递额外元数据：

- 当前工作流的唯一 `client ID`；
- 当前 Agent 的身份；
- Agent Step Graph；
- 各 Agent 的 `steps-to-execution`；
- 固定前缀的结束位置。

论文实现中假设一个 `sgl.function` 对应一个 Agent。

---

## 8.2 为什么需要 Client ID？

不同应用可能都存在名为 `Planner` 的 Agent：

```text
Workflow A：Planner
Workflow B：Planner
```

如果只用 Agent 名称作为标识，后端会混淆缓存归属。

因此 KVFlow 使用：

```text
(client_id, agent_id)
```

共同标识一个 Agent。

---

## 8.3 如何识别固定前缀？

这是实现中一个实际难点。

KVFlow 提供两种方式：

### 方式一：用户显式标记

应用开发者明确告诉系统：

```text
固定部分在第 N 个 token 结束
```

优点：

- 准确；
- 实现简单；
- 不容易误判。

缺点：

- 需要修改 Agent 框架或应用代码；
- 增加开发者负担。

### 方式二：基于命中历史推断

系统观察某 Agent 的多次请求，把持续重复命中的最长前缀视为固定部分。

优点：

- 应用接入更透明。

缺点：

- 需要历史请求；
- 可能在早期误判；
- 如果 Agent Prompt 动态变化，推断结果可能不稳定。

---

## 8.4 后端修改

后端主要扩展了 SGLang 的 Radix Cache：

- 为 KV 节点增加工作流优先级；
- 根据 Step Graph 更新优先级；
- 增加 CPU 到 GPU 的后台预取线程；
- 维护节点状态；
- 修改请求调度器；
- 避免正在 `offloading` 的节点同时被再次淘汰；
- 避免正在 `loading` 的节点被重复加载。

论文认为这一设计也能迁移到 vLLM。vLLM 使用块级自动前缀缓存，因此可以给每个 KV block 维护类似的工作流优先级。

---

# 9. 实验设计与结果

## 9.1 对比系统

论文主要对比三种配置：

| 系统 | GPU Prefix Cache | CPU 二级缓存 | 淘汰策略 | 提前预取 |
|---|---:|---:|---|---:|
| SGLang | 是 | 否 | LRU | 否 |
| SGLang + HiCache | 是 | 是 | LRU | 主要是响应式加载与层级流水 | 否 |
| KVFlow | 是 | 是 | 工作流感知 | 是 |

---

## 9.2 单工作流实验

工作负载：

- 一个顺序执行的 $10$ Agent 工作流；
- Batch size 为 $1$；
- 每个 Agent 输入包含固定前缀和动态后缀；
- 先预热缓存；
- 工作流执行 $10$ 次；
- 每次动态后缀不同。

硬件和模型：

| 平台 | 模型 | GPU | GPU 显存 | PCIe |
|---|---|---|---:|---|
| 平台一 | Llama-3.1-8B | A10G | 24 GB | 约 2 GB/s |
| 平台二 | Qwen2.5-32B | H100 | 80 GB | 约 64 GB/s |

测试配置用：

```text
固定 token / 动态 token / 输出 token
```

表示，例如：

```text
8192 / 32 / 32
```

即：

- 固定前缀 $8192$ token；
- 动态后缀 $32$ token；
- 输出 $32$ token。

---

## 9.3 单工作流结果

在 A10G、`8192/32/32` 配置下，KVFlow：

- 相比 SGLang + HiCache：最高约 $1.83\times$；
- 相比 GPU-only SGLang：最高约 $2.91\times$。

论文还观察到：

- 固定前缀为 $8192$ token 时，平均加速约 $1.48\times$；
- 固定前缀为 $4096$ token 时，平均加速约 $1.28\times$。

原因很直观：

> 固定前缀越长，一次缓存未命中的代价越大，KVFlow 越有价值。

---

## 9.4 输出越长，加速越不明显

KVFlow 主要优化的是：

- prefill；
- 缓存恢复；
- CPU-GPU 数据传输等待。

它并没有直接优化自回归 decode。

总延迟可以粗略写成：

$$
T_{\mathrm{total}}
=
T_{\mathrm{prefill}}
+
T_{\mathrm{decode}}
+
T_{\mathrm{cache\ miss}}
$$

如果输出很长：

$$
T_{\mathrm{decode}}
\gg
T_{\mathrm{prefill}}+T_{\mathrm{cache\ miss}}
$$

那么即使完全消除 Cache Miss，总体加速比例也会下降。

因此 KVFlow 特别适合：

- 长固定提示词；
- 短输出；
- 多次反复调用；
- 频繁 Agent 切换。

而对于长篇生成，收益会被 decode 时间稀释。

---

## 9.5 高并发实验

论文还在单张 H100 上同时运行多个独立工作流：

- 固定前缀长度主要为 $512$ 或 $1024$ token；
- 动态部分为 $256$ token；
- 输出部分为 $256$ token；
- 并发工作流最高达到数十个。

结果：

- 相比 GPU-only SGLang，KVFlow 最高约 $1.25\times$；
- 相比 SGLang HiCache，最高约 $2.19\times$。

值得注意的是，在某些高并发配置下，HiCache 甚至比不使用 CPU Cache 的 SGLang 更慢。

论文给出的解释包括：

1. 频繁缓存未命中触发响应式加载；
2. 加载过程干扰 SGLang 原有调度流水线；
3. KV 数据布局较碎片化，PCIe 带宽利用率有限；
4. 请求在等待 CPU 到 GPU 传输时造成 GPU 空闲。

KVFlow 没有彻底解决内存碎片问题，但通过提前预取和就绪请求调度，更好地隐藏了传输成本。

---

## 9.6 更真实的 PEER 工作流

论文还模拟了 PEER 风格的多 Agent 应用：

- 每个工作流包含 $4$ 个 Agent；
- 使用 Financial QA 数据集；
- Agent Prompt 由角色和指令生成；
- 不同 Agent 的 Prompt 部分重叠，但不完全相同；
- 固定、动态和输出长度更接近真实应用。

结果：

- 相比 SGLang：最高约 $1.12\times$；
- 相比 HiCache：最高约 $1.08\times$。

这个结果比合成的超长前缀实验温和很多，说明：

> KVFlow 的最大收益主要出现在长固定前缀或显存压力较大的场景；普通、较短的现实 Prompt 中仍有收益，但通常不是数量级提升。

---

# 10. 这篇论文真正的创新点

## 10.1 从历史局部性转向未来语义局部性

LRU 使用的是历史信号：

```text
过去多久没访问？
```

KVFlow 使用的是工作流语义：

```text
距离下一次可能执行还有几步？
```

这是一种从“通用缓存策略”转向“应用语义感知系统”的变化。

---

## 10.2 把 Agent 图信息下沉到推理系统

过去多 Agent 框架和 LLM Serving 系统通常是分离的：

```text
Agent 框架：知道执行图，但不知道 GPU Cache
Serving 后端：知道 GPU Cache，但不知道执行图
```

KVFlow 建立了一条跨层通路：

```text
Agent 工作流结构
      ↓
HTTP 请求元数据
      ↓
Serving 调度器与缓存管理器
```

它证明了前端工作流语义可以帮助后端做更好的资源决策。

---

## 10.3 节点级而不是 Agent 级缓存管理

如果只给每个 Agent 一个优先级，就无法正确处理共享前缀。

KVFlow 将优先级传播到 Radix Tree 节点，在共享节点上取最小执行距离，这使得设计能适配：

- Agent 之间共享系统提示词；
- 工具描述共享；
- few-shot 模板共享；
- 多工作流部分前缀共享。

---

## 10.4 淘汰、预取、调度三者联合优化

单独做任何一项都不够：

- 只改淘汰：仍然可能有不可避免的 Cache Miss；
- 只预取：传输没结束时仍会阻塞；
- 只调度：不知道哪些数据应提前加载。

KVFlow 的完整路径是：

```text
预测未来使用
    ↓
尽量避免错误淘汰
    ↓
不可避免淘汰时备份到 CPU
    ↓
请求到来前提前换入
    ↓
换入没完成时调度其他任务
```

---

# 11. 局限性与值得质疑的地方

## 11.1 依赖工作流结构可知

KVFlow 最适合结构明确的工作流：

- DAG；
- 固定流水线；
- 显式分支；
- 可识别的循环。

但完全自主的 Agent 可能根据 LLM 输出临时决定：

- 调哪个工具；
- 选择哪个 Agent；
- 是否重试；
- 是否生成新的子任务。

此时 Step Graph 只能提供近似预测，工作流越动态，优先级越可能失真。

---

## 11.2 分支可能导致过度预取

如果一个节点后面有很多候选 Agent：

```text
Planner → {A, B, C, D, E}
```

为了避免漏掉正确分支，系统可能预取多个缓存。

代价包括：

- 浪费 CPU-GPU 带宽；
- 占用 GPU 显存；
- 挤掉其他真正有用的缓存；
- 增加状态管理复杂度。

虽然论文使用预取并发上限控制问题，但没有从根本上解决预测准确率和预取成本之间的权衡。

---

## 11.3 “完全重叠”依赖足够的并发性

预取开销只有在以下条件满足时才能完全隐藏：

$$
T_{\mathrm{available\ overlap}}
\geq
T_{\mathrm{prefetch}}
$$

可重叠时间可能来自：

- 当前 Agent 的计算；
- 同一工作流的并行 Agent；
- 其他工作流的就绪请求。

如果：

- 只有一个工作流；
- Agent 输出很短；
- PCIe 较慢；
- 没有其他请求可执行；

那么仍然会暴露部分传输延迟。因此“fully overlapped”不是对所有负载都成立的绝对保证。

---

## 11.4 固定前缀识别并不总是容易

现实中的 Agent Prompt 可能包含：

- 当前日期；
- 动态工具列表；
- 临时权限；
- 用户画像；
- 会话摘要；
- 持续变化的 system context。

这会让“固定前缀”边界变得模糊。

显式标记准确但需要应用改造；启发式识别接入简单，但可能误判。

---

## 11.5 主要优化 prefill，不优化 decode

KVFlow 不改变：

- 模型权重；
- 注意力计算；
- 采样逻辑；
- 每 token 解码成本。

所以它与 speculative decoding、量化、稀疏注意力、Prefix/Prompt 压缩等方法是正交的，但如果应用主要耗时来自长输出，单独使用 KVFlow 的收益有限。

---

## 11.6 现实实验收益相对有限

合成的 $8192$ token 固定前缀实验展示了很高的峰值收益，但 PEER 风格真实负载中的增益约为：

- $1.12\times$；
- $1.08\times$。

这并不意味着方法无效，而是说明其收益高度依赖：

- Prompt 长度；
- GPU 显存压力；
- Agent 重复调用频率；
- CPU-GPU 带宽；
- 并发度；
- 动态后缀和输出长度。

因此不能把论文中的 $2.19\times$ 理解成所有 Agent 应用都能获得两倍加速。

---

## 11.7 评价主要集中在延迟

论文认为 KVFlow 不修改模型、Prompt 和解码逻辑，因此输出语义天然不变，实验主要关注系统性能。

不过还可以进一步评估：

- TTFT；
- P50、P95、P99 延迟；
- 吞吐量；
- PCIe 实际带宽利用率；
- CPU 内存开销；
- 无效预取率；
- 预取准确率；
- 不同分支熵下的表现；
- 调度改变是否导致工作流间公平性问题；
- 多租户场景下的优先级隔离。

这些在论文中并不是重点。

---

# 12. 哪些场景最适合 KVFlow？

## 非常适合

- 每个 Agent 有很长的系统提示词或 few-shot 示例；
- Agent 会在循环中反复执行；
- GPU 显存不足以保存全部固定前缀；
- 工作流图比较稳定；
- 有多个工作流并发执行；
- 输出较短、prefill 占比高；
- CPU 内存充足；
- CPU-GPU 传输能够和 GPU 计算重叠。

例如：

- 多 Agent 软件工程；
- 多阶段代码生成与测试；
- Planner–Executor–Reviewer 流水线；
- 复杂 RAG 工作流；
- 带大量工具描述的 Agent；
- 多角色仿真；
- 多轮审核与迭代。

## 收益可能较小

- 固定 Prompt 很短；
- 工作流只执行一次；
- 输出非常长；
- GPU 显存充足、不会淘汰缓存；
- Agent 路由高度随机；
- 并发度很低且 PCIe 很慢；
- Prompt 每次都大幅改变，几乎没有可复用前缀。

---

# 13. 用伪代码理解 KVFlow

## 13.1 计算执行距离

```python
def compute_steps(agent, predecessor_steps, aggregation):
    if aggregation == "all":
        return max(predecessor_steps) + 1

    if aggregation == "any":
        return min(predecessor_steps) + 1

    return custom_aggregation(predecessor_steps)
```

---

## 13.2 将 Agent 优先级映射到缓存树

```python
def assign_cache_priority(agent, steps_to_execution):
    fixed_end_node = agent.fixed_prefix_end_node
    fixed_end_node.priority = steps_to_execution

    node = fixed_end_node.parent

    while node is not None:
        child_priorities = [
            child.priority
            for child in node.children
            if child.priority is not None
        ]

        node.priority = min(child_priorities)
        node = node.parent
```

动态后缀则直接设置成最容易淘汰：

```python
for node in agent.dynamic_suffix_nodes:
    node.priority = HIGHEST_EVICTION_PRIORITY
```

---

## 13.3 淘汰逻辑

```python
def evict_until_enough_memory(required_bytes):
    candidates = get_evictable_nodes()

    candidates.sort(
        key=lambda node: (
            node.is_dynamic_suffix,
            node.priority,
        ),
        reverse=True,
    )

    freed = 0

    for node in candidates:
        if node.status in {"loading", "offloading"}:
            continue

        offload_to_cpu_if_needed(node)
        freed += node.size

        if freed >= required_bytes:
            break
```

---

## 13.4 预取与状态感知调度

```python
def prefetch_next_agents(step_graph):
    next_agents = step_graph.possible_next_agents()

    for agent in next_agents[:MAX_PREFETCH_COUNT]:
        for node in agent.required_fixed_prefix_nodes:
            if node.status == "backup_in_cpu":
                node.status = "loading"
                submit_background_load(node)
```

调度请求：

```python
def schedule(requests):
    for request in requests:
        states = [
            node.status
            for node in request.required_cache_nodes
        ]

        if "loading" in states:
            continue

        if all(state == "in_gpu" for state in states):
            return request

    return None
```

这些伪代码不是论文源码，但体现了论文的核心逻辑。

---

# 14. 与相关技术的关系

| 技术 | 主要解决的问题 | 与 KVFlow 的关系 |
|---|---|---|
| vLLM PagedAttention | KV 内存碎片和分页管理 | 可作为底层存储机制 |
| SGLang RadixAttention | 跨请求共享前缀 | KVFlow 在其上增加未来感知策略 |
| SGLang HiCache | GPU/CPU 分层缓存 | KVFlow改进淘汰和预取时机 |
| Continuous batching | 提高并发吞吐 | 可为 KVFlow 提供更多可重叠计算 |
| Speculative decoding | 减少解码步数 | 与 KVFlow 正交 |
| Prompt compression | 缩短输入长度 | 会降低 KV 压力，也可组合 |
| Autellix、Parrot | Agent 请求调度 | 偏请求级调度，KVFlow 偏前缀缓存管理 |

---

# 15. 总结与评价

KVFlow 并没有提出新的模型结构，也没有改变注意力算法。它的价值在于发现：

> 多 Agent 系统本来就掌握未来执行结构，但传统 LLM Serving 后端完全没有利用这类信息。

它把工作流信息转化为三个具体优化：

1. 用 `steps-to-execution` 代替纯 LRU 信号；
2. 在 Radix Tree 上进行细粒度、共享感知的 KV 节点淘汰；
3. 提前从 CPU 预取即将使用的 KV Cache，并通过状态感知调度隐藏传输延迟。

从系统研究角度看，这篇论文最重要的启示不是某个具体缓存公式，而是：

> **未来的 Agent Serving 系统不应把每次 LLM 请求视为彼此独立的黑盒，而应该利用整个工作流的控制流、数据流和语义信息进行跨层优化。**

其峰值结果很亮眼：

- 单工作流长 Prompt：相比 HiCache 最高 $1.83\times$；
- 高并发：相比 HiCache 最高 $2.19\times$；
- 相比 GPU-only SGLang，特定单工作流配置达到 $2.91\times$。

但在更真实、Prompt 较短的 PEER 工作流中，收益约为 $1.08\times$ 到 $1.12\times$。因此，KVFlow 更像是一项**针对长固定前缀、显存受限和多工作流并发场景的精准系统优化**，而不是对所有 Agent 应用都能带来翻倍加速的通用方案。