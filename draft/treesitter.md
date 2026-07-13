
```lua
vim.pack.add({
  "https://github.com/neovim-treesitter/treesitter-parser-registry",
  "https://github.com/neovim-treesitter/nvim-treesitter",
  "https://github.com/nvim-treesitter/nvim-treesitter-textobjects",
})
```

其中：

- `nvim-treesitter`：安装 parser 和 query
- `treesitter-parser-registry`：新的 parser 注册表
- `nvim-treesitter-textobjects`：函数、类、参数等语法对象操作

> `nvim-treesitter` 不应该延迟加载，你现在直接通过 `vim.pack.add()` 加载是合适的。

---

## 2. 推荐的完整 Treesitter 配置

将下面代码放到插件安装代码之后，例如放在 `render-markdown` 配置之前。

```lua
-- =========================================================
-- Treesitter
-- =========================================================

local treesitter = require("nvim-treesitter")

treesitter.setup({
  -- 默认安装目录已经足够，一般不需要修改
  install_dir = vim.fn.stdpath("data") .. "/site",
})

-- 常用语言 parser
local treesitter_parsers = {
  -- Neovim 配置
  "lua",
  "luadoc",
  "vim",
  "vimdoc",
  "query",

  -- C/C++
  "c",
  "cpp",

  -- Python
  "python",

  -- Shell
  "bash",

  -- Markdown
  "markdown",
  "markdown_inline",

  -- 配置文件
  "json",
  "jsonc",
  "yaml",
  "toml",

  -- Git
  "diff",
  "gitcommit",
  "gitignore",

  -- Web
  "html",
  "css",
  "javascript",
  "typescript",
  "tsx",

  -- 其他常用 parser
  "regex",
}

-- 异步安装缺少的 parser。
-- 已经安装的 parser 不会重复安装。
treesitter.install(treesitter_parsers)

-- CodeCompanion buffer 本质上是 Markdown
vim.treesitter.language.register("markdown", { "codecompanion" })

-- 需要启用 Treesitter 功能的实际 filetype
local treesitter_filetypes = {
  "lua",
  "vim",
  "vimdoc",

  "c",
  "cpp",
  "python",

  "sh",
  "bash",

  "markdown",
  "codecompanion",

  "json",
  "jsonc",
  "yaml",
  "toml",

  "diff",
  "gitcommit",

  "html",
  "css",
  "javascript",
  "javascriptreact",
  "typescript",
  "typescriptreact",
}

-- 适合启用 Treesitter 缩进的 filetype
-- Treesitter 缩进仍然属于实验功能，如果某种语言缩进异常，
-- 可以直接从这里删除。
local treesitter_indent_filetypes = {
  c = true,
  cpp = true,
  python = true,
  lua = true,
  sh = true,
  bash = true,
  json = true,
  jsonc = true,
  yaml = true,
  toml = true,
  html = true,
  css = true,
  javascript = true,
  javascriptreact = true,
  typescript = true,
  typescriptreact = true,
}

local treesitter_group =
  vim.api.nvim_create_augroup("UserTreesitter", { clear = true })

vim.api.nvim_create_autocmd("FileType", {
  group = treesitter_group,
  pattern = treesitter_filetypes,
  callback = function(args)
    -- 启用 Treesitter 高亮和语言注入
    local ok = pcall(vim.treesitter.start, args.buf)

    -- 第一次启动时 parser 可能仍在异步安装。
    -- 此时不继续设置，安装完成后重新打开文件即可。
    if not ok then
      return
    end

    -- Treesitter 折叠
    vim.wo.foldmethod = "expr"
    vim.wo.foldexpr = "v:lua.vim.treesitter.foldexpr()"

    -- 打开文件时默认不要折叠所有代码
    vim.wo.foldlevel = 99

    -- Treesitter 智能缩进
    if treesitter_indent_filetypes[vim.bo[args.buf].filetype] then
      vim.bo[args.buf].indentexpr =
        "v:lua.require'nvim-treesitter'.indentexpr()"
    end
  end,
})
```

### 为什么需要同时安装 `markdown` 和 `markdown_inline`

- `markdown`：负责标题、列表、代码块、引用块等结构
- `markdown_inline`：负责粗体、斜体、行内代码、链接等行内语法

你的 `render-markdown.nvim` 和 `CodeCompanion` 都会用到它们。

---

## 3. 配置 Treesitter Textobjects

Textobjects 是 Treesitter 最实用的功能之一。

例如：

- `vaf`：选择整个函数
- `vif`：选择函数内部
- `daf`：删除整个函数
- `yif`：复制函数内部
- `cia`：修改当前参数
- `]f`：跳转到下一个函数
- `<leader>sn`：将当前参数与下一个参数交换

添加下面配置：

