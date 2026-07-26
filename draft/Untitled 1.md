可以，使用 `uv` 会更简洁，不需要手动创建和激活虚拟环境。下面把作业中的环境配置和运行命令全部改成 `uv` 版本；Python 推理代码基本不需要变化。

# 使用 `uv` 部署 Qwen3-0.6B

## 一、安装 `uv`

如果已经安装，可以跳过。

### Windows PowerShell

```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

### Linux 或 macOS

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

检查是否安装成功：

```bash
uv --version
```

---

## 二、创建项目

```bash
uv init qwen3-local
cd qwen3-local
```

建议使用 Python 3.11：

```bash
uv python install 3.11
uv python pin 3.11
```

执行后，项目结构大致如下：

```text
qwen3-local/
├── .python-version
├── main.py
├── pyproject.toml
└── README.md
```

`uv` 会在第一次安装依赖或运行程序时自动创建 `.venv`，通常不需要手动激活。

---

## 三、安装依赖

### 方案一：CPU 推理

如果没有 NVIDIA 显卡，直接执行：

```bash
uv add torch "transformers>=4.51.0" huggingface-hub safetensors
```

### 方案二：NVIDIA GPU 推理

GPU 版本的 PyTorch 与操作系统、显卡驱动和 CUDA 版本有关，建议先打开 PyTorch 官方安装页面：

<https://pytorch.org/get-started/locally/>

选择：

- Package：`Pip`
- Language：`Python`
- Compute Platform：选择推荐的 CUDA 版本

然后使用对应的 PyTorch Wheel 索引安装。例如，假设官方推荐的索引是 CUDA 12.8：

```bash
uv add torch --index https://download.pytorch.org/whl/cu128
uv add "transformers>=4.51.0" huggingface-hub safetensors
```

> CUDA 索引中的版本号可能随时间变化，应以 PyTorch 官方页面给出的地址为准，不要盲目照抄 `cu128`。

检查 PyTorch 是否可以使用 GPU：

```bash
uv run python -c "import torch; print('PyTorch版本:', torch.__version__); print('CUDA版本:', torch.version.cuda); print('CUDA可用:', torch.cuda.is_available())"
```

正常情况下应看到：

```text
CUDA可用: True
```

如果输出 `False`，程序仍然可以运行，但会自动使用 CPU 推理。

---

## 四、使用 `hf` 下载模型

通过 `uv run` 执行项目环境中的 `hf` 命令：

```bash
uv run hf auth login
```

然后下载模型：

```bash
uv run hf download Qwen/Qwen3-0.6B --local-dir ./Qwen3-0.6B
```

注意，`--local-dir` 前面是两个英文半角减号，不能写成排版用的长横线 `–`。

模型下载完成后，项目结构如下：

```text
qwen3-local/
├── .python-version
├── .venv/
├── Qwen3-0.6B/
│   ├── config.json
│   ├── generation_config.json
│   ├── model.safetensors
│   ├── tokenizer.json
│   └── tokenizer_config.json
├── main.py
├── pyproject.toml
└── uv.lock
```

Qwen3-0.6B 是公开模型，一般也可以不登录直接下载：

```bash
uv run hf download Qwen/Qwen3-0.6B --local-dir ./Qwen3-0.6B
```

---

## 五、完整代码

将项目中的 `main.py` 修改为：

```python
from pathlib import Path

import torch
from transformers import AutoModelForCausalLM, AutoTokenizer


# 模型已经通过 hf download 下载到项目目录
MODEL_PATH = Path(__file__).parent / "Qwen3-0.6B"


def select_device() -> tuple[torch.device, torch.dtype]:
    """自动选择 GPU 或 CPU，并选择合适的数据精度。"""
    if torch.cuda.is_available():
        device = torch.device("cuda")

        # 新显卡优先使用 BF16，否则使用 FP16
        dtype = (
            torch.bfloat16
            if torch.cuda.is_bf16_supported()
            else torch.float16
        )
    else:
        device = torch.device("cpu")
        dtype = torch.float32

    return device, dtype


