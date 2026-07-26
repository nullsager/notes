# 使用 Transformers 在本地部署 Qwen 3-0.6 B

下面先介绍完成作业需要掌握的前置知识，然后给出完整的环境安装、模型下载、推理代码及运行方法。

---

## 一、需要了解的前置知识

### 1. 什么是 Qwen 3-0.6 B

Qwen 3-0.6 B 是通义千问团队发布的因果语言模型：

- 参数量约为 $0.6B$，即约 $6$ 亿参数
- 支持中文、英文等多种语言
- 支持普通对话模式和思考模式
- 上下文长度最高为 $32768$ 个 Token
- 模型规模较小，可以在普通电脑 CPU 上运行
- 使用 NVIDIA 显卡可以显著提高生成速度

这里的“部署”主要指：

1. 将模型文件下载到本地
2. 使用 Transformers 加载本地模型
3. 在本机 CPU 或 GPU 上完成推理
4. 推理过程中不再从网络下载模型

---

### 2. Transformers 是什么

`transformers` 是 Hugging Face 提供的预训练模型库。它为不同模型提供了统一的加载和推理接口。

本作业主要使用两个类：

```python
from transformers import AutoModelForCausalLM, AutoTokenizer
```

- `AutoTokenizer`：将用户输入的文本转换成模型能够处理的 Token ID
- `AutoModelForCausalLM`：加载因果语言模型，根据已有文本逐步预测后续内容

Qwen 3 要求使用较新的 Transformers。官方说明：如果版本低于 `4.51.0`，可能出现下面的错误：

```text
KeyError: 'qwen3'
```

因此应安装：

```text
transformers>=4.51.0
```

---

### 3. Tokenizer 和 Token

语言模型不能直接处理字符串，需要先将文本切分并转换成数字。

例如：

```text
你好，请介绍一下人工智能。
```

经过 Tokenizer 处理后，会转换成类似下面的整数序列：

```text
[1234, 5678, 9012, ...]
```

模型生成的结果同样是 Token ID，需要使用 Tokenizer 解码回文本。

---

### 4. Chat Template

不同对话模型对输入格式有不同要求。Qwen 3 使用带角色的消息格式：

```python
messages = [
    {"role": "system", "content": "你是一个有帮助的AI助手。"},
    {"role": "user", "content": "请介绍一下人工智能。"},
]
```

角色一般包括：

- `system`：规定助手的身份和行为
- `user`：用户输入
- `assistant`：模型回答

`apply_chat_template()` 会将这些消息转换为 Qwen 3 所要求的对话格式。

---

### 5. CPU、GPU 和数据精度

#### CPU 推理

CPU 兼容性最好，一般使用 `float32`：

```python
dtype = torch.float32
```

优点是稳定，不需要 NVIDIA 显卡；缺点是生成速度较慢，内存占用相对较大。

#### GPU 推理

如果安装了支持 CUDA 的 PyTorch，并且电脑具有 NVIDIA 显卡，可以使用：

```python
device = torch.device("cuda")
```

GPU 通常使用 `bfloat16` 或 `float16`，可以减少显存占用并提高速度：

```python
dtype = torch.bfloat16
```

程序可以通过下面的方式判断 CUDA 是否可用：

```python
torch.cuda.is_available()
```

需要注意：电脑有 NVIDIA 显卡，并不代表 Python 一定能使用 GPU，还需要正确安装显卡驱动以及 CUDA 版本的 PyTorch。

---

### 6. 推理中的几个常用参数

| 参数 | 作用 |
|---|---|
| `max_new_tokens` | 最多生成多少个新 Token |
| `do_sample` | 是否使用随机采样 |
| `temperature` | 控制回答的随机程度 |
| `top_p` | 从累计概率较高的候选词中采样 |
| `top_k` | 只从概率最高的若干候选词中采样 |
| `repetition_penalty` | 对重复内容进行惩罚 |

Qwen 3 官方建议：

- 思考模式：`temperature=0.6`、`top_p=0.95`、`top_k=20`
- 非思考模式：`temperature=0.7`、`top_p=0.8`、`top_k=20`