```lua
-- =========================================================
-- Treesitter Textobjects
-- =========================================================

require("nvim-treesitter-textobjects").setup({
  select = {
    -- 当前光标不在目标内部时，自动向后寻找目标
    lookahead = true,

    selection_modes = {
      ["@parameter.outer"] = "v",
      ["@function.outer"] = "V",
      ["@class.outer"] = "V",
    },

    include_surrounding_whitespace = false,
  },

  move = {
    -- 跳转时记录到 jumplist，可以使用 <C-o> 返回
    set_jumps = true,
  },
})

local ts_select =
  require ("nvim-treesitter-textobjects. select")

local ts_move =
  require ("nvim-treesitter-textobjects. move")

local ts_swap =
  require ("nvim-treesitter-textobjects. swap")

-- ---------------------------------------------------------
-- 选择语法对象
-- x：可视模式
-- o：操作符等待模式，因此可以配合 d、c、y 使用
-- ---------------------------------------------------------

vim.keymap.set ({ "x", "o" }, "af", function ()
  ts_select. select_textobject ("@function. outer", "textobjects")
end, {
  desc = "Around function",
})

vim.keymap.set ({ "x", "o" }, "if", function ()
  ts_select. select_textobject ("@function. inner", "textobjects")
end, {
  desc = "Inside function",
})

vim.keymap.set ({ "x", "o" }, "ac", function ()
  ts_select. select_textobject ("@class. outer", "textobjects")
end, {
  desc = "Around class",
})

vim.keymap.set ({ "x", "o" }, "ic", function ()
  ts_select. select_textobject ("@class. inner", "textobjects")
end, {
  desc = "Inside class",
})

vim.keymap.set ({ "x", "o" }, "aa", function ()
  ts_select. select_textobject ("@parameter. outer", "textobjects")
end, {
  desc = "Around parameter",
})

vim.keymap.set ({ "x", "o" }, "ia", function ()
  ts_select. select_textobject ("@parameter. inner", "textobjects")
end, {
  desc = "Inside parameter",
})

vim.keymap.set ({ "x", "o" }, "al", function ()
  ts_select. select_textobject ("@loop. outer", "textobjects")
end, {
  desc = "Around loop",
})

vim.keymap.set ({ "x", "o" }, "il", function ()
  ts_select. select_textobject ("@loop. inner", "textobjects")
end, {
  desc = "Inside loop",
})

vim.keymap.set ({ "x", "o" }, "ai", function ()
  ts_select. select_textobject ("@conditional. outer", "textobjects")
end, {
  desc = "Around conditional",
})

vim.keymap.set ({ "x", "o" }, "ii", function ()
  ts_select. select_textobject ("@conditional. inner", "textobjects")
end, {
  desc = "Inside conditional",
})

-- ---------------------------------------------------------
-- 函数和类之间跳转
-- ---------------------------------------------------------

vim.keymap.set ({ "n", "x", "o" }, "]f", function ()
  ts_move. goto_next_start ("@function. outer", "textobjects")
end, {
  desc = "Next function start",
})

vim.keymap.set ({ "n", "x", "o" }, "[f", function ()
  ts_move. goto_previous_start ("@function. outer", "textobjects")
end, {
  desc = "Previous function start",
})

vim.keymap.set ({ "n", "x", "o" }, "]F", function ()
  ts_move. goto_next_end ("@function. outer", "textobjects")
end, {
  desc = "Next function end",
})

vim.keymap.set ({ "n", "x", "o" }, "[F", function ()
  ts_move. goto_previous_end ("@function. outer", "textobjects")
end, {
  desc = "Previous function end",
})

vim.keymap.set ({ "n", "x", "o" }, "]c", function ()
  ts_move. goto_next_start ("@class. outer", "textobjects")
end, {
  desc = "Next class start",
})

vim.keymap.set ({ "n", "x", "o" }, "[c", function ()
  ts_move. goto_previous_start ("@class. outer", "textobjects")
end, {
  desc = "Previous class start",
})

-- ---------------------------------------------------------
-- 交换函数参数
-- ---------------------------------------------------------

vim.keymap.set ("n", "<leader>sn", function ()
  ts_swap. swap_next ("@parameter. inner")
end, {
  desc = "Swap with next parameter",
})

vim.keymap.set ("n", "<leader>sp", function ()
  ts_swap. swap_previous ("@parameter. inner")
end, {
  desc = "Swap with previous parameter",
})
```

### 常用 Textobjects 示例

| 按键 | 功能 |
|---|---|
| `vaf` | 选择整个函数 |
| `vif` | 选择函数内部 |
| `daf` | 删除整个函数 |
| `cif` | 修改函数内部 |
| `yaf` | 复制整个函数 |
| `vac` | 选择整个类 |
| `daa` | 删除当前参数，包括分隔部分 |
| `cia` | 修改当前参数内容 |
| `dal` | 删除整个循环 |
| `dai` | 删除整个条件语句 |
| `]f` / `[f` | 下一个/上一个函数 |
| `]c` / `[c` | 下一个/上一个类 |
| `<leader>sn` | 参数向后交换 |
| `<leader>sp` | 参数向前交换 |

