初始化环境：

```shell
uv init
```

运行：

```
uv run main.py
```

neovim+pyright 识别环境：

打开项目根目录下的 `pyproject.toml`，在文件末尾（或任意位置）添加以下内容：

```toml
[tool.pyright]
venvPath = "."
venv = ". venv"
```

操作, 命令, 备注
装包, uv add numpy, 自动管理配置文件，推荐
删包, uv remove numpy, 自动清理配置文件
看树状图, uv tree, 最清晰，能看到依赖关系
看列表, uv pip list, 传统的扁平列表
看详情, uv pip show numpy, 查看版本、路径、描述


| 操作         | 命令                              |
| ---------- | ------------------------------- |
| 装包         | uv add numpy                    |
| 删包         | uv remove numpy                 |
| 看树状图（依赖关系） | uv tree                         |
| 看列表        | uv pip list                     |
| 看某一个包的详情   | uv pip show numpy               |
| 更新单个包      | uv sync --upgrade-package numpy |
| 更新所有包      | uv sync --upgrade               |

ruff 配置：

```toml
[tool.ruff]
# 1. 目标 Python 版本 (影响语法的自动升级)
target-version = "py311"

# 2. 指定行长 (Black 默认 88，也有团队喜欢 100 或 120)
line-length = 88

# 3. 排除的文件
exclude = [
    ".venv",
    "migrations",
]

[tool.ruff.lint]
# 4. 启用的规则集 (这是最关键的部分)
# E: pycodestyle (错误)
# F: Pyflakes (逻辑错误)
# I: isort (导入排序)
# B: flake8-bugbear (常见 Bug)
# UP: pyupgrade (自动升级旧语法，如 list() -> [])
# N: pep8-naming (变量命名规范)
select = ["E", "F", "I", "B", "UP", "N"]

# 5. 忽略的规则 (根据项目实际情况调整)
# 例如：忽略行过长 (E501)，因为 ruff format 会自动处理，处理不了的通常有特殊原因
ignore = ["E501"]

# 6. 自动修复的行为
fixable = ["ALL"]
unfixable = []

[tool.ruff.format]
# 7. 格式化风格配置
quote-style = "double"  # 字符串使用双引号
indent-style = "space"  # 使用空格缩进
skip-magic-trailing-comma = false # 尊重末尾逗号
line-ending = "auto"

[tool.ruff.lint.isort]
# 8. isort 配置 (将标准库、第三方库、本地库分开)
known-first-party = ["my_project_name"]
```