本作业默认关闭思考模式，以获得更快的生成速度。

---

## 二、项目结构

建议创建以下项目：

```text
qwen 3-local/
├── Qwen 3-0.6 B/
├── main. py
└── requirements. txt
```

其中：

- `Qwen 3-0.6 B/`：保存模型文件
- `main. py`：本地推理程序
- `requirements. txt`：Python 依赖

---

## 三、创建 Python 环境

建议使用 Python `3.10` 或更高版本。

### 1. 创建虚拟环境

```bash
python -m venv .venv
```

Windows 激活环境：

```bash
.venv\Scripts\activate
```

Linux 或 macOS 激活环境：

```bash
source .venv/bin/activate
```

---

### 2. 编写依赖文件

创建 `requirements. txt`：

```text
torch
transformers>=4.51.0
huggingface_hub
safetensors
```

然后安装：

```bash
python -m pip install --upgrade pip
pip install -r requirements. txt
```

### NVIDIA GPU 用户注意

直接执行 `pip install torch` 不一定会安装正确的 CUDA 版本。建议根据自己的操作系统和 CUDA 环境，从 PyTorch 官方网站选择对应命令：

<https://pytorch.org/get-started/locally/>

安装后可以执行下面的命令检查：

```bash
python -c "import torch; print ('PyTorch: ', torch.__version__); print ('CUDA 可用: ', torch. cuda. is_available ())"
```

如果输出为：

```text
CUDA 可用: True
```

说明可以使用 NVIDIA GPU 推理。

---

## 四、下载模型到本地

### 1. 安装并检查 `hf` 命令

前面的 `huggingface_hub` 包通常会提供 `hf` 命令。

```bash
hf --help
```

如果提示找不到命令，可以升级：

```bash
pip install -U huggingface_hub
```

---

### 2. 登录 Hugging Face

```bash
hf auth login
```

按照终端提示通过浏览器登录，或者粘贴 Hugging Face Access Token。

Qwen 3-0.6 B 是公开模型，通常不登录也能下载；但登录后下载限制更少，因此建议登录。

---

### 3. 下载完整模型

在项目根目录执行：

```bash
hf download Qwen/Qwen 3-0.6 B --local-dir ./Qwen 3-0.6 B
```

> 注意：参数前面必须使用两个英文半角减号 `--`。题目中的 `–local-dir` 是排版产生的长横线，直接复制可能会报错。

下载完成后，模型目录中应该包含类似文件：

```text
Qwen 3-0.6 B/
├── config. json
├── generation_config. json
├── model. safetensors
├── tokenizer. json
├── tokenizer_config. json
└── ...
```

可以执行下面的命令检查：

Windows：

```bash
dir Qwen 3-0.6 B
```

Linux 或 macOS：

```bash
ls -lh Qwen 3-0.6 B
```

---

## 五、完整推理代码

创建 `main. py`：

