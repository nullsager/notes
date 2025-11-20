`fzf` (Fuzzy Finder) 是命令行极客的神器。它本质上是一个**通用的命令行模糊查找器**。它不产生数据，而是接收标准输入（stdin），允许你交互式地筛选，然后将选中的内容输出到标准输出（stdout）。

由于你知道 Arch Linux、Neovim 和 Zsh，这篇指南将直接切入**高阶配置**和**生产力整合**（Shell + Vim）。

-----

### 1\. 核心概念与基础语法

最简单的用法是将文件列表传给它：

```bash
find . | fzf
```

但在使用前，必须掌握它的搜索语法，这决定了你的筛选效率：

| 符号 | 说明 | 示例 | 匹配内容 |
| :--- | :--- | :--- | :--- |
| (无) | 模糊匹配 | `sb` | **s**ome\_**b**ar, **s**aa**b** |
| `'` | 精确匹配 (Exact) | `'sb` | **sb**in/bash (必须连在一起) |
| `^` | 前缀匹配 | `^main` | **main**. c |
| `$` | 后缀匹配 | `.c$` | main.**c** |
| `!` | 取反 (Inverse) | `!.c$` | 不包含 .c 的行 |
| ` | ` | 或 (Or) | `py$ | go$ ` | 以 py 或 go 结尾 |

> **技巧**: 你可以组合使用，例如 `^src 'main !test` (以 src 开头，包含 main，且不包含 test)。

-----

### 2\. Shell 集成 (Zsh/Bash)

这是 `fzf` 的灵魂。安装后（Arch: `sudo pacman -S fzf`），你需要启用按键绑定。

通常在 Arch Linux 上，安装包会自动放置脚本。你需要在 `.zshrc` 或 `.bashrc` 中 source 它们：

```bash
# 在 .zshrc 中
source /usr/share/fzf/key-bindings.zsh
source /usr/share/fzf/completion.zsh
```

启用后，你将获得三个超强快捷键：

1.  **`Ctrl + r`**: **命令历史搜索**。比默认的 reverse-i-search 强一万倍。支持模糊搜索过去输过的长命令，回车直接上屏。
2.  **`Ctrl + t`**: **文件搜索**。在命令行任何位置按下，选中的文件路径会自动填充到光标处。
      * *场景*: `vim config/` -\> 按 `Ctrl+t` -\> 搜 `ngin` -\> 选中 -\> 变成 `vim config/nginx.conf`。
3.  **`Alt + c`**: **目录跳转**。搜索子目录并直接 `cd` 进去。

-----

### 3\. 进阶配置：美化与预览 (FZF\_DEFAULT\_OPTS)

默认的 `fzf` 界面比较简陋。我们可以通过环境变量 `FZF_DEFAULT_OPTS` 来定制 UI 和功能。

#### 推荐工具搭配

为了达到最佳效果，强烈建议安装以下两个 Rust 工具（都在 Arch 官方源里）：

  * **`fd`**: 比 `find` 快得多的查找工具，且默认忽略 `.git` 和 `.gitignore`。
  * **`bat`**: 带语法高亮的 `cat` 替代品。

#### 终极配置代码

将以下内容加入你的 `.zshrc`：

```bash
# 1. 让 fzf 使用 fd 代替 find (速度更快，忽略 gitignore)
export FZF_DEFAULT_COMMAND='fd --type f --strip-cwd-prefix --hidden --follow --exclude .git'
export FZF_CTRL_T_COMMAND="$FZF_DEFAULT_COMMAND"

# 2. 界面配置与预览窗口
# --height: 占用屏幕高度
# --layout: reverse 让搜索栏在顶部
# --border: 边框样式
# --preview: 使用 bat 预览文件内容
export FZF_DEFAULT_OPTS="
  --height 60% 
  --layout=reverse 
  --border rounded 
  --info=inline 
  --prompt='Search > '
  --pointer='▶' 
  --marker='✓'
  --bind '?:toggle-preview'
  --preview 'bat --style=numbers --color=always --line-range :500 {}'
"

# 3. 针对 Alt-c (目录跳转) 的特殊预览 (使用 tree)
export FZF_ALT_C_OPTS="
  --preview 'tree -C {} | head -200'
"
```

**效果：**
当你按下 `Ctrl+t` 找文件时，右侧会直接出现该代码文件的**语法高亮预览**。

-----

### 4\. 实战：自定义 Fzf 脚本 (Workflow)

`fzf` 的强大在于你可以把它插入到任何管道中。以下是几个经典的“单行脚本”：

#### A. 交互式杀进程 (Kill Process)

不用再手动输 PID 了。

```bash
# 别名 fkill
alias fkill="ps -ef | fzf --header-lines=1 --layout=reverse | awk '{print \$2}' | xargs kill -9"
```

#### B. 切换 Git 分支

显示所有分支，带提交记录预览，回车切换。

```bash
alias gsw="git branch -a | fzf --preview 'git show --color=always {-1}' --height=50% --border | sed 's/remotes\/origin\///' | xargs git checkout"
```

#### C. 查找并用 Neovim 打开 (配合 rg)

使用 `ripgrep` (rg) 搜索文件内容，找到后用 `vim` 打开对应行。

```bash
# 这是一个脚本函数
fv() {
  IFS=: read -r file line <<< "$(rg --line-number --no-heading --color=always --smart-case "$@" | fzf --ansi --delimiter : --preview 'bat --style=numbers --color=always --highlight-line {2} {1}' | awk -F: '{print $1 ":" $2}')"
  [ -n "$file" ] && nvim "$file" +/"$line"
}
```

-----

### 5\. Neovim 集成

既然你用 Neovim，你有两个主流选择将 `fzf` 嵌入编辑器：

#### 选项一：`fzf-lua` (强烈推荐)

这是专为 Neovim 编写的 Lua 版本，速度极快，利用了 Neovim 的内置浮窗 API，且不需要外部 Ruby/Python 依赖。

  * **安装 (Lazy. nvim):**
    ```lua
    {
      "ibhagwan/fzf-lua",
      dependencies = { "nvim-tree/nvim-web-devicons" },
      config = function()
        require("fzf-lua").setup({
            winopts = {
                preview = {
                    layout = "vertical", -- 适合宽屏
                }
            }
        })
        -- keymaps
        vim.keymap.set("n", "<leader>ff", require('fzf-lua').files, { desc = "Fzf Files" })
        vim.keymap.set("n", "<leader>fg", require('fzf-lua').live_grep, { desc = "Fzf Grep" })
      end
    }
    ```

#### 选项二：`fzf.vim` (经典)

这是 Vim 社区最老牌的插件，作者是 Junegunn (也是 fzf 的作者)。如果你需要在 Vim 和 Neovim 之间共用配置，选这个。

-----

### 6\. 总结与检查清单

要构建一个完美的 `fzf` 环境，请按此顺序操作：

1.  **安装基础工具**: `pacman -S fzf fd bat ripgrep`
2.  **配置 Shell**: 在 `.zshrc` 中添加 `FZF_DEFAULT_OPTS` (设置预览窗口) 和 `source` 按键绑定。
3.  **配置 Neovim**: 安装 `ibhagwan/fzf-lua`。
4.  **习惯养成**: 强迫自己停止使用 `cd` 漫游，改用 `Alt-c`；停止使用 `history | grep`，改用 `Ctrl-r`。

-----