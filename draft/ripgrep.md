`fzf` 和 `ripgrep` (简写为 `rg`) 是现代终端中最强大的两个命令行工具。它们一个负责**“模糊过滤与交互”**，另一个负责**“极速文本搜索”**。将它们结合起来，你的终端搜索效率将得到质的飞跃。

下面我为你详细拆解这两个工具的用法，以及它们如何“双剑合璧”。

---

### 第一部分：Ripgrep (`rg`) —— 极速文本搜索工具

`ripgrep` 是一个用 Rust 编写的面向行的搜索工具（类似于 `grep`）。它的最大特点是**极其快**，并且默认会自动忽略 `.gitignore` 中的文件和隐藏文件。

#### 1. 安装
*   **macOS:** `brew install ripgrep`
*   **Linux (Ubuntu/Debian):** `sudo apt install ripgrep`
*   **Windows:** `choco install ripgrep` 或 `scoop install ripgrep`

#### 2. 基本用法
```bash
# 在当前目录下搜索包含 "hello" 的内容
rg "hello"

# 在指定目录搜索
rg "hello" /path/to/dir
```

#### 3. 常用高频参数（必学）
*   **忽略大小写：**
    *   `-i` : 完全忽略大小写 (`rg -i "hello"`)。
    *   `-S` : **智能大小写**（推荐）。如果你的搜索词全是小写，它就忽略大小写；如果包含大写字母，它就精确匹配 (`rg -S "Hello"`)。
*   **全词匹配：**
    *   `-w` : 只匹配完整的单词。搜索 "app" 不会匹配出 "apple" (`rg -w "app"`)。
*   **指定文件类型搜索：**
    *   `-t` : 只在特定类型文件中搜索。例如只在 Python 文件中搜索 (`rg -t py "def main"`)。
    *   `-T` : 排除特定类型文件 (`rg -T js "function"`)。
    *   *提示：输入 `rg --type-list` 可以查看所有支持的文件类型。*
*   **使用通配符（按文件名过滤）：**
    *   `-g` : 例如只在 Markdown 文件中搜 (`rg "TODO" -g "*.md"`)。排除测试文件 (`rg "bug" -g "!*test*"`)。
*   **显示上下文（和 grep 一样）：**
    *   `-C 2` : 显示匹配行的前后各 2 行。
    *   `-A 2` : 显示匹配行及**后** 2 行 (After)。
    *   `-B 2` : 显示匹配行及**前** 2 行 (Before)。
*   **只列出包含匹配项的文件名：**
    *   `-l` : 不显示具体匹配了哪一行，只列出文件路径 (`rg -l "password"`)。
*   **搜索隐藏文件：**
    *   `--hidden` : 默认不搜隐藏文件，加上这个参数即可搜索 (`rg --hidden "config"`)。
*   **不忽略 `.gitignore`：**
    *   `--no-ignore` : 强制搜索被 git 忽略的文件。

---

### 第二部分：fzf —— 命令行模糊查找器

`fzf` (Fuzzy Finder) 是一个用 Go 编写的通用模糊查找器。它的核心逻辑是：**接收一段多行文本输入 -> 提供交互式搜索界面 -> 输出你选中的那一行。**

#### 1. 安装
*   **macOS:** `brew install fzf`
*   **Linux:** `sudo apt install fzf`
*   **重要一步（绑定快捷键）:** 安装后，强烈建议运行它的安装脚本以启用快捷键和自动补全：
    ```bash
    $(brew --prefix)/opt/fzf/install  # macOS brew 用户
    # 或者如果你是通过 git clone 安装的，运行 ~/.fzf/install
    ```

#### 2. 核心快捷键（终端神器）
运行了上面的安装脚本后，重启终端，你将获得三个改变习惯的快捷键：
*   **`Ctrl + R` (历史命令搜索):** 替代原生的 `Ctrl+R`。你可以输入零散的关键词，瞬间找到几个月前敲过的一长串复杂命令。
*   **`Ctrl + T` (文件路径补全):** 在敲命令时按下。例如你想 `vim` 某个深层目录的文件，输入 `vim `，然后按 `Ctrl+T`，模糊搜索文件，回车，文件路径直接上屏。
*   **`Alt + C` (快速 cd):** 按下后模糊搜索当前目录下的所有子目录，选中回车后直接 `cd` 进去。