```python
from pathlib import Path

import torch
from transformers import AutoModelForCausalLM, AutoTokenizer


# 模型必须已经通过 hf download 下载到该目录
MODEL_PATH = Path (__file__). parent / "Qwen 3-0.6 B"


def select_device () -> tuple[torch. device, torch. dtype]:
    """根据运行环境自动选择 GPU 或 CPU 以及合适的数据精度。"""
    if torch. cuda. is_available ():
        device = torch.device ("cuda")

        # 支持 BF 16 时优先使用 BF 16，否则使用 FP 16
        dtype = (
            torch. bfloat 16
            if torch. cuda. is_bf 16_supported ()
            else torch. float 16
        )
    else:
        device = torch.device ("cpu")
        dtype = torch. float 32

    return device, dtype


def load_model ():
    """从本地目录加载 Tokenizer 和 Qwen 3 模型。"""
    if not MODEL_PATH.exists ():
        raise FileNotFoundError (
            f"没有找到模型目录：{MODEL_PATH}\n"
            "请先执行：hf download Qwen/Qwen 3-0.6 B "
            "--local-dir ./Qwen 3-0.6 B"
        )

    device, dtype = select_device ()

    # local_files_only=True 表示只读取本地文件，不访问网络
    tokenizer = AutoTokenizer. from_pretrained (
        MODEL_PATH,
        local_files_only=True,
    )

    model = AutoModelForCausalLM. from_pretrained (
        MODEL_PATH,
        torch_dtype=dtype,
        local_files_only=True,
    ). to (device)

    # 切换到推理模式，关闭 Dropout 等训练行为
    model.eval ()

    return tokenizer, model, device, dtype


def generate_response (
    prompt: str,
    tokenizer,
    model,
    device: torch. device,
    max_new_tokens: int = 512,
) -> str:
    """根据用户输入生成回答。"""
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

    # 将消息转换成 Qwen 3 所要求的对话格式
    text = tokenizer. apply_chat_template (
        messages,
        tokenize=False,
        add_generation_prompt=True,
        enable_thinking=False,  # 关闭思考模式，提高生成速度
    )

    # 文本编码为 PyTorch 张量，并移动到模型所在设备
    inputs = tokenizer (
        text,
        return_tensors="pt",
    ). to (device)

    with torch. inference_mode ():
        generated_ids = model.generate (
            **inputs,
            max_new_tokens=max_new_tokens,
            do_sample=True,
            temperature=0.7,
            top_p=0.8,
            top_k=20,
            repetition_penalty=1.05,
            pad_token_id=tokenizer. eos_token_id,
        )

    # generate () 的结果包含输入部分，因此只保留新生成的 Token
    input_length = inputs["input_ids"]. shape[1]
    response_ids = generated_ids[0, input_length:]

    return tokenizer.decode (
        response_ids,
        skip_special_tokens=True,
    ). strip ()


def main () -> None:
    """程序入口：加载模型并进行交互式对话。"""
    tokenizer, model, device, dtype = load_model ()

    print ("=" * 60)
    print ("Qwen 3-0.6 B 本地部署成功")
    print (f"模型目录：{MODEL_PATH.resolve ()}")
    print (f"推理设备：{device}")
    print (f"数据精度：{dtype}")
    if device. type == "cuda":
        print (f"显卡型号：{torch. cuda. get_device_name (0)}")
    print ("输入 exit 或 quit 退出程序")
    print ("=" * 60)

    while True:
        prompt = input ("\n 用户："). strip ()

        if prompt.lower () in {"exit", "quit"}:
            print ("程序已退出。")
            break

        if not prompt:
            print ("请输入有效内容。")
            continue

        response = generate_response (
            prompt=prompt,
            tokenizer=tokenizer,
            model=model,
            device=device,
        )

        print (f"Qwen：{response}")


if __name__ == "__main__":
    main ()
```

---

## 六、运行程序

确认当前目录中存在 `main. py` 和 `Qwen 3-0.6 B/`，然后执行：

```bash
python main. py
```

可能得到类似输出：

```text
============================================================
Qwen 3-0.6 B 本地部署成功
模型目录：D:\qwen 3-local\Qwen 3-0.6 B
推理设备：cuda
数据精度：torch. bfloat 16
显卡型号：NVIDIA GeForce RTX 4060
输入 exit 或 quit 退出程序
============================================================

用户：请用三句话介绍一下人工智能。

Qwen：人工智能是让计算机模拟人类感知、推理和决策能力的技术。它广泛应用于自然语言处理、计算机视觉和智能推荐等领域。随着算法和算力的发展，人工智能正在不断改变人们的工作和生活方式。
```

如果没有可用的 NVIDIA GPU，会显示：

```text
推理设备：cpu
数据精度：torch. float 32
```

这属于正常情况，只是生成速度会慢一些。

---

## 七、代码执行流程

整个程序的执行流程可以概括为：

```text
用户输入文本
    ↓
Chat Template 构造对话格式
    ↓
Tokenizer 将文本转换为 Token ID
    ↓
Token ID 被移动到 CPU 或 GPU
    ↓
Qwen 3 自回归生成新的 Token ID
    ↓
Tokenizer 将新 Token 解码为文本
    ↓
输出模型回答
```