def load_model():
    """从本地目录加载 Qwen3 模型和分词器。"""
    if not MODEL_PATH.is_dir():
        raise FileNotFoundError(
            f"未找到模型目录：{MODEL_PATH}\n"
            "请先执行：\n"
            "uv run hf download Qwen/Qwen3-0.6B "
            "--local-dir ./Qwen3-0.6B"
        )

    device, dtype = select_device()

    # local_files_only=True：只使用本地文件，不访问网络
    tokenizer = AutoTokenizer.from_pretrained(
        MODEL_PATH,
        local_files_only=True,
    )

    model = AutoModelForCausalLM.from_pretrained(
        MODEL_PATH,
        dtype=dtype,
        local_files_only=True,
    ).to(device)

    # 切换到推理模式，关闭训练阶段才需要的功能
    model.eval()

    return tokenizer, model, device, dtype


def generate_response(
    prompt: str,
    tokenizer,
    model,
    device: torch.device,
    max_new_tokens: int = 512,
) -> str:
    """根据用户输入生成模型回答。"""
    messages = [
        {
            "role": "system",
            "content": "你是一个友好、准确、简洁的人工智能助手。",
        },
        {
            "role": "user",
            "content": prompt,
        },
    ]

    # 按照 Qwen3 的对话模板组织输入
    text = tokenizer.apply_chat_template(
        messages,
        tokenize=False,
        add_generation_prompt=True,
        enable_thinking=False,  # 关闭思考模式，提高生成速度
    )

    # 将文本转换成模型可以处理的 Token，并移动到推理设备
    inputs = tokenizer(
        text,
        return_tensors="pt",
    ).to(device)

    # inference_mode 会关闭梯度计算，降低内存占用
    with torch.inference_mode():
        generated_ids = model.generate(
            **inputs,
            max_new_tokens=max_new_tokens,
            do_sample=True,
            temperature=0.7,
            top_p=0.8,
            top_k=20,
            repetition_penalty=1.05,
            pad_token_id=tokenizer.eos_token_id,
        )

    # 输出中包含原始输入，只截取模型新生成的部分
    input_length = inputs["input_ids"].shape[1]
    response_ids = generated_ids[0, input_length:]

    return tokenizer.decode(
        response_ids,
        skip_special_tokens=True,
    ).strip()


def main() -> None:
    """加载模型并启动命令行对话。"""
    tokenizer, model, device, dtype = load_model()

    print("=" * 60)
    print("Qwen3-0.6B 本地部署成功")
    print(f"模型位置：{MODEL_PATH.resolve()}")
    print(f"推理设备：{device}")
    print(f"数据精度：{dtype}")

    if device.type == "cuda":
        print(f"显卡型号：{torch.cuda.get_device_name(0)}")

    print("输入 exit 或 quit 退出程序")
    print("=" * 60)

    while True:
        prompt = input("\n用户：").strip()

        if prompt.lower() in {"exit", "quit"}:
            print("程序已退出。")
            break

        if not prompt:
            print("请输入有效内容。")
            continue

        response = generate_response(
            prompt=prompt,
            tokenizer=tokenizer,
            model=model,
            device=device,
        )

        print(f"Qwen：{response}")


if __name__ == "__main__":
    main()
```

---

## 六、运行程序

不需要手动激活 `.venv`，直接执行：

```bash
uv run main.py
```

也可以写成：

```bash
uv run python main.py
```

示例输出：

```text
============================================================
Qwen3-0.6B 本地部署成功
模型位置：D:\qwen3-local\Qwen3-0.6B
推理设备：cuda
数据精度：torch.bfloat16
显卡型号：NVIDIA GeForce RTX 4060
输入 exit 或 quit 退出程序
============================================================

用户：简单介绍一下人工智能。

