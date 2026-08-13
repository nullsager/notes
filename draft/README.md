# Neovim 个人配置手册

> 基于 **Neovim 0.12+** 的模块化配置，使用内置 `vim.pack` 管理插件。
> 面向 C/C++、Python 开发、Markdown 笔记（Obsidian）与 Org mode 任务管理工作流，
> 集成 DeepSeek AI 助手。

**Leader 键 = 空格（`<Space>`）**，按下后稍作停留会弹出 which-key 快捷键提示面板。

---

## 目录

- [一、配置结构](#一配置结构)
- [二、环境与依赖](#二环境与依赖)
- [三、插件清单与管理](#三插件清单与管理)
- [四、编辑器基础行为](#四编辑器基础行为)
- [五、快捷键速查表](#五快捷键速查表)
- [六、功能详解](#六功能详解)
- [七、自定义指南](#七自定义指南)
- [八、常见问题（FAQ）](#八常见问题faq)
- [九、备份与回滚](#九备份与回滚)

---

## 一、配置结构

```
~/.config/nvim/
├── init.lua                    入口：按顺序加载各模块，本身不含逻辑
├── README.md                   本文件
├── nvim-pack-lock.json         插件版本锁定文件（vim.pack 自动生成，勿手改）
├── nvim.log                    Neovim 运行日志（自动生成）
├── snippets/                   自定义代码片段（文件名 = filetype，如 org.json）
└── lua/
    ├── core/                   与插件无关的基础配置
    │   ├── options.lua         编辑器选项（含 Leader 键，必须最先加载）
    │   ├── keymaps.lua         通用快捷键、代码格式化
    │   ├── terminal.lua        底部终端切换（<C-\>）
    │   └── autocmds.lua        自动命令：fcitx5 输入法同步、恢复光标位置
    └── plugins/                每个插件一个文件，配置与其快捷键放在一起
        ├── init.lua            插件清单（vim.pack.add）+ 各插件模块的加载入口
        ├── completion.lua      自动补全：blink.cmp + mini.pairs
        ├── lsp.lua             LSP：clangd / pyright + 诊断开关
        ├── fzf.lua             模糊搜索：fzf-lua
        ├── treesitter.lua      语法高亮 / 折叠 / 缩进 / 语法对象
        ├── obsidian.lua        Obsidian 笔记 + 笔记元素快速插入
        ├── orgmode.lua         Org mode：任务 / 议程 / 速记
        ├── codecompanion.lua   AI 助手（DeepSeek）
        └── markdown.lua        Markdown 渲染：render-markdown + mini.icons
```

**加载顺序**（`init.lua`）：

```
core.options  →  core.keymaps  →  core.terminal  →  core.autocmds  →  plugins
```

`core.options` 必须最先加载，因为 `vim.g.mapleader` 需要在所有快捷键映射之前设置；
插件配置放在最后，因为 `vim.pack.add` 会把插件目录加入 `runtimepath`。

---

## 二、环境与依赖

### 必需

| 依赖 | 用途 | 安装（Arch Linux） |
|------|------|---------------------|
| Neovim ≥ 0.12 | 本配置使用 `vim.pack`、`vim.lsp.enable` 等新 API | `sudo pacman -S neovim` |
| git | vim.pack 克隆插件 | `sudo pacman -S git` |
| fzf | fzf-lua 的后端 | `sudo pacman -S fzf` |
| Nerd Font | 显示补全图标、Markdown 标题图标 | 如 `ttf-jetbrainsmono-nerd`，并在终端中启用 |

### 语言服务器与格式化工具

| 工具 | 用途 | 当前状态 |
|------|------|----------|
| clangd | C/C++ LSP | 已安装 |
| pyright | Python LSP | 已安装 |
| clang-format | C/C++ 格式化（`<leader>cf`） | 已安装 |
| ruff | Python 格式化（`<leader>cf`） | **未安装**，需 `sudo pacman -S ruff` |

### 可选

| 依赖 | 用途 |
|------|------|
| fcitx5 + fcitx5-remote | 输入法状态自动切换（见 [fcitx5 输入法自动切换](#61-fcitx5-输入法自动切换)） |
| wl-clipboard / xclip | 系统剪贴板互通（`unnamedplus`） |
| `DEEPSEEK_API_KEY` 环境变量 | CodeCompanion 调用 DeepSeek API（见 [CodeCompanion AI 助手](#68-codecompanion-ai-助手)） |
| C 编译器（cc / gcc / clang） | 首次启动时编译 Org 的 Treesitter parser（一次性，见 [Org mode 任务管理](#610-org-mode-任务管理)） |
| pandoc 或 emacs | Org 导出为 HTML / PDF 等格式（不用导出功能可忽略） |

---

## 三、插件清单与管理

### 插件清单（18 个）

| 插件 | 用途 |
|------|------|
| which-key.nvim | 快捷键提示面板（无需 setup，自动生效） |
| mini.nvim | 使用其中的 mini.pairs（自动配对括号引号） |
| mini.icons | 图标库 |
| blink.cmp（`1.*`） | 自动补全引擎 |
| blink-cmp-latex | LaTeX 命令补全源 |
| friendly-snippets | 代码片段集合 |
| nvim-lspconfig | LSP 官方配置集合 |
| fzf-lua | 模糊搜索 |
| nvim-web-devicons | 文件类型图标 |
| obsidian.nvim | Obsidian 笔记库集成 |
| orgmode | Org mode：任务管理、议程（Agenda）、速记（Capture） |
| render-markdown.nvim | Markdown 实时渲染 |
| markdown-preview.nvim | Markdown 浏览器实时预览（`<leader>mp`；退出 Neovim 后页面保留为静态快照） |
| plenary.nvim | 通用 Lua 依赖库 |
| treesitter-parser-registry | Treesitter parser 注册表 |
| nvim-treesitter | 语法分析：高亮 / 折叠 / 缩进 |
| nvim-treesitter-textobjects | 语法对象（函数、类、参数……） |
| codecompanion.nvim（`^19`） | AI 助手，接入 DeepSeek |

### 使用 vim.pack 管理插件

插件声明集中在 `lua/plugins/init.lua`，版本锁定在 `nvim-pack-lock.json`。

```lua
-- 添加插件：在 lua/plugins/init.lua 的 vim.pack.add({...}) 中加一行
"https://github.com/owner/repo",

-- 指定版本（示例）
{ src = "https://github.com/owner/repo", version = vim.version.range("1.*") },
```

常用命令：

| 命令 | 作用 |
|------|------|
| `:lua vim.pack.update()` | 更新所有插件（会逐个列出变更并询问确认） |
| `:lua vim.pack.update({ "fzf-lua" })` | 只更新指定插件 |
| `:lua vim.pack.get()` | 查看已安装插件及状态 |
| `:lua vim.pack.del({ "插件名" })` | 删除插件（同时从清单中移除该行） |
| `:lua vim.pack.clean()` | 清理清单中已不存在的插件目录 |

> 提示：`nvim-pack-lock.json` 记录了每个插件的精确 commit，把它纳入版本管理（git）
> 即可在任意机器上复现完全一致的插件版本。

---

## 四、编辑器基础行为

| 选项 | 值 | 说明 |
|------|-----|------|
| `number` + `relativenumber` | 开 | 当前行显示绝对行号，其余行显示相对行号 |
| `termguicolors` | 开 | 24-bit 真彩色 |
| `cursorline` | 开 | 高亮光标所在行 |
| 主题 | catppuccin | 来自系统运行时（Arch neovim 包附带） |
| `winborder` | `single` | 所有浮窗使用单线边框 |
| `clipboard` | `+unnamedplus` | 与系统剪贴板互通（需 wl-clipboard/xclip） |
| `scrolloff` / `sidescrolloff` | 10 | 光标距窗口上下/左右边缘至少保留 10 行/列 |
| `tabstop` / `shiftwidth` / `softtabstop` | 2 | Tab 宽度与缩进宽度均为 2 |
| `expandtab` | 开 | 用空格代替 Tab |
| `smartindent` / `autoindent` | 开 | 智能自动缩进 |
| `undofile` | 开 | 持久化撤销历史，关闭文件后仍可撤销 |
| `splitbelow` / `splitright` | 开 | 新分屏默认在下方/右方 |

---

## 五、快捷键速查表

> 模式缩写：**n** 普通 / **i** 插入 / **v** 可视 / **x** 可视(不含选择) / **o** 操作符等待 / **t** 终端

### 5.1 基础编辑与窗口（core/keymaps.lua）

| 快捷键 | 模式 | 作用 |
|--------|------|------|
| `jk` | i | 退出插入模式（等同 `<Esc>`） |
| `jk` | t | 退出终端模式（等同 `<C-\><C-n>`） |
| `<C-q>` | i / n / t | 强制删除当前 buffer（`:bd!`） |
| `<C-h>` / `<C-j>` / `<C-k>` / `<C-l>` | n | 移动光标到 左 / 下 / 上 / 右 窗口 |
| `<C-Up>` / `<C-Down>` | i / n / t | 窗口高度 ±3 |
| `<C-Left>` / `<C-Right>` | i / n / t | 窗口宽度 ±3 |
| `<leader>cf` | n | 格式化当前文件（见 [代码格式化](#65-代码格式化)） |
| `<leader>zM` | n | 折叠所有代码块 |
| `<leader>zR` | n | 展开所有代码块 |

### 5.2 终端（core/terminal.lua）

| 快捷键 | 模式 | 作用 |
|--------|------|------|
| `<C-\>` | n / t | 切换底部终端（占屏 30%，会话保留，详见 [底部终端](#63-底部终端)） |

### 5.3 文件浏览 netrw（内置）

| 快捷键 | 模式 | 作用 |
|--------|------|------|
| `<leader>e` | n | 在左侧开关文件树（`:Lexplore`） |

netrw 已定制为树形显示、宽 20%、回车在右侧窗口打开文件。常用内置按键：

| 按键 | 作用 |
|------|------|
| `回车` | 打开文件 / 展开收起目录 |
| `%` / `d` | 新建文件 / 新建目录 |
| `D` / `R` | 删除 / 重命名 |
| `I` | 显示/隐藏顶部帮助横幅 |
| `<C-l>` | 刷新列表 |

### 5.4 模糊搜索 fzf-lua

**文件与内容**（`<leader>f` 前缀）：

| 快捷键 | 模式 | 作用 |
|--------|------|------|
| `<leader>ff` | n | 模糊查找文件 |
| `<leader>fg` | n | 全项目实时搜索文本（live grep） |
| `<leader>fw` | n | 搜索光标下的单词 |
| `<leader>fw` | v | 搜索选中的文本 |
| `<leader>fb` | n | 在已打开的 buffer 间切换 |
| `<leader>fr` | n | 最近打开过的文件 |
| `<leader>fR` | n | 恢复上一次的搜索窗口 |

**Neovim 内置资源**：

| 快捷键 | 模式 | 作用 |
|--------|------|------|
| `<leader>fh` | n | 搜索帮助文档（help tags） |
| `<leader>fk` | n | 搜索所有快捷键映射 |
| `<leader>fc` | n | 搜索所有可用命令 |

**LSP 与诊断**：

| 快捷键 | 模式 | 作用 |
|--------|------|------|
| `<leader>fd` | n | 当前文件的诊断列表（显示开关见 `<leader>ul`） |
| `<leader>fs` | n | 当前文件的符号大纲（函数 / 类等） |

**Git**（`<leader>g` 前缀，依赖系统 git 命令，需在 Git 仓库内使用）：

| 快捷键 | 模式 | 作用 |
|--------|------|------|
| `<leader>gf` | n | 查找被 Git 跟踪的文件 |
| `<leader>gs` | n | 查看 Git 状态（改动文件） |
| `<leader>gc` | n | 浏览提交历史 |
| `<leader>gb` | n | 浏览 / 切换分支 |

> fzf-lua 还有大量内置命令可直接使用，如 `:FzfLua lsp_references`、
> `:FzfLua quickfix` 等，输入 `:FzfLua` 后按 Tab 可查看全部。

### 5.5 自动补全 blink.cmp（插入模式）

补全菜单自动弹出，也可用 `<C-Space>` 手动唤起：

| 快捷键 | 作用 |
|--------|------|
| `<C-n>` / `<C-p>`（或 `↓` / `↑`） | 选择下 / 上一个候选 |
| `<CR>` | 确认补全 |
| `<C-y>` | 确认补全（同上） |
| `<C-e>` | 取消并关闭菜单 |
| `<C-Space>` | 打开补全菜单 / 展开或收起文档浮窗 |
| `<C-b>` / `<C-f>` | 在文档浮窗中向上 / 向下滚动 |
| `<C-k>` | 显示 / 隐藏函数签名帮助 |
| `<Tab>` / `<S-Tab>` | snippet 占位符之间向前 / 向后跳转 |

补全行为特点：

- 候选**不预选、不自动插入**，必须手动确认（`preselect = false`）
- 文档浮窗延迟 300ms 自动弹出
- 补全菜单三列显示：补全项与描述 | 类型图标与类型 | 来源（lsp/path/snippets/buffer/LaTeX）

### 5.6 LSP 与诊断

| 快捷键 | 模式 | 作用 |
|--------|------|------|
| `<leader>ul` | n | 全局开关诊断信息（**默认关闭**） |
| `gl` | n | 浮窗显示当前行诊断详情 |

以下快捷键为 Neovim 0.12 **内置**的 LSP 映射（clangd/pyright attach 后可用）：

| 快捷键 | 作用 |
|--------|------|
| `K` | 悬浮文档（hover） |
| `grn` | 重命名符号 |
| `gra` | Code Action |
| `grr` | 跳转到引用列表 |
| `gri` | 跳转到实现 |
| `grt` | 跳转到类型定义 |
| `gO` | 文档符号大纲 |
| `<C-s>`（插入模式） | 函数签名帮助 |

### 5.7 Treesitter 语法对象（plugins/treesitter.lua）

**选择**（x / o 模式，可配合 `d`、`c`、`y` 等操作符）：

| 快捷键 | 选中目标 | 示例 |
|--------|----------|------|
| `af` / `if` | 整个函数 / 函数体 | `daf` 删除函数、`vif` 选中函数体 |
| `ac` / `ic` | 整个类 / 类内部 | `yac` 复制整个类 |
| `aa` / `ia` | 函数参数（含 / 不含逗号） | `daa` 删除当前参数 |
| `al` / `il` | 整个循环 / 循环体 | `cil` 改写循环体 |
| `ai` / `ii` | 整个条件分支 / 分支内部 | `vai` 选中 if 块 |

**跳转**（n / x / o 模式，记入 jumplist，可用 `<C-o>` 跳回）：

| 快捷键 | 作用 |
|--------|------|
| `]f` / `[f` | 下一个 / 上一个函数开头 |
| `]F` / `[F` | 下一个 / 上一个函数结尾 |
| `]c` / `[c` | 下一个 / 上一个类开头 |

**交换参数**（n 模式）：

| 快捷键 | 作用 |
|--------|------|
| `<leader>sn` | 当前参数与下一个参数交换 |
| `<leader>sp` | 当前参数与上一个参数交换 |

### 5.8 Obsidian 笔记（plugins/obsidian.lua）

| 快捷键 | 模式 | 作用 |
|--------|------|------|
| `<leader>oo` | n | 用 netrw 浏览整个笔记库 |
| `<leader>of` | n | 按文件名快速打开笔记（fzf） |
| `<leader>os` | n | 全文搜索笔记内容 |
| `<leader>ob` | n | 查看当前笔记的反向链接 |
| `<leader>ot` | n | 打开 / 创建今天的日记 |
| `<leader>on` | n | 输入标题创建新笔记（存入 `draft/`） |
| `<leader>op` | n | 粘贴剪贴板图片到笔记（存入 `images/`） |
| `<leader>ov` | n | 保存并在 Obsidian 应用中打开当前笔记 |
| `<leader>mc` | n | 插入 Callout（9 种类型菜单选择，见下文） |
| `<leader>mt` | n | 插入 Markdown 表格（输入 列x行，自动对齐） |
| `<leader>mb` | n / v | 切换复选框状态 |

### 5.9 CodeCompanion AI 助手（plugins/codecompanion.lua）

**全局快捷键**：

| 快捷键 | 模式 | 作用 |
|--------|------|------|
| `<leader>aa` | n / v | 打开动作面板（所有预设动作的菜单） |
| `<leader>ac` | n | 打开 / 隐藏当前聊天窗口 |
| `<leader>an` | n | 新建聊天 |
| `<leader>ai` | n | Inline 模式：让 AI 生成 / 修改光标附近代码 |
| `<leader>ai` | v | 对选中代码执行 Inline 请求 |
| `<leader>ad` | v | 把选中代码追加到当前聊天 |
| `<leader>ae` | v | 解释选中代码（`/explain`） |
| `<leader>af` | v | 修复选中代码（`/fix`） |
| `<leader>at` | v | 为选中代码生成测试（`/tests`） |
| `<leader>aF` | n | 使用 deepseek-v4-flash 新建聊天 |
| `<leader>aP` | n | 使用 deepseek-v4-pro 新建聊天 |
| `<leader>ag` | n | 根据暂存区生成 Git commit message（`/commit`） |

**聊天窗口内置按键**（codecompanion 默认）：

| 按键 | 模式 | 作用 |
|------|------|------|
| `?` | n | 显示全部聊天按键帮助 |
| `<CR>` 或 `<C-s>` | n（`<C-s>` 也可用于 i） | 发送消息 |
| `q` | n | 停止当前请求 |
| `gr` | n | 重新生成上一条回复 |
| `gx` | n | 清空聊天记录 |
| `<C-c>` | n / i | 关闭聊天窗口 |
| `ga` | n | 切换 adapter / 模型 |
| `gc` | n | 插入空代码块 |
| `gy` | n | 复制最后一个代码块 |
| `gf` | n | 折叠所有代码块 |
| `}` / `{` | n | 下一个 / 上一个聊天 |
| `]]` / `[[` | n | 跳到下 / 上一个消息标题 |

### 5.10 Org mode（plugins/orgmode.lua）

**全局快捷键**（任意 buffer 可用）：

| 快捷键 | 模式 | 作用 |
|--------|------|------|
| `<leader>oa` | n | 打开 Agenda 菜单（`a` 周议程 / `t` 待办列表 / `m` 标签过滤 / `s` 搜索） |
| `<leader>oc` | n | 打开 Capture 速记菜单（默认 `t` = Task 模板） |

**.org 文件内**（buffer 局部映射，完整帮助随时按 `g?`）：

| 快捷键 | 作用 |
|--------|------|
| `<Tab>` / `<S-Tab>` | 折叠 / 展开当前标题；循环全文折叠级别 |
| `cit` / `ciT` | 向前 / 向后切换 TODO 状态 |
| `<leader>ot` | 修改标题标签（tags） |
| `<leader>o,` | 设置优先级（A / B / C） |
| `<leader>mb` | 切换复选框 `- [ ]` / `- [X]`（默认 `<C-Space>`，因与输入法切换冲突改键） |
| `<<` / `>>` | 提升 / 降低当前标题层级 |
| `<s` / `>s` | 提升 / 降低整个子树层级 |
| `<leader>o*` | 当前行在标题与正文间互转 |
| `<leader><CR>` | 智能新建同级标题 / 列表项 / 表格行（依上下文） |
| `<leader>oih` / `<leader>oiT` / `<leader>oit` | 内容后插入标题 / 紧随其后插入 TODO / 内容后插入 TODO |
| `<leader>oK` / `<leader>oJ` | 子树上移 / 下移 |
| `}` / `{` | 下一个 / 上一个可见标题 |
| `]]` / `[[` | 下一个 / 上一个同级标题 |
| `<leader>oid` / `<leader>ois` | 设置 DEADLINE / SCHEDULED 日期 |
| `<leader>oi.` / `<leader>oi!` | 插入主动 / 非主动时间戳 |
| `<C-a>` / `<C-x>` | 增减光标处的日期或时间 |
| `<S-Up>` / `<S-Down>` | 按天增减光标处日期 |
| `cid` | 弹出日历修改光标处日期 |
| `<leader>oo` | 打开光标处的链接 / 日期 |
| `<leader>oli` / `<leader>ols` | 插入链接 / 存储当前位置的链接 |
| `<leader>or` | Refile：把当前标题移动到其他文件 / 标题下 |
| `<leader>o$` / `<leader>oA` | 归档子树到 `*_archive` 文件 / 切换 ARCHIVE 标签 |
| `<leader>ona` | 为当前标题添加备注 |
| `<leader>oxi` / `<leader>oxo` / `<leader>oxj` | 开始 / 结束 / 跳转到计时（clocking） |
| `<leader>oe` | 导出（HTML / PDF 等，需 pandoc 或 emacs） |
| `<leader>o'` | 在独立窗口编辑光标所在的 `#+begin_src` 代码块（仅限代码块，不支持表格） |

**Agenda 视图内**：

| 快捷键 | 作用 |
|--------|------|
| `f` / `b` | 下一个 / 上一个时间段 |
| `.` | 回到今天 |
| `vd` / `vw` / `vm` / `vy` | 切换 日 / 周 / 月 / 年 视图 |
| `J` | 跳转到指定日期 |
| `<CR>` / `<Tab>` | 在当前窗口 / 其他窗口打开任务 |
| `r` | 刷新视图 |
| `t` | 切换任务的 TODO 状态 |
| `/` | 按标签 / 关键字过滤 |
| `K` | 预览任务在文件中的位置 |
| `q` | 退出 Agenda |

**Capture 窗口内**：`<C-c>` 保存并关闭，`<leader>or` refile 到指定位置，`<leader>ok` 放弃。

> 冲突说明：org buffer 内 `<leader>oo` / `<leader>ot` 是插件的 buffer 局部映射，
> 会覆盖 Obsidian 的同名全局映射（仅在 org 文件内生效，见 [Q8](#八常见问题faq)）。
> 另外复选框切换键插件默认为 `<C-Space>`，与 fcitx5 输入法切换冲突，
> 已在 `orgmode.lua` 中改为 `<leader>mb`（与 Markdown 复选框键一致，
> 在 org buffer 内覆盖 Obsidian 的同名全局映射，语义相同）。

---

## 六、功能详解

### 6.1 fcitx5 输入法自动切换

位置：`lua/core/autocmds.lua`（系统安装 fcitx5-remote 时自动启用）

解决"插入模式用中文输入，退回普通模式后快捷键被输入法拦截"的经典痛点：

| 时机 | 行为 |
|------|------|
| `InsertLeave` / `TermLeave` | 记录当前输入法状态；若处于中文状态则切换为英文 |
| `InsertEnter` / `TermEnter` | 恢复之前记录的中文状态 |
| `VimEnter` | 启动时确保普通模式为英文 |
| `VimLeavePre` | 退出 Neovim 时恢复系统输入法状态 |

效果：普通模式下 `hjkl` 等快捷键永远可用，回到插入模式中文输入无缝恢复。

### 6.2 恢复上次光标位置

位置：`lua/core/autocmds.lua`

重新打开文件时自动跳回上次退出时的光标位置（`BufReadPost` 事件，读取 `"` 标记）。
diff 模式下自动跳过；标记位置超出文件行数时安全忽略。

### 6.3 底部终端

位置：`lua/core/terminal.lua`，快捷键 `<C-\>`

- 在窗口底部打开占屏 **30%** 的终端，再按一次隐藏
- 终端 **buffer 常驻**：隐藏后再打开会回到同一会话（命令历史、运行状态都在）
- 终端窗口自动关闭行号与标志列，打开即进入输入模式
- 在终端里按 `jk` 退回普通模式，可像普通 buffer 一样滚动复制
- 边界保护：终端是唯一窗口时按 `<C-\>` 不会报 E444 错误

### 6.4 Treesitter：高亮 / 折叠 / 缩进

位置：`lua/plugins/treesitter.lua`

**parser 管理**：`treesitter_parsers` 列表中的 25 个 parser 在启动时异步安装缺失项，
已安装的不会重复安装。想支持新语言，往列表里加一行即可。

**按 filetype 启用**（`FileType` 自动命令）：

1. **高亮**：`vim.treesitter.start()`（含语言注入，如 Markdown 代码块内按各自语言高亮）
2. **折叠**：`foldmethod = expr`，打开文件时 `foldlevel = 99`（默认全部展开）
3. **智能缩进**：对 `treesitter_indent_filetypes` 中的语言启用实验性 Treesitter 缩进；
   若某语言缩进异常，从该表中删掉对应行即可回退到默认缩进

折叠相关按键（Vim 内置）：`zc` 折叠 / `zo` 展开 / `za` 切换当前折叠，
配合 `<leader>zM` / `<leader>zR` 全局折叠与展开。

> 注：CodeCompanion 聊天 buffer 被注册为 markdown 文件类型，
> 因此聊天内容同样享受 Markdown 高亮与渲染。

### 6.5 代码格式化

位置：`lua/core/keymaps.lua`，快捷键 `<leader>cf`

| 文件类型 | 命令 |
|----------|------|
| c / cpp | `clang-format`（项目根目录的 `.clang-format` 文件自动生效） |
| python | `ruff format -` |

安全机制（重构时加固）：

- 当前 filetype 无对应工具 → 仅提示，不动 buffer
- 格式化工具未安装 → 仅提示，不动 buffer（避免 `%!` 过滤器把 shell 报错写进文件）
- 命令执行失败 → 自动 `undo` 恢复原内容并提示
- 成功后恢复光标位置与视图

**添加新语言**：在 `keymaps.lua` 的 `formatters` 表中加一行，例如 `go = "gofmt"`。

### 6.6 补全、Snippet 与 LaTeX

位置：`lua/plugins/completion.lua`

**补全源**（默认）：`lsp` → `path` → `snippets` → `buffer`

**按文件类型扩展**（`per_filetype`）：`markdown`、`codecompanion`、`tex`、`plaintex`
额外启用 `latex` 源。

**LaTeX 补全**（blink-cmp-latex）：

- 在 Markdown 中输入 `\alpha`、`\sum` 等会出现补全候选
- `insert_command = true`：确认后插入的是 **LaTeX 命令本身**（`\alpha`），
  而不是 Unicode 字符（α），符合 Markdown 公式编写习惯
- `score_offset = 100`：LaTeX 候选排在最前

**Snippet**（friendly-snippets 自动加载）：

- `markdown` 文件额外加载 `tex` 的 snippets
- `codecompanion` 额外加载 `markdown` + `tex` 的 snippets
- 用 `<Tab>` / `<S-Tab>` 在占位符间跳转

**自定义 Snippet**：`~/.config/nvim/snippets/` 下的 JSON 文件会被自动扫描，
文件名去 `.json` 后缀即 filetype。当前内置 `org.json`（Org 表格 `<t`、
TODO 标题 `tdo`），详见 [orgmode.md](orgmode.md) 5.5 节。

**自动配对**：mini.pairs 提供括号、引号等的自动闭合。

### 6.7 Obsidian 笔记工作流

位置：`lua/plugins/obsidian.lua`

与 Obsidian 应用**共用同一个笔记库** `~/Documents/notes`，目录约定完全对齐：

| 项目 | 值 |
|------|-----|
| 新笔记 | `draft/`，以笔记标题为文件名（非法字符替换为 `-`），无标题时用时间戳 |
| 日记 | `dailynote/`，文件名格式 `YYYY-MM-DD` |
| 附件 / 图片 | `images/` |
| 链接风格 | 标准 Markdown 链接 `[text](path)` |
| frontmatter | 不自动生成，保留笔记原样 |
| 搜索面板 | fzf-lua |
| UI 渲染 | 关闭（交给 render-markdown） |

**快速插入笔记元素**（纯 Lua 实现，替代手工排版）：

- `<leader>mc` **Callout**：菜单选择 9 种类型
  （note / tip / important / warning / caution / question / example / bug / quote），
  再输入可选标题，自动插入 `> [!TYPE] 标题` 块并进入编辑位置
- `<leader>mt` **表格**：输入 `列x行`（如 `3x2`，支持 `x`、`X`、`×`、`*`），
  生成按中文显示宽度对齐的管道表格，占位文字可直接覆盖输入
- `<leader>mb` **复选框**：在 `- [ ]` / `- [x]` 间切换

> 注意：多数 `:Obsidian` 子命令需要在笔记库目录内使用；
> 先用 `<leader>oo` 或 `<leader>of` 进入笔记库即可。

### 6.8 CodeCompanion AI 助手

位置：`lua/plugins/codecompanion.lua`

**准备工作**：设置环境变量（写入 `~/.bashrc` 或 `~/.zshrc`）：

```bash
export DEEPSEEK_API_KEY="sk-xxxxxxxx"
```

**模型配置**（DeepSeek adapter）：

| 参数 | 值 |
|------|-----|
| 默认模型 | `deepseek-v4-flash`（`<leader>aP` 可切换 pro） |
| 思考模式 | 默认开启，聊天中显示思考内容（默认折叠） |
| reasoning_effort | `max` |
| temperature | 0.4 |
| 回复语言 | 简体中文 |

**界面**：聊天窗口为右侧垂直分栏，宽 45%、全高；显示 token 用量；
AI 修改代码时提供 diff 视图供确认。

**三种交互方式**：

1. **Chat**（`<leader>an` / `<leader>ac`）：多轮对话，支持 `/explain`、
   `/fix`、`/tests`、`/commit` 等 slash 命令；聊天 buffer 本质是 Markdown，
   可补全 LaTeX 公式
2. **Inline**（`<leader>ai`）：对当前位置或选区直接生成 / 改写代码，
   以 diff 形式确认后应用
3. **动作面板**（`<leader>aa`）：浏览全部预设动作的菜单

### 6.9 Markdown 实时渲染

位置：`lua/plugins/markdown.lua`

- 作用文件类型：`markdown` 与 `codecompanion`
- **普通 / 命令 / 终端模式渲染，插入模式显示原文**，编辑所见即所得不打架
- 光标所在行始终显示原始 Markdown（anti-conceal），方便修改语法
- 渲染项：标题图标（󰲡~󰲫，需 Nerd Font）、代码块（块宽 + 右边距）、
  圆角管道表格、LaTeX 公式
- sign column 图标全局关闭，保持界面干净

### 6.10 Org mode 任务管理

位置：`lua/plugins/orgmode.lua`

与 Obsidian 的**分工**：Obsidian 管 Markdown 知识笔记（`~/Documents/notes`），
Org mode 管任务与日程（`~/Documents/org`），两套文件互不干扰。

**目录约定**：

| 项目 | 值 |
|------|-----|
| Org 文件目录 | `~/Documents/org`（启动时自动创建） |
| Agenda 扫描范围 | 该目录下所有 `.org` 文件 |
| Capture 默认文件 | `~/Documents/org/refile.org` |

**首次启动**：插件自动克隆并用系统 C 编译器（cc / gcc）编译 Org 专用的
Treesitter parser（终端显示 "Tree-sitter grammar installed!"，仅需一次）。
Org 文件的高亮与折叠由插件自身的 ftplugin 管理，不占用
`treesitter.lua` 的 parser 列表，两者互不影响。

**快速上手**：

1. `<leader>oc` 打开 Capture 菜单，按 `t` 选 Task 模板，写完按 `<C-c>` 保存
   （内容存入 `refile.org`；想放弃按 `<leader>ok`）
2. 在任意 `.org` 文件中写 `* TODO 任务标题`，光标在标题上按 `cit`
   循环切换 TODO → DONE
3. `<leader>ois` / `<leader>oid` 给任务加 SCHEDULED（计划何时做）/
   DEADLINE（何时必须完成）日期
4. `<leader>oa` 打开 Agenda 菜单：按 `a` 看本周议程、`t` 看全部待办、
   `m` 按标签过滤、`s` 全文搜索
5. 忘记按键时在 org buffer 内按 `g?`，调出插件内置的完整按键帮助

**核心概念**：

- **TODO 状态**：默认 `TODO → DONE`；切换为 DONE 时自动在标题下记录完成时间
  （`org_log_done = 'time'`）
- **日期**：支持重复周期写法，如 `<2026-08-13 Thu +1w>` 表示每周重复；
  Agenda 依此把任务排到未来各天
- **折叠**：打开文件默认只显示顶层标题（`org_startup_folded = 'overview'`），
  `<Tab>` 展开当前标题，`<S-Tab>` 循环全文折叠级别
- **Refile**（`<leader>or`）：把当前标题移动到其他文件或标题下，
  配合 Capture 实现"先速记、后归档"的 GTD 流程
- **归档**（`<leader>o$`）：把已完成的子树移入同名 `_archive` 文件，
  保持主文件清爽
- **计时（Clocking）**：`<leader>oxi` 开始计时、`<leader>oxo` 结束，
  Agenda 里按 `R` 可看工时统计

**内置 LSP**（实验性）：插件自带的纯 Lua server，为 org buffer 提供链接、
标签、TODO 关键字等补全，经 blink.cmp 的 `lsp` 源自动生效；
不需要时删除 `orgmode.lua` 末尾的 `vim.lsp.enable("org")` 即可。

**导出**：org 文件内按 `<leader>oe` 打开导出菜单，HTML / PDF 等格式
需系统安装 pandoc 或 emacs，不用导出功能可忽略。

**官方文档**：Neovim 内执行 `:Org help`，或访问 <https://nvim-orgmode.github.io>。

> 日常使用教程（工作流 + 11 个使用场景）见 [orgmode.md](orgmode.md)。

### 6.11 which-key 快捷键提示

按下 `<leader>`（空格）或其他前缀后稍作停留，自动弹出可选快捷键面板。
所有自定义快捷键都写有 `desc` 描述，面板中显示的是可读说明而非原始命令。

---

## 七、自定义指南

| 需求 | 修改位置 |
|------|----------|
| 改编辑器选项 | `lua/core/options.lua` |
| 加 / 改通用快捷键 | `lua/core/keymaps.lua` |
| 加 / 改某插件的快捷键 | `lua/plugins/` 下对应文件（就近原则） |
| 加插件 | `lua/plugins/init.lua` 清单 + 新建 `lua/plugins/xxx.lua` 配置 |
| 加语言服务器 | `lua/plugins/lsp.lua` 中 `vim.lsp.enable("xxx")` |
| 加格式化工具 | `lua/core/keymaps.lua` 的 `formatters` 表 |
| 加 Treesitter 语言 | `lua/plugins/treesitter.lua` 的 `treesitter_parsers` 等三张表 |
| 换 AI 模型 / 参数 | `lua/plugins/codecompanion.lua` 的 `adapters` 段 |
| 换笔记库路径 | `lua/plugins/obsidian.lua` 顶部的 `notes_path` |
| 换 Org 文件目录 | `lua/plugins/orgmode.lua` 顶部的 `org_path` |

**示例：添加一个插件**

```lua
-- 1. lua/plugins/init.lua 的 vim.pack.add({...}) 中加入：
"https://github.com/lewis6991/gitsigns.nvim",

-- 2. 新建 lua/plugins/gitsigns.lua：
require("gitsigns").setup({})

-- 3. 在 lua/plugins/init.lua 底部加入：
require("plugins.gitsigns")
```

重启 Neovim 即自动安装。

---

## 八、常见问题（FAQ）

**Q1：Python 文件按 `<leader>cf` 提示 "Formatter not installed: ruff"？**
系统未安装 ruff，执行 `sudo pacman -S ruff` 即可。配置已做防护：
工具缺失或执行失败都不会破坏 buffer 内容。

**Q2：诊断信息（错误波浪线）怎么不见了？**
本配置**默认全局关闭诊断**（`vim.diagnostic.enable(false)`）。
按 `<leader>ul` 切换显示，`gl` 随时查看当前行详情。

**Q3：which-key 不需要 setup 吗？**
不需要。which-key v3 的 plugin 文件会在启动 500ms 后自动执行默认 setup，
因此插件清单里只声明、不配置也能正常工作。

**Q4：catppuccin 主题在插件清单里找不到？**
它不是 vim.pack 管理的插件，而是 Arch 的 neovim 包自带的
`/usr/share/nvim/runtime/colors/catppuccin.vim`。换系统或发行版时，
若提示主题不存在，需改用插件方式安装（如 `catppuccin/nvim`）。

**Q5：`<leader>ff` 搜索结果里找不到某些文件？**
fzf-lua 遵循 `.gitignore` 并受 `fd`/`rg` 规则影响（本机设置了
`FZF_DEFAULT_COMMAND=fd --type f --hidden --exclude .git`）。
被 git 忽略的文件默认不会出现，这是预期行为。

**Q6：终端里怎么滚动查看历史输出？**
按 `jk` 退出终端模式进入普通模式，即可用 `hjkl`/`gg`/`G` 滚动翻页，
按 `i` 回到输入。

**Q7：启动时看到 fcitx5 相关报错或没有输入法切换？**
确认 `fcitx5-remote` 在 PATH 中（`which fcitx5-remote`）。
未安装时该功能整体静默跳过，不影响其他功能。

**Q8：在 Org 文件里按 `<leader>oo` / `<leader>ot`，为什么不是 Obsidian 的功能？**
orgmode 的快捷键是 org buffer 的**局部映射**，会覆盖同名全局映射：
org 文件内 `<leader>oo` 是"打开链接 / 日期"，`<leader>ot` 是"修改标签"；
离开 org buffer 后即恢复为 Obsidian 的全局映射。这是预期行为，
两者按文件类型各司其职（`.md` 用 Obsidian，`.org` 用 orgmode）。

---

## 九、备份与回滚

重构前的原始单文件配置完整备份于：

```
~/.config/nvim-backup-20260811-193741/
```

如需整体回滚：

```bash
rm -rf ~/.config/nvim
cp -a ~/.config/nvim-backup-20260811-193741 ~/.config/nvim
```

确认新配置稳定使用后，可自行删除备份目录。

---

> 配置即文档：每个 Lua 文件顶部都有职责说明，关键分支均有中文注释，
> 配合本手册的"修改位置"表格即可快速定位任何功能。