这里的 Textobjects 按键不会和你的 CodeCompanion `<leader>aa` 冲突，因为 `aa` 和 `<leader>aa` 是两个不同的映射。

---

## 4. Treesitter 增量选择

Neovim 0.12 已经内置 Treesitter 增量选择映射，不需要额外插件。

先按 `v` 进入可视模式，然后使用：

| 按键 | 功能 |
|---|---|
| `an` | 扩大到外层语法节点 |
| `in` | 缩小到内层语法节点 |
| `]n` | 选择下一个语法节点 |
| `[n` | 选择上一个语法节点 |

典型操作：

```text
v
an
an
an
```

每按一次 `an`，选择范围会从：

```text
变量 → 表达式 → 语句 → 代码块 → 函数
```

逐渐扩大。

---

## 5. 添加 Treesitter 调试快捷键

这几个快捷键在检查高亮、语法节点和编写 query 时很有用：

```lua
vim.keymap.set ("n", "<leader>ti", "<cmd>Inspect<cr>", {
  silent = true,
  desc = "Inspect Treesitter highlight",
})

vim.keymap.set ("n", "<leader>tt", "<cmd>InspectTree<cr>", {
  silent = true,
  desc = "Open Treesitter syntax tree",
})

vim.keymap.set ("n", "<leader>tq", "<cmd>EditQuery highlights<cr>", {
  silent = true,
  desc = "Edit Treesitter highlight query",
})
```

- `<leader>ti`：查看光标处使用了哪些 highlight capture
- `<leader>tt`：打开当前文件的完整语法树
- `<leader>tq`：查看或修改当前语言的高亮 query

---

## 6. 折叠快捷键

配置 Treesitter 折叠后，直接使用 Vim 内置快捷键：

| 按键 | 功能 |
|---|---|
| `za` | 切换当前折叠 |
| `zc` | 关闭当前折叠 |
| `zo` | 打开当前折叠 |
| `zM` | 关闭全部折叠 |
| `zR` | 打开全部折叠 |
| `zj` | 跳到下一个折叠 |
| `zk` | 跳到上一个折叠 |

如果想用更直观的快捷键，也可以添加：

```lua
vim.keymap.set ("n", "<leader>zz", "za", {
  remap = true,
  desc = "Toggle current fold",
})

vim.keymap.set ("n", "<leader>zM", "zM", {
  remap = true,
  desc = "Close all folds",
})

vim.keymap.set ("n", "<leader>zR", "zR", {
  remap = true,
  desc = "Open all folds",
})
```

---

## 7. WSL 安装依赖

新版本需要：

- C/C++ 编译器
- `curl`
- `tar`
- `tree-sitter` CLI，至少为 `0.26.1`

在 WSL 中可以执行：

```bash
sudo apt update
sudo apt install -y build-essential curl tar cargo
cargo install tree-sitter-cli --locked
```

检查版本：

```bash
tree-sitter --version
```

然后进入 Neovim 执行：

```vim
:TSRegistryUpdate
: TSInstall lua vim vimdoc c cpp python bash markdown markdown_inline json yaml toml
:TSUpdate
:TSStatus
: checkhealth nvim-treesitter
```

由于配置中的 `treesitter.install ()` 是异步安装，第一次启动后建议等待安装完成，然后重新打开文件或重启一次 Neovim。

以后执行 `vim.pack.update ()` 更新插件后，再执行一次：

```vim
:TSUpdate
```

确保 parser、query 与插件版本匹配。

---

## 8. 你的配置中还有一个潜在错误

你前面使用了：

```lua
vim. api. nvim_create_autocmd ("BufReadPost", {
  group = augroup,
```

但贴出的配置里没有定义 `augroup`。如果确实没有定义，配置会在这里报错并停止，后面的 Treesitter 等插件也不会执行。

在这个 autocmd 前面添加：

```lua
local augroup =
  vim. api. nvim_create_augroup ("UserConfig", { clear = true })
```

或者直接写成：

```lua
local restore_cursor_group =
  vim. api. nvim_create_augroup ("RestoreCursor", { clear = true })

vim. api. nvim_create_autocmd ("BufReadPost", {
  group = restore_cursor_group,
  desc = "Restore last cursor position",
  callback = function ()
    if vim. o. diff then
      return
    end

    local last_pos = vim. api. nvim_buf_get_mark (0, '"')
    local last_line = vim. api. nvim_buf_line_count (0)
    local row = last_pos[1]

    if row < 1 or row > last_line then
      return
    end

    pcall (vim. api. nvim_win_set_cursor, 0, last_pos)
  end,
})
```