代码中设置：

```python
local_files_only=True
```

可以确保 Transformers 只读取 `Qwen 3-0.6 B/` 目录中的文件。下载完成后，即使断开网络，也可以进行本地推理。

---

## 八、可选：启用思考模式

如果需要让 Qwen 3 对数学、编程或逻辑问题进行更深入的推理，可以修改：

```python
enable_thinking=False
```

为：

```python
enable_thinking=True
```

同时将生成参数改为官方推荐值：

```python
generated_ids = model.generate (
    **inputs,
    max_new_tokens=2048,
    do_sample=True,
    temperature=0.6,
    top_p=0.95,
    top_k=20,
    repetition_penalty=1.05,
    pad_token_id=tokenizer. eos_token_id,
)
```

思考模式会生成更多 Token，因此运行时间更长。对于本次部署作业，默认关闭思考模式即可。

---

## 九、常见问题

<details>
<summary><strong>1. 出现 KeyError: 'qwen 3'</strong></summary>

原因是 Transformers 版本过低。

执行：

```bash
pip install -U "transformers>=4.51.0"
```

然后检查版本：

```bash
python -c "import transformers; print (transformers.__version__)"
```

</details>

<details>
<summary><strong>2. 提示 hf 不是内部或外部命令</strong></summary>

重新安装或升级 Hugging Face Hub：

```bash
pip install -U huggingface_hub
```

也可以使用模块方式下载：

```bash
python -m huggingface_hub. commands. huggingface_cli --help
```

如果仍然无法识别，关闭终端并重新打开，确认虚拟环境已经激活。

</details>

<details>
<summary><strong>3. 明明有 NVIDIA 显卡，却使用 CPU</strong></summary>

执行：

```bash
python -c "import torch; print (torch. cuda. is_available ()); print (torch. version. cuda)"
```

如果输出为：

```text
False
None
```

通常表示安装了 CPU 版本的 PyTorch。需要根据 PyTorch 官网提供的命令重新安装 CUDA 版本。

</details>

<details>
<summary><strong>4. 出现 CUDA out of memory</strong></summary>

可以先关闭占用显存的其他程序，然后重新运行。

也可以直接强制使用 CPU，把 `select_device ()` 简化为：

```python
def select_device () -> tuple[torch. device, torch. dtype]:
    return torch.device ("cpu"), torch. float 32
```

</details>

<details>
<summary><strong>5. 下载模型时网络超时</strong></summary>

Linux 或 macOS 可以适当增加超时时间：

```bash
export HF_HUB_DOWNLOAD_TIMEOUT=60
hf download Qwen/Qwen 3-0.6 B --local-dir ./Qwen 3-0.6 B
```

Windows PowerShell：

```powershell
$env: HF_HUB_DOWNLOAD_TIMEOUT = "60"
hf download Qwen/Qwen 3-0.6 B --local-dir ./Qwen 3-0.6 B
```

`hf download` 支持断点续传，失败后通常可以直接重新执行原命令。

</details>

---

## 十、作业总结

本次作业完成了以下内容：

1. 使用 `hf` 命令将 Qwen 3-0.6 B 下载到本地目录
2. 使用 Transformers 加载本地 Tokenizer 和语言模型
3. 使用 `local_files_only=True` 保证模型从本地加载
4. 自动检测 NVIDIA CUDA 环境
5. 有可用显卡时使用 GPU 推理
6. 没有可用显卡时自动使用 CPU 推理
7. 使用 Qwen 3 Chat Template 构造对话输入
8. 实现了简洁的交互式命令行对话程序

参考资料：

- [Qwen3-0.6B 模型页面](https://huggingface.co/Qwen/Qwen3-0.6B)
- [Hugging Face CLI 文档](https://huggingface.co/docs/huggingface_hub/guides/cli)
- [PyTorch 安装页面](https://pytorch.org/get-started/locally/)