Qwen：人工智能是让计算机模拟人类感知、学习、推理和决策能力的技术。它被广泛应用于自然语言处理、计算机视觉、智能推荐和自动驾驶等领域。
```

退出程序时输入：

```text
exit
```

---

## 七、最终需要提交的文件

通常不建议提交体积较大的模型文件和虚拟环境。可以提交：

```text
qwen3-local/
├── main.py
├── pyproject.toml
├── uv.lock
├── .python-version
└── README.md
```

建议在 `.gitignore` 中加入：

```gitignore
.venv/
Qwen3-0.6B/
__pycache__/
*.pyc
```

模型可以由老师通过下面的命令重新下载：

```bash
uv sync
uv run hf download Qwen/Qwen3-0.6B --local-dir ./Qwen3-0.6B
uv run main.py
```

其中：

- `pyproject.toml` 记录项目依赖
- `uv.lock` 锁定依赖的具体版本
- `uv sync` 根据锁文件恢复运行环境
- `Qwen3-0.6B/` 不提交，但需要在本地保留才能运行

---

## 八、常用 `uv` 命令对照

| 原来的操作 | 使用 `uv` |
|---|---|
| 创建虚拟环境 | `uv` 自动创建 |
| `pip install 包名` | `uv add 包名` |
| `pip install -r requirements.txt` | `uv sync` |
| `python main.py` | `uv run main.py` |
| `hf auth login` | `uv run hf auth login` |
| `hf download ...` | `uv run hf download ...` |
| 查看依赖 | `uv tree` |
| 更新锁文件 | `uv lock` |

最简完整执行流程如下：

```bash
uv init qwen3-local
cd qwen3-local

uv python pin 3.11
uv add torch "transformers>=4.51.0" huggingface-hub safetensors

uv run hf auth login
uv run hf download Qwen/Qwen3-0.6B --local-dir ./Qwen3-0.6B

uv run main.py
```








# 温度采样与 Top-p 采样接入 Qwen3-0.6B

这次作业不直接使用 `model.generate()` 内置的采样功能，而是：

1. 调用 Qwen3 得到 Logits
2. 手动实现温度缩放
3. 手动实现 Top-p 筛选
4. 使用 `torch.multinomial()` 抽取下一个 Token
5. 循环完成自回归文本生成

这样可以清楚展示两种采样算法的实现原理。

---

## 一、前置知识

### 1. 什么是 Logits

模型每生成一个 Token，都会输出一个长度等于词表大小的向量：

```python
outputs.logits
```

其形状通常为：

```text
[batch_size, sequence_length, vocabulary_size]
```

我们只需要最后一个位置的 Logits：

```python
next_token_logits = outputs.logits[:, -1, :]
```

Logits 不是概率，需要经过 Softmax 转换：

$$
p_i=\frac{\exp(z_i)}{\sum_j\exp(z_j)}
$$

其中：

- $z_i$ 是第 $i$ 个 Token 的 Logit
- $p_i$ 是第 $i$ 个 Token 的概率

---

### 2. 温度采样

温度采样会在 Softmax 前将 Logits 除以温度 $T$：

$$
p_i=\frac{\exp(z_i/T)}{\sum_j\exp(z_j/T)}
$$

温度的效果如下：

| 温度 | 效果 |
|---|---|
| $T<1$ | 概率分布更加集中，回答更加稳定 |
| $T=1$ | 保持原始概率分布 |
| $T>1$ | 概率分布更加平坦，回答更加随机 |
| $T\to 0$ | 趋近于每次选择概率最大的 Token |

常用值一般为 $0.6$ 到 $1.0$。

---

### 3. Top-p 采样

Top-p 也称为核采样或 Nucleus Sampling。

其步骤是：

1. 将 Token 按概率从高到低排序
2. 从最高概率开始累加
3. 保留累计概率首次达到 $p$ 的最小 Token 集合
4. 将其余 Token 的概率设为 $0$
5. 在保留下来的 Token 中随机采样

例如，排序后的概率为：

```text
Token A：0.40
Token B：0.30
Token C：0.15
Token D：0.10
Token E：0.05
```

当 `top_p=0.8` 时：

```text
A + B + C = 0.85
```

因此保留 A、B、C，过滤 D、E。

---

### 4. 两种采样如何结合

实际生成时通常按照以下顺序处理：

```text
原始 Logits
    ↓
除以 Temperature
    ↓
Softmax 得到概率
    ↓
按照 Top-p 筛选候选 Token
    ↓
重新归一化概率
    ↓
随机抽取下一个 Token
```

---

## 二、完整项目结构

继续使用上一份作业的项目：

```text
qwen3-local/
├── Qwen3-0.6B/
├── main.py
├── pyproject.toml
└── uv.lock
```

如果依赖还没有安装，可以执行：

```bash
uv add torch "transformers>=4.51.0" huggingface-hub safetensors
```

下载模型：

```bash
uv run hf download Qwen/Qwen3-0.6B --local-dir ./Qwen3-0.6B
```

---

## 三、完整代码

将 `main.py` 替换为以下内容：

```python
import argparse
from pathlib import Path