#### 3. 基本命令用法
单独输入 `fzf`，它会列出当前目录下的所有文件让你搜索。
*   **输入关键词：** 比如输入 `corejs` 可以匹配到 `src/core/main.js`。
*   **操作：** 键盘 `上下方向键` 选择，`Enter` 确认并输出路径，`Ctrl+C` 或 `Esc` 退出。

#### 4. 高阶参数
*   **多选模式：**
    *   `-m` : 允许使用 `Tab` 键（或 `Shift+Tab`）选择多行。
    *   例如：`vim $(fzf -m)` 可以搜索并同时用 vim 打开多个文件。
*   **预览窗口：**
    *   `--preview` : 在搜索时预览内容。
    *   例如：`fzf --preview 'cat {}'` （`{}` 代表当前光标选中的文件路径）。如果你安装了 `bat`（带高亮的 cat），可以用 `fzf --preview 'bat --color=always {}'`。

---

### 第三部分：双剑合璧 (Ripgrep + fzf)

单独用很强，结合起来更是天下无敌。

#### 玩法 1：用 rg 替代 fzf 的默认文件搜索底层
`fzf` 默认使用 `find` 命令找文件，速度较慢且会搜出一堆没用的 `.git` 和 `node_modules` 文件。我们可以配置 `fzf` 默认使用 `rg`。
在你的 `~/.bashrc` 或 `~/.zshrc` 中添加以下环境变量：

```bash
# 让 fzf 默认使用 rg 找文件（更快，且尊重 .gitignore）
export FZF_DEFAULT_COMMAND='rg --files --hidden --glob "!.git"'
export FZF_CTRL_T_COMMAND="$FZF_DEFAULT_COMMAND"
export FZF_ALT_C_COMMAND="rg --sort-files --files --null | xargs -0 dirname | awk '!seen[\$0]++'"
```
*配置后，你的 `Ctrl+T` 找文件将会变得极快。*

#### 玩法 2：在搜索结果中二次模糊过滤
你记得项目中有一个文件包含了 "database"，但不记得具体在哪。
```bash
rg "database" | fzf
```
`rg` 会瞬间搜出所有包含 database 的行，然后通过管道 `|` 交给 `fzf`，你可以在弹出的界面中继续输入文件名或代码片段进行二次过滤。

#### 玩法 3：终极交互式全局搜索与预览（进阶配置）
你想在整个项目中搜索内容，并且想**实时看到高亮的代码预览**。你可以把下面这段代码作为一个 alias（别名）或者函数放到你的 `~/.zshrc` 中（需要安装 `bat` 工具来提供语法高亮）：

```bash
# 交互式搜索文件内容 (Interactive Grep)
# 依赖: ripgrep (rg), fzf, bat
igrep() {
  rg --line-number --no-heading --color=always --smart-case "${*:-}" |
  fzf --ansi \
      --color "hl:-1:underline,hl+:-1:underline:reverse" \
      --delimiter : \
      --preview 'bat --color=always {1} --highlight-line {2}' \
      --preview-window 'up,60%,border-bottom,+{2}+3/3,~3'
}
```
**如何使用：**
1. 输入 `igrep` 启动。
2. 界面会列出当前目录下所有的代码行。
3. 输入关键词，`fzf` 会模糊过滤 `rg` 提供的内容。
4. 上半部分（或右半部分）会实时显示带语法高亮的代码上下文。
5. 选中回车后，会输出 `文件名:行号:内容`。

#### 玩法 4：在 Vim / Neovim 中使用
如果你是 Vim/Neovim 用户，这两个工具是必备插件的底层。
安装 `fzf.vim` 或 `Telescope.nvim` 插件后，你可以在编辑器内直接调用 `rg` 搜索，用 `fzf` 过滤，按下回车直接跳转到对应文件的对应行。

### 总结
*   **找文件在哪里，或者按正则找文本** -> 优先用 `rg`。
*   **不知道完整路径，只记得零碎字母，想找文件/历史命令** -> 优先用 `fzf`。
*   **在一个巨大的项目里找一段模糊的代码** -> `rg` 搜全项目交由 `fzf` 过滤。