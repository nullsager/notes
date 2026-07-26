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