import torch
from transformers import AutoModelForCausalLM, AutoTokenizer


MODEL_PATH = Path(__file__).parent / "Qwen3-0.6B"


def select_device() -> tuple[torch.device, torch.dtype]:
    """自动选择 GPU 或 CPU，并选择合适的数据精度。"""
    if torch.cuda.is_available():
        device = torch.device("cuda")
        dtype = (
            torch.bfloat16
            if torch.cuda.is_bf16_supported()
            else torch.float16
        )
    else:
        device = torch.device("cpu")
        dtype = torch.float32

    return device, dtype


def load_model():
    """从本地目录加载 Qwen3 模型和分词器。"""
    if not MODEL_PATH.is_dir():
        raise FileNotFoundError(
            f"没有找到模型目录：{MODEL_PATH}\n"
            "请先执行：\n"
            "uv run hf download Qwen/Qwen3-0.6B "
            "--local-dir ./Qwen3-0.6B"
        )

    device, dtype = select_device()

    tokenizer = AutoTokenizer.from_pretrained(
        MODEL_PATH,
        local_files_only=True,
    )

    model = AutoModelForCausalLM.from_pretrained(
        MODEL_PATH,
        torch_dtype=dtype,
        local_files_only=True,
    ).to(device)

    model.eval()

    return tokenizer, model, device, dtype


def apply_temperature(
    logits: torch.Tensor,
    temperature: float,
) -> torch.Tensor:
    """
    对 Logits 进行温度缩放。

    temperature 越低，概率分布越集中；
    temperature 越高，概率分布越平坦。
    """
    if temperature <= 0:
        raise ValueError("temperature 必须大于 0")

    return logits / temperature


def apply_top_p(
    logits: torch.Tensor,
    top_p: float,
) -> torch.Tensor:
    """
    对 Logits 进行 Top-p 筛选。

    保留累计概率首次达到 top_p 的最小 Token 集合，
    其余 Token 的 Logit 被设置为负无穷。
    """
    if not 0 < top_p <= 1:
        raise ValueError("top_p 必须满足 0 < top_p <= 1")

    # top_p=1 时保留全部 Token，不需要筛选
    if top_p == 1:
        return logits

    # 按照 Logit 从高到低排序
    sorted_logits, sorted_indices = torch.sort(
        logits,
        descending=True,
        dim=-1,
    )

    # 将排序后的 Logits 转换成概率并计算累计概率
    sorted_probs = torch.softmax(sorted_logits, dim=-1)
    cumulative_probs = torch.cumsum(sorted_probs, dim=-1)

    # 累计概率超过 top_p 的 Token 原本需要删除
    sorted_remove_mask = cumulative_probs > top_p

    # 将掩码向右移动一位，保留首次使累计概率达到 top_p 的 Token
    sorted_remove_mask[..., 1:] = sorted_remove_mask[
        ..., :-1
    ].clone()
    sorted_remove_mask[..., 0] = False

    # 将排序后的删除掩码恢复到原词表顺序
    remove_mask = torch.zeros_like(
        sorted_remove_mask
    ).scatter(
        dim=-1,
        index=sorted_indices,
        src=sorted_remove_mask,
    )

    # 被删除 Token 的 Logit 设为负无穷
    return logits.masked_fill(remove_mask, float("-inf"))


def sample_next_token(
    logits: torch.Tensor,
    temperature: float,
    top_p: float,
) -> torch.Tensor:
    """使用温度采样和 Top-p 采样抽取下一个 Token。"""
    # 第一步：温度缩放
    scaled_logits = apply_temperature(
        logits=logits,
        temperature=temperature,
    )

    # 第二步：Top-p 筛选
    filtered_logits = apply_top_p(
        logits=scaled_logits,
        top_p=top_p,
    )

    # 第三步：将筛选后的 Logits 转换成概率
    probabilities = torch.softmax(
        filtered_logits,
        dim=-1,
    )

    # 第四步：根据概率分布随机抽取一个 Token
    return torch.multinomial(
        probabilities,
        num_samples=1,
    )


