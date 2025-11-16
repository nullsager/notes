
> [!important] 重要
> 想创建终端 2，就用 `2ToggleTerm`, 创建终端 3 就用 `3ToggleTerm`, vertical 下会直接同时显示。
> 如果想删除，在终端下按 `<C-d>` 即可（注意是 normal 模式），或者进入 normal 模式按 `:wq`
> 

### 安装

```lua
{
	{
	  'akinsho/toggleterm.nvim', version = "*",
	  opts = {--[[ 你想更改的选项放在这里 ]]}
	}
}
```

> [!important]
> 此插件必须通过使用 `require("toggleterm").setup{}` 显式启用。

# 设置项

```lua
require("toggleterm").setup{
  -- size 可以是一个数字或函数，该函数接收当前终端作为参数
  size = 20 | function(term)
	if term.direction == "horizontal" then
	  return 15
	elseif term.direction == "vertical" then
	  return vim.o.columns * 0.4
	end
  end,
  open_mapping = [[<c-\>]], -- 或者 { [[<c-\>]], [[<c-¥>]] } 如果您也使用日文键盘。
  on_create = fun(t: Terminal), -- 终端首次创建时运行的函数
  on_open = fun(t: Terminal), -- 终端打开时运行的函数
  on_close = fun(t: Terminal), -- 终端关闭时运行的函数
  on_stdout = fun(t: Terminal, job: number, data: string[], name: string) -- 处理 stdout 输出的回调函数
  on_stderr = fun(t: Terminal, job: number, data: string[], name: string) -- 处理 stderr 输出的回调函数
  on_exit = fun(t: Terminal, job: number, exit_code: number, name: string) -- 终端进程退出时运行的函数
  hide_numbers = true, -- 在 toggleterm 缓冲区中隐藏编号列
  shade_filetypes = {},
  autochdir = false, -- 当 neovim 更改其当前工作目录时，终端将在下次打开时更改自己的目录
  highlights = {
	-- 映射到高亮组名称及其值表的亮点
	-- 注意：这仅是值的子集，放置在此处的任何组都将为终端窗口分割设置
	Normal = {
	  guibg = "<VALUE-HERE>",
	},
	NormalFloat = {
	  link = 'Normal'
	},
	FloatBorder = {
	  guifg = "<VALUE-HERE>",
	  guibg = "<VALUE-HERE>",
	},
  },
  shade_terminals = true, -- 注意：此选项优先于指定的高亮，因此如果您指定了 Normal 高亮，应将其设置为 false
  shading_factor = '<number>', -- 用于减淡终端深色背景的百分比，默认值: -30
  shading_ratio = '<number>', -- 用于浅色/深色终端背景的减淡因子比率，默认值: -3
  start_in_insert = true,
  insert_mappings = true, -- 打开映射是否适用于插入模式
  terminal_mappings = true, -- 打开映射是否适用于已打开的终端
  persist_size = true,
  persist_mode = true, -- 如果设置为 true（默认），将记住先前的终端模式
  direction = 'vertical' 'horizontal' 'tab'
'float', -- 方向
  close_on_exit = true, -- 进程退出时关闭终端窗口
  clear_env = false, -- 仅使用来自 env 的环境变量，传递给 jobstart()
   -- 更改默认 shell。可以是字符串或返回字符串的函数
  shell = vim.o.shell,
  auto_scroll = true, -- 终端输出时自动滚动到底部
  -- 仅当 direction 设置为 'float' 时，此字段才相关
  float_opts = {
	-- border 键几乎与 'nvim_open_win' 相同
	-- 有关边框的详细信息请参阅 :h nvim_open_win
	-- 但 'curved' 边框是一种自定义边框类型
	-- 并非原生支持，但在此插件中实现。
	border = 'single' 'double' 'shadow' 'curved'
... win open 支持的其他选项
	-- 像 size, width, height, row, 和 col 可以是数字或函数，该函数接收当前终端作为参数
	width = <value>,
	height = <value>,
	row = <value>,
	col = <value>,
	winblend = 3,
	zindex = <value>,
	title_pos = 'left' 'center'
'right', -- 浮动窗口标题的位置
  },
  winbar = {
	enabled = false,
	name_formatter = function(term) --  term: Terminal
	  return term.name
	end
  },
  responsiveness = {
	-- 以 vim.o.columns 表示的断点，低于此值时终端将开始堆叠在一起
	-- 而不是彼此相邻
	-- 默认值 = 0 表示此功能关闭
	horizontal_breakpoint = 135,
  }
}
```