def get_eos_token_ids(model, tokenizer) -> set[int]:
    """取得模型的所有结束标记 Token ID。"""
    eos_token_ids = model.generation_config.eos_token_id

    if eos_token_ids is None:
        eos_token_ids = tokenizer.eos_token_id

    if eos_token_ids is None:
        return set()

    if isinstance(eos_token_ids, int):
        return {eos_token_ids}

    return set(eos_token_ids)


def generate_response(
    prompt: str,
    tokenizer,
    model,
    device: torch.device,
    temperature: float = 0.7,
    top_p: float = 0.8,
    max_new_tokens: int = 512,
) -> str:
    """
    手动执行自回归生成。

    每次调用模型得到下一个位置的 Logits，
    再使用温度采样和 Top-p 采样选择 Token。
    """
    messages = [
        {
            "role": "system",
            "content": "你是一个友好、准确、简洁的人工智能助手。",
        },
        {
            "role": "user",
            "content": prompt,
        },
    ]

    # 使用 Qwen3 对话模板构造模型输入
    text = tokenizer.apply_chat_template(
        messages,
        tokenize=False,
        add_generation_prompt=True,
        enable_thinking=False,
    )

    inputs = tokenizer(
        text,
        return_tensors="pt",
    ).to(device)

    current_input_ids = inputs["input_ids"]
    attention_mask = inputs["attention_mask"]

    generated_tokens = []
    past_key_values = None
    eos_token_ids = get_eos_token_ids(model, tokenizer)

    with torch.inference_mode():
        for _ in range(max_new_tokens):
            # 第一次输入完整提示词；
            # 后续只输入最新生成的 Token，并复用 KV Cache
            outputs = model(
                input_ids=current_input_ids,
                attention_mask=attention_mask,
                past_key_values=past_key_values,
                use_cache=True,
                return_dict=True,
            )

            # 保存 KV Cache，避免每一步重复计算前面的 Token
            past_key_values = outputs.past_key_values

            # 取序列最后一个位置的 Logits
            next_token_logits = outputs.logits[:, -1, :]

            # 手动执行温度采样和 Top-p 采样
            next_token_id = sample_next_token(
                logits=next_token_logits,
                temperature=temperature,
                top_p=top_p,
            )

            generated_tokens.append(next_token_id)

            # 遇到结束标记时停止生成
            if next_token_id.item() in eos_token_ids:
                break

            # 下一轮只需要输入本轮新生成的 Token
            current_input_ids = next_token_id

            # Attention Mask 长度也要同步增加
            attention_mask = torch.cat(
                [
                    attention_mask,
                    torch.ones_like(next_token_id),
                ],
                dim=-1,
            )

    if not generated_tokens:
        return ""

    generated_ids = torch.cat(
        generated_tokens,
        dim=-1,
    )

    return tokenizer.decode(
        generated_ids[0],
        skip_special_tokens=True,
    ).strip()


def parse_args() -> argparse.Namespace:
    """读取命令行参数。"""
    parser = argparse.ArgumentParser(
        description="使用温度采样和 Top-p 采样运行 Qwen3-0.6B"
    )

    parser.add_argument(
        "--temperature",
        type=float,
        default=0.7,
        help="采样温度，必须大于 0，默认值为 0.7",
    )
    parser.add_argument(
        "--top-p",
        type=float,
        default=0.8,
        help="Top-p 阈值，范围为 0 到 1，默认值为 0.8",
    )
    parser.add_argument(
        "--max-new-tokens",
        type=int,
        default=512,
        help="最多生成的 Token 数，默认值为 512",
    )
    parser.add_argument(
        "--seed",
        type=int,
        default=42,
        help="随机种子，默认值为 42",
    )

    args = parser.parse_args()

    if args.temperature <= 0:
        parser.error("--temperature 必须大于 0")

    if not 0 < args.top_p <= 1:
        parser.error("--top-p 必须满足 0 < top-p <= 1")

    if args.max_new_tokens <= 0:
        parser.error("--max-new-tokens 必须大于 0")

    return args


def main() -> None:
    """程序入口。"""
    args = parse_args()

    # 设置随机种子，使实验结果尽量可复现
    torch.manual_seed(args.seed)

    if torch.cuda.is_available():
        torch.cuda.manual_seed_all(args.seed)

    tokenizer, model, device, dtype = load_model()

    print("=" * 60)
    print("Qwen3-0.6B 加载成功")
    print(f"模型位置：{MODEL_PATH.resolve()}")
    print(f"推理设备：{device}")
    print(f"数据精度：{dtype}")
    print(f"Temperature：{args.temperature}")
    print(f"Top-p：{args.top_p}")
    print(f"最大生成长度：{args.max_new_tokens}")

    if device.type == "cuda":
        print(f"显卡型号：{torch.cuda.get_device_name(0)}")

    print("输入 exit 或 quit 退出程序")
    print("=" * 60)

    while True:
        prompt = input("\n用户：").strip()

        if prompt.lower() in {"exit", "quit"}:
            print("程序已退出。")
            break

        if not prompt:
            print("请输入有效内容。")
            continue

        response = generate_response(
            prompt=prompt,
            tokenizer=tokenizer,
            model=model,
            device=device,
            temperature=args.temperature,
            top_p=args.top_p,
            max_new_tokens=args.max_new_tokens,
        )

        print(f"Qwen：{response}")


if __name__ == "__main__":
    main()
```

---

## 四、运行程序

使用默认参数运行：

```bash
uv run main.py
```

默认参数为：

```text
temperature = 0.7
top_p = 0.8
max_new_tokens = 512
```

也可以通过命令行设置：

```bash
uv run main.py --temperature 0.7 --top-p 0.8
```

CPU 推理时，建议适当降低最大生成长度：

```bash
uv run main.py --temperature 0.7 --top-p 0.8 --max-new-tokens 128
```

示例输出：

```text
============================================================
Qwen3-0.6B 加载成功
模型位置：D:\qwen3-local\Qwen3-0.6B
推理设备：cuda
数据精度：torch.bfloat16
Temperature：0.7
Top-p：0.8
最大生成长度：512
显卡型号：NVIDIA GeForce RTX 4060
输入 exit 或 quit 退出程序
============================================================

用户：请简单介绍一下机器学习。

Qwen：机器学习是人工智能的一个重要分支，它让计算机从数据中自动学习规律，并利用这些规律完成分类、预测和决策等任务。
```

---

## 五、核心代码分析

### 1. Transformers 输出 Logits

模型调用代码为：

```python
outputs = model(
    input_ids=current_input_ids,
    attention_mask=attention_mask,
    past_key_values=past_key_values,
    use_cache=True,
)
```

读取最后一个位置的 Logits：

```python
next_token_logits = outputs.logits[:, -1, :]
```

例如，假设词表大小为 $V$，那么它的形状是：

```text
[1, V]
```

这组 Logits 表示每一个 Token 作为下一个 Token 的相对可能性。

---

### 2. 温度采样代码

核心代码为：

```python
scaled_logits = logits / temperature
```

对应公式：

$$
z_i'=\frac{z_i}{T}
$$

随后再通过 Softmax 得到概率。

---

### 3. Top-p 采样代码

首先排序：

```python
sorted_logits, sorted_indices = torch.sort(
    logits,
    descending=True,
    dim=-1,
)
```

然后计算概率和累计概率：

```python
sorted_probs = torch.softmax(sorted_logits, dim=-1)
cumulative_probs = torch.cumsum(sorted_probs, dim=-1)
```

找出累计概率超出阈值的 Token：

```python
sorted_remove_mask = cumulative_probs > top_p
```

为了保留首次使累计概率超过 `top_p` 的 Token，需要将删除掩码向右移动一位：

```python
sorted_remove_mask[..., 1:] = sorted_remove_mask[
    ..., :-1
].clone()
sorted_remove_mask[..., 0] = False
```

最后将不参与采样的 Token 设置为负无穷：

```python
logits.masked_fill(remove_mask, float("-inf"))
```

经过 Softmax 后：

$$
\exp(-\infty)=0
$$

所以这些 Token 的概率会变成 $0$。

---

### 4. 随机抽取 Token

使用：

```python
next_token_id = torch.multinomial(
    probabilities,
    num_samples=1,
)
```

`torch.multinomial()` 会根据每个 Token 的概率随机抽取一个 Token，而不是直接选择概率最大的 Token。

---

### 5. 自回归生成

手动生成流程为：

```text
输入提示词
    ↓