> [!warning] Warning
> 不要直接复制粘贴！

### 用法 :

```vim
:ToggleTerm size=40 dir=~/Desktop direction=horizontal name=desktop
```

> [!tip]
> size、dir、direction 和 name 都是可选参数

```
ToggleTermToggleAll
```

> [!summary]
> 此命令允许您一次性打开所有先前切换过的终端，或一次性关闭所有打开的终端。

```
TermExec
```

> [!summary] 
> 此命令允许您使用特定操作打开终端。

> [!example] 
> ```
> 2TermExec cmd="git status" dir=~/<my-repo-path> 
> ```
> 将在终端 2 中运行 git status。注意 cmd 参数必须加引号
> 
> 如果 dir 参数包含空格，也可以_选择性地_加引号。 

```vim
TermSelect
```

> [!summary]
> 此命令使用 `vim.ui.select` 允许用户选择要打开的终端，或者如果它已经打开则聚焦它。如果您有很多终端并且想要打开一个特定的终端，这会很有用。

向终端发送行：

```
:ToggleTermSendCurrentLine <T_ID> 发送您光标所在的整行

:ToggleTermSendVisualLines <T_ID> 发送您视觉选择中的所有（整）行

:ToggleTermSendVisualSelection <T_ID> 仅发送视觉选择的文本（这可以是一个文本块或单行中的选择）
```

> [!info]
> T_ID 是一个可选的终端 ID 参数，它定义了我们应该将行发送到哪里。如果未提供该参数，则默认为 第一个终端

或者，为了更精细的控制和在映射中使用，在 lua 中：

```lua
local trim_spaces = true
vim.keymap.set("v", "<space>s", function()
	require("toggleterm").send_lines_to_terminal("single_line", trim_spaces, { args = vim.v.count })
end)
	-- 用以下内容替换为其他两个选项
	-- require("toggleterm").send_lines_to_terminal("visual_lines", trim_spaces, { args = vim.v.count })
	-- require("toggleterm").send_lines_to_terminal("visual_selection", trim_spaces, { args = vim.v.count })

-- 用作操作符映射：
-- 将 motion 发送到终端
vim.keymap.set("n", [[<leader><c-\>]], function()
  set_opfunc(function(motion_type)
	require("toggleterm").send_lines_to_terminal(motion_type, false, { args = vim.v.count })
  end)
  vim.api.nvim_feedkeys("g@", "n", false)
end)
-- 双击命令将行发送到终端
vim.keymap.set("n", [[<leader><c-\><c-\>]], function()
  set_opfunc(function(motion_type)
	require("toggleterm").send_lines_to_terminal(motion_type, false, { args = vim.v.count })
  end)
  vim.api.nvim_feedkeys("g@_", "n", false)
end)
-- 发送整个文件
vim.keymap.set("n", [[<leader><leader><c-\>]], function()
  set_opfunc(function(motion_type)
	require("toggleterm").send_lines_to_terminal(motion_type, false, { args = vim.v.count })
  end)
  vim.api.nvim_feedkeys("ggg@G''", "n", false)
end)
```

```
ToggleTERMSetName
```

> [!Summary]
> 此函数允许为终端设置显示名称。此名称主要用于窗口栏 (winbar) 内部，并且可以作为一种更具描述性的方式来记住哪个终端是用于什么的。