模型输出最后一个位置的 Logits
    ↓
温度缩放
    ↓
Top-p 筛选
    ↓
随机抽取下一个 Token
    ↓
将新 Token 重新交给模型
    ↓
重复以上步骤，直到遇到结束标记
```

代码中没有调用：

```python
model.generate()
```

而是直接读取：

```python
outputs.logits
```

因此温度采样和 Top-p 采样均由我们手动完成，符合本次作业要求。

---

## 六、参数对比实验

为了观察两种采样参数的影响，可以使用同一个问题进行多组实验。

例如统一输入：

```text
请写一个关于人工智能的短故事。
```

### 实验一：低温度

```bash
uv run main.py --temperature 0.2 --top-p 0.8
```

特点：

- 回答稳定
- 更倾向选择高概率 Token
- 多次生成的结果比较相似
- 创造性相对较弱

### 实验二：正常温度

```bash
uv run main.py --temperature 0.7 --top-p 0.8
```

特点：

- 稳定性和随机性较平衡
- 适合普通对话
- 是本作业推荐参数

### 实验三：高温度

```bash
uv run main.py --temperature 1.5 --top-p 0.9
```

特点：

- 回答更加随机
- 创造性更强
- 可能出现不连贯或不准确内容

### 实验四：较小的 Top-p

```bash
uv run main.py --temperature 0.7 --top-p 0.3
```

特点：

- 候选 Token 数量较少
- 输出更加保守
- 回答通常更加稳定

### 实验五：不进行 Top-p 截断

```bash
uv run main.py --temperature 0.7 --top-p 1.0
```

当 `top_p=1.0` 时，全部 Token 都会参与采样，相当于只进行温度采样。

---

## 七、实验记录表

作业报告中可以加入以下表格：

| 实验 | Temperature | Top-p | 预期效果 |
|---|---:|---:|---|
| 1 | $0.2$ | $0.8$ | 输出稳定、随机性较低 |
| 2 | $0.7$ | $0.8$ | 稳定性和多样性平衡 |
| 3 | $1.5$ | $0.9$ | 随机性强、创造性较高 |
| 4 | $0.7$ | $0.3$ | 候选范围小、输出保守 |
| 5 | $0.7$ | $1.0$ | 不进行 Top-p 截断 |

因为随机采样具有不确定性，即使提示词和参数相同，也可能生成不同结果。代码设置了随机种子：

```python
torch.manual_seed(args.seed)
```

也可以修改随机种子进行对比：

```bash
uv run main.py --seed 100
```

---

## 八、与 `model.generate()` 的区别

Transformers 本身可以直接完成相同操作：

```python
generated_ids = model.generate(
    **inputs,
    do_sample=True,
    temperature=0.7,
    top_p=0.8,
)
```

但是这会隐藏采样过程。

本次作业手动实现了以下逻辑：

```python
next_token_logits = outputs.logits[:, -1, :]
scaled_logits = next_token_logits / temperature
filtered_logits = apply_top_p(scaled_logits, top_p)
probabilities = torch.softmax(filtered_logits, dim=-1)
next_token_id = torch.multinomial(probabilities, num_samples=1)
```

因此可以直接观察并理解模型如何从 Logits 得到最终生成的 Token。

---

## 九、作业总结

本次作业完成了：

1. 使用 Transformers 获取 Qwen3-0.6B 输出的 Logits
2. 手动实现温度缩放
3. 手动实现 Top-p 候选 Token 筛选
4. 使用 `torch.multinomial()` 按照概率抽取 Token
5. 手动实现自回归文本生成循环
6. 使用 KV Cache 提高生成效率
7. 支持 CPU 和 NVIDIA GPU 自动切换
8. 支持通过命令行修改温度、Top-p、最大生成长度和随机种子

最终的采样流程为：

```text
Qwen3 输出 Logits
    ↓
Temperature 缩放
    ↓
Top-p 筛选
    ↓
Softmax 概率归一化
    ↓
Multinomial 随机抽样
    ↓
得到下一个 Token
```