> [!tip]
> 可以将其映射到一个键并使用计数调用它，这将提示您为匹配 ID 的终端输入名称。或者，您可以仅使用名称调用它，例如：`:ToggleTermSetName work<CR>`，这将提示您它应应用于哪个终端。最后，您可以在没有任何参数的情况下调用它，它将提示您它应应用于哪个终端，然后提示您要使用的名称。


添加映射可以使切换后更容易进出终端，同时保持其打开状态：


```
function _G.set_terminal_keymaps()
  local opts = {buffer = 0}
  vim.keymap.set('t', '<esc>', [[<C-\><C-n>]], opts)
  vim.keymap.set('t', 'jk', [[<C-\><C-n>]], opts)
  vim.keymap.set('t', '<C-h>', [[<Cmd>wincmd h<CR>]], opts)
  vim.keymap.set('t', '<C-j>', [[<Cmd>wincmd j<CR>]], opts)
  vim.keymap.set('t', '<C-k>', [[<Cmd>wincmd k<CR>]], opts)
  vim.keymap.set('t', '<C-l>', [[<Cmd>wincmd l<CR>]], opts)
  vim.keymap.set('t', '<C-w>', [[<C-\><C-n><C-w>]], opts)
end

-- 如果您只希望这些映射用于 toggle term，请使用 term://toggleterm# 代替
vim.cmd('autocmd! TermOpen term://* lua set_terminal_keymaps()')
```

Toggleterm 还公开了 Terminal 类，以便可用于创建自定义终端来显示终端 UI，如 lazygit、htop 等。

每个终端可以接受以下参数：

```lua
Terminal:new {
  cmd = string -- 创建终端时执行的命令，例如 'top'
  display_name = string -- 终端的名称
  direction = string -- 终端的布局，与主配置选项相同
  dir = string -- 终端的目录
  close_on_exit = bool -- 进程退出时关闭终端窗口
  highlights = table -- 带有高亮的表
  env = table -- 传递给 jobstart() 的包含环境变量的键:值表
  clear_env = bool -- 仅使用来自 env 的环境变量，传递给 jobstart()
  on_open = fun(t: Terminal) -- 终端打开时运行的函数
  on_close = fun(t: Terminal) -- 终端关闭时运行的函数
  auto_scroll = boolean -- 终端输出时自动滚动到底部
  -- 用于处理输出的回调函数
  on_stdout = fun(t: Terminal, job: number, data: string[], name: string) -- 处理 stdout 输出的回调函数
  on_stderr = fun(t: Terminal, job: number, data: string[], name: string) -- 处理 stderr 输出的回调函数
  on_exit = fun(t: Terminal, job: number, exit_code: number, name: string) -- 终端进程退出时运行的函数
}
```

如果您想生成一个不运行任何命令的自定义终端，可以省略 cmd 选项。

自定义终端用法：

```lua
    local Terminal  = require('toggleterm.terminal').Terminal
    local lazygit = Terminal:new({ cmd = "lazygit", hidden = true })
    
    function _lazygit_toggle()
      lazygit:toggle()
    end
    
    vim.api.nvim_set_keymap("n", "<leader>g", "<cmd>lua _lazygit_toggle()<CR>", {noremap = true, silent = true})
```

这将创建一个新终端，但指定的命令不会立即运行。该命令将在终端打开时运行。或者，可以使用 term:spawn() 在后台缓冲区中启动命令而不立即打开终端窗口。如果 hidden 键设置为 true，则此终端不会被普通的 toggleterm 命令（如 :ToggleTerm 或打开映射）切换。它只能使用返回的终端对象打开和关闭。可以如上例所示设置用于切换终端的映射。

或者，可以使用计数指定终端，该计数是可用于触发此特定终端的数字。然后可以使用当前计数触发它，例如 `:5ToggleTerm<CR>`

```lua
local lazygit = Terminal:new({ cmd = "lazygit", count = 5 })
```

您还可以为终端设置自定义布局：


```lua
local lazygit = Terminal:new({
  cmd = "lazygit",
  dir = "git_dir",
  direction = "float",
  float_opts = {
	border = "double",
  },
  -- 打开终端时运行的函数
  on_open = function(term)
	vim.cmd("startinsert!")
	vim.api.nvim_buf_set_keymap(term.bufnr, "n", "q", "<cmd>close<CR>", {noremap = true, silent = true})
  end,
  -- 关闭终端时运行的函数
  on_close = function(term)
	vim.cmd("startinsert!")
  end,
})

function _lazygit_toggle()
  lazygit:toggle()
end

vim.api.nvim_set_keymap("n", "<leader>g", "<cmd>lua _lazygit_toggle()<CR>", {noremap = true, silent = true})
```

状态栏：

为了区分每个终端，您可以在状态栏中使用终端缓冲区变量 `b:toggle_number`

```lua
" 这是伪代码
let statusline .= '%{&ft == "toggleterm" ? "terminal (".b:toggle_number.")" : ""}'
```

```lua
command! -count=1 TermGitPush  lua require'toggleterm'.exec("git push",    <count>, 12)
command! -count=1 TermGitPushF lua require'toggleterm'.exec("git push -f", <count>, 12)
```


```
return {
  "akinsho/toggleterm.nvim",
  version = "*",
  config = function()
    require("toggleterm").setup({
      size = function(term)
        if term.direction == "horizontal" then
          return vim.o.lines * 0.4
        elseif term.direction == "vertical" then
          return vim.o.columns * 0.5
        end
      end,
      open_mapping = [[<c-\>]],
      persist_size = true,
      persist_mode = true,
      direction = "float", --'vertical' | 'horizontal' | 'tab' | 'float',
      close_on_exit = true, -- close the terminal window when the process exits
      clear_env = false, -- use only environmental variables from `env`, passed to jobstart()
      shell = vim.o.shell,
      float_opts = {
        border = "curved", -- 'single' | 'double' | 'shadow' | 'curved' | ... other options supported by win open
        winblend = 3,
      },
      winbar = {
        enabled = false,
        name_formatter = function(term) --  term: Terminal
          return term.name
        end,
      },
      responsiveness = {
        -- breakpoint in terms of `vim.o.columns` at which terminals will start to stack on top of each other
        -- instead of next to each other
        -- default = 0 which means the feature is turned off
        horizontal_breakpoint = 135,
      },
    })

    vim.keymap.set(
      "n",
      "<leader>tf",
      ":ToggleTerm direction=float<CR>",
      { noremap = true, silent = true, desc = "浮动终端" }
    )
    vim.keymap.set(
      "n",
      "<leader>tv",
      ":ToggleTerm direction=vertical<CR>",
      { noremap = true, silent = true, desc = "右终端" }
    )
    vim.keymap.set(
      "n",
      "<leader>th",
      ":ToggleTerm direction=horizontal<CR>",
      { noremap = true, silent = true, desc = "下终端" }
    )
    vim.keymap.set("n", "<leader>tt", ":TermSelect<CR>", { noremap = true, silent = true, desc = "切换终端" })

    function _G.set_terminal_keymaps()
      local opts = { buffer = 0 }
      vim.keymap.set("t", "<esc>", [[<C-\><C-n>]], opts)
      vim.keymap.set("t", "jk", [[<C-\><C-n>]], opts)
      vim.keymap.set("t", "<C-h>", [[<Cmd>wincmd h<CR>]], opts)
      vim.keymap.set("t", "<C-j>", [[<Cmd>wincmd j<CR>]], opts)
      vim.keymap.set("t", "<C-k>", [[<Cmd>wincmd k<CR>]], opts)
      vim.keymap.set("t", "<C-l>", [[<Cmd>wincmd l<CR>]], opts)
    end

    -- 如果您只希望这些映射用于 toggle term，请使用 term://toggleterm# 代替
    vim.cmd("autocmd! TermOpen term://* lua set_terminal_keymaps()")
  end,
}

```