#开发工具

这是一份关于 Zed 编辑器 **Vim Mode（Vim 模式）** 官方文档的全面、详尽的总结与讲解。为了方便阅读和记忆，我将内容划分为十个核心模块，确保**不遗漏任何一个细节、快捷键和配置项**。

---

### 一、 Zed Vim Mode 的设计理念与核心差异
Zed 的 Vim 模式并非 1:1 复刻传统的 Vim，而是将 Vim 的模态设计与 Zed 的现代编辑器特性（如 Tree-sitter 语法树、语义导航、多光标等）深度融合。其目标是提供开箱即用、流畅且高度可配置的体验。

**四大核心差异：**
1. **Motions (光标运动)**：利用 Zed 的**语义解析**按语言调整行为。例如在 Rust 中，`%` 可以匹配管道符 `|`；在 JavaScript 中，`w` 会将 `$` 视为单词字符。
2. **Visual block selections (可视块选择)**：使用 Zed 的**多光标系统**模拟。这使得块选择极其灵活：在块选择后插入内容会在所有行实时更新，且你可以随时添加或删除光标。
3. **Macros (宏录制)**：使用 Zed 原生的录制系统，因此可以录制更复杂的操作（包括自动补全等现代编辑器行为）。
4. **Search and replace (搜索与替换)**：使用 Zed 的搜索引擎，其**正则表达式语法**与传统 Vim 存在差异（详见后文）。

---

### 二、启用与禁用 Vim Mode
- **首次启用**：在 Zed 欢迎界面勾选启用复选框。
- **随时切换**：打开命令面板（`Cmd/Ctrl + Shift + P`），执行 `workspace: toggle vim mode`。
- **底层设置**：该命令实际上是切换用户配置文件（`settings.json`）中的 `"vim_mode": true`。

---

### 三、 Zed 特有功能与快捷键
Zed 借助语言服务器（LSP）、Git、Tree-sitter 等现代技术，提供了许多 Vim 原生没有的快捷键。

#### 1. Language Server (语言服务器)
*注：部分快捷键与多光标快捷键重合，具体生效取决于当前上下文。*
| 功能 | 默认快捷键 |
| :--- | :--- |
| 跳转定义 (Go to definition) | `g d` |
| 跳转声明 (Go to declaration) | `g D` |
| 跳转类型定义 (Go to type definition) | `g y` |
| 跳转实现 (Go to implementation) | `g I` |
| 重命名 (Rename / Change definition) | `c d` |
| 查找当前词的所有引用 (All references) | `g A` |
| 查找当前文件符号 (Symbol in file) | `g s` |
| 查找全项目符号 (Symbol in project) | `g S` |
| 下一个 / 上一个诊断 (错误/警告) | `g ]` 或 `] d` / `g [` 或 `[ d` |
| 悬浮显示内联错误 (Hover) | `g h` |
| 打开代码操作菜单 (Code actions) | `g .` |

#### 2. Git 版本控制
| 功能                          | 默认快捷键         |
| :-------------------------- | :------------ |
| 下一个 / 上一个 Git 变更            | `] c` / `[ c` |
| 展开 Diff Hunk (差异块)          | `d o`         |
| 切换 Staged (暂存) 状态           | `d O`         |
| Stage 并跳转下一个 (在 Diff 视图中)   | `d u`         |
| Unstage 并跳转下一个 (在 Diff 视图中) | `d U`         |
| 恢复变更 (Restore / Revert)     | `d p`         |

#### 3. Tree-sitter (语法树解析)
Tree-sitter 让 Zed 能够理解代码结构，从而提供精准的跳转和文本对象。
**运动跳转：**
- **方法**：`] m` / `[ m` (下/上一个方法)，`] M` / `[ M` (方法结尾)
- **Section (区块)**：`] ]` / `[ [` (下/上一个区块)，`] [` / `[ ]` (区块结尾)
- **注释**：`] /`, `] *` / `[ /`, `[ *` (下/上一个注释)
- **节点选择**：`[ x` (选择更大的语法节点)，`] x` (选择更小的语法节点)

**Text Objects (文本对象)：**
*可作为 `d`, `c`, `y` 等操作符的目标（如 `daf` 删除整个函数）。*
| 目标 | Around (包含边界) | Inside (仅内部) |
| :--- | :--- | :--- |
| 类/定义 (Class) | `a c` | `i c` |
| 函数/方法 (Function) | `a f` | `i f` |
| 注释 (Comment) | - | `g c` (注：`[/` 跳转目标与此相同) |
| 参数/列表项 (Argument) | `a a` (含尾随逗号) | `i a` (不含尾随逗号) |
| HTML 类标签 (Tag) | `a t` | `i t` |
| 缩进级别 (Indent) | `a I` (当前及上下各一行)<br>`a i` (当前及上一行) | `i i` (仅当前缩进) |
> **细节补充**：`[m` 的边界等同于 `af`；`[[` 的边界等同于 `ac`（若无类则包含函数）。这些定义依赖于语言，可通过扩展的 `[textobjects.scm]` 文件增加支持。参数和标签在 Tree-sitter 层面运行，目前不可按语言自定义。

#### 4. Multi cursor (多光标管理)
| 功能 | 默认快捷键 |
| :--- | :--- |
| 选择当前词的下一个 / 上一个副本并添加光标 | `g l` / `g L` |
| 在当前视觉选择的每行行尾 / 行首添加光标 | `g A` / `g I` |
| 为当前词的**每个**副本添加视觉选择 | `g a` |
| 跳过最新单词选择，并添加下一个 / 上一个 | `g >` / `g <` |

#### 5. Pane management (面板与窗口管理)
| 功能 | 默认快捷键 |
| :--- | :--- |
| 全项目搜索 | `g /` |
| 打开当前搜索摘录 (Excerpt) | `g <space>` |
| 在拆分面板中打开当前搜索摘录 | `<ctrl-w> <space>` |
| 在拆分面板中跳转定义 / 类型定义 | `<ctrl-w> g d` / `<ctrl-w> g D` |

#### 6. In insert mode (插入模式专属)
无需退出插入模式即可调用 Zed 的现代 AI 和补全功能：
| 功能 | 默认快捷键 |
| :--- | :--- |
| 打开补全菜单 | `ctrl-x ctrl-o` |
| 请求 GitHub Copilot 建议 | `ctrl-x ctrl-c` |
| 打开内联 AI 助手 | `ctrl-x ctrl-a` |
| 打开代码操作菜单 | `ctrl-x ctrl-l` |
| 隐藏所有建议 | `ctrl-x ctrl-z` |

---

### 四、内置插件功能与高级文本对象
Zed 原生内置了许多 Vim 生态中需要插件才能实现的功能：
- **Surround (环绕文本)**：`ys` (添加), `cs` (修改), `ds` (删除)。
- **Comment (注释)**：可视模式 `gc`，普通模式 `gcc`。
- **netrw (项目面板)**：支持 `hjkl` 导航，`o` 打开文件，`t` 在新标签页打开。
- **ReplaceWithRegister**：使用 `gR` 用寄存器内容替换文本。
- **Indent wise (缩进导航)**：使用 `[-`, `]-`, `[+`, `]+`, `[=`, `]=` 按缩进深度跳转。

#### 💡 核心特性：Any Bracket Functionality (任意括号/引号选择)
Zed 提供了两套强大的文本对象策略（**默认未启用，需在 keymap 中配置**），用于选择被任意引号或括号包围的文本。
**支持的字符**：
- **Quotes**: `'`, `"`, `` ` ``
- **Brackets**: `()`, `[]`, `{}`, `<>`

| 策略 | 行为特点 | 适用场景 |
| :--- | :--- | :--- |
| **AnyQuotes / AnyBrackets**<br>*(传统 Vim 行为)* | 优先寻找**最内层**匹配；找不到则回退到当前行；纯字符匹配（可能误判如 `=>` 中的 `>` 为闭合括号）。 | 喜欢传统 Vim 行为、需要严格匹配最内层。 |
| **MiniQuotes / MiniBrackets**<br>*(mini. ai 插件行为)* | 优先在**当前行**搜索并向外扩展；结合 Tree-sitter **语法感知**（能区分箭头函数 `=>` 和真正的闭合 `>`）。 | 喜欢上下文感知、优先当前行搜索。 |

**配置示例**（在 `keymap.json` 中添加）：
```json
{
  "context": "vim_operator == a || vim_operator == i || vim_operator == cs",
  "bindings": {
    "q": "vim::AnyQuotes", "b": "vim::AnyBrackets", // 传统行为
    "Q": "vim::MiniQuotes", "B": "vim::MiniBrackets" // mini.ai 行为
  }
}
```
*配置后，你可以使用 `cib` (传统修改括号内), `ciB` (智能修改括号内), `ciq`, `ciQ` 等命令。*

---

### 五、命令面板 (Command Palette)
按 `:` 打开命令面板。Zed 支持 Vim 命令别名，但**目前不支持带参数的命令**。

#### 1. 文件与窗口管理
- `:w[rite][!]` 保存 / `:wq[!]` 保存并关闭 / `:q[uit][!]` 关闭 / `:x[it][!]` 关闭
- `:wa[ll][!]` 保存所有 / `:wqa[ll][!]` 保存并关闭所有 / `:qa[ll][!]` 关闭所有
- `:up[date]` 保存 / `:cq` 完全退出 Zed 所有实例
- `:bd[elete][!]` 在所有面板中关闭当前文件
- `:vs[plit]` 垂直拆分 / `:sp[lit]` 水平拆分
- `:new` / `:vne[w]` 新建文件并拆分
- `:tabedit` / `:tabnew` 新标签页打开 / `:tabn[ext]` / `:tabp[rev]` / `:tabc[lose]`
- `:ls` 显示所有 buffers

#### 2. Ex Commands (打开各类面板)
`:Explore` (项目), `:Collab` (协作), `:Chat` (聊天), `:AI` (AI 面板), `:Git`, `:Debug`, `:Notif` (通知), `:feedback`, `:clist` (诊断), `:term` (终端), `:Ext` (扩展)。

#### 3. 诊断与 Git
- **诊断**：`:cn[ext]` / `:ln[ext]` (下一个), `:cp[rev]` / `:lp[rev]` (上一个), `:cc` / `:ll` (打开错误页)。
- **Git**：`:diff` (查看光标处 diff), `:rev` (恢复光标处 diff)。

#### 4. 跳转、替换与编辑
- **跳转**：`:<number>` (行号), `:$` (文件末尾), `:/foo` 和 `:?foo` (向下/向上搜索 foo)。
- **替换**：`:[range]s/foo/bar/[g]`。*(注意：Zed 默认只替换行内**第一个**匹配项，必须加 `g` 才会替换所有。)*
- **编辑**：`:j` (合并行), `:d` (删除行), `:s [i]` (排序，i 为忽略大小写), `:y` (复制行)。

#### 5. Set (本地缓冲区设置)
`:se[t] [no]wrap` (自动换行), `:se[t] [no]nu` (绝对行号), `:se[t] [no]rnu` (相对行号), `:se[t] [no]ic` (忽略大小写搜索)。

---

### 六、命令别名 (Command Mnemonics)
你可以为 Zed 原生命令设置简短的“别名”，在 `:` 面板中快速调用。
**配置示例** (`settings.json`)：
```json
{
  "command_aliases": {
    "zlog": "zed::OpenLog",
    "newf": "workspace::NewFile",
    "crp": "workspace::CopyRelativePath",
    "reveal": "editor::RevealInFileManager"
  }
}
```
*配置后，输入 `:zlog` 即可打开日志，输入 `:crp` 即可复制相对路径。*

---

### 七、自定义快捷键与上下文 (Contexts)
Zed 的快捷键系统基于 **Context (上下文)**，只有当上下文匹配时，快捷键才会生效。

#### 1. Context 机制与嵌套
- **嵌套规则**：`Workspace` (全局) -> `Pane` (面板) -> `Editor` (编辑器)。
- **Vim 专属 Context**：Vim 状态被定义在 `Editor` 层级。因此，**不能**使用 `Workspace && vim_mode == normal`（永远不会匹配），必须使用 `Editor && vim_mode == normal`。
- 支持逻辑运算符 `&&` (与), `||` (或)。

#### 2. Vim 专属 Context 变量
- `VimControl`：等同于 `normal || visual || operator`。
- `vim_mode`：可取值 `normal`, `visual`, `insert`, `replace`, `waiting` (等待按键如 `f`), `operator` (等待操作符如 `d`)。
- `vim_operator`：当处于 operator 模式时，记录当前操作符（如按下 `d` 后，值为 `d`）。

#### 3. 推荐的可选快捷键配置 (Optional Key Bindings)
你可以将以下配置加入 `keymap.json` 来解锁高级玩法：
- **Dock 面板导航**：在 `Dock` 上下文绑定 `ctrl-w h/j/k/l` 以在终端、项目面板间跳转。
- **Subword motion (子词运动)**：让 `w/b/e` 支持 `camelCase` 和 `snake_case` 单词内部跳转。
- **Visual 模式 Surround**：将 `shift-s` 绑定为 `vim::PushAddSurrounds`（替代默认的 substitute 擦除功能）。
- **光标跨行包裹 (whichwrap)**：让 `h/l` 和左右方向键在行尾/行首自动换行。
- **Sneak 运动**：将 `s` / `shift-s` 绑定为双字符跳转（替代 substitute）。
- **Helix 跳转**：绑定 `g w` 显示可视单词跳转标签。
- **vim-exchange**：在 visual 模式将 `shift-x` 绑定为文本交换功能。

#### 4. 恢复系统默认快捷键 (Linux/Windows 用户必看)
Vim 模式会覆盖 `Ctrl+C/V/F` 等系统快捷键。在 `Editor && !menu` 上下文中重新绑定它们即可恢复：
```json
"ctrl-f": "buffer_search::Deploy", "ctrl-c": "editor::Copy", "ctrl-v": "editor::Paste",
"ctrl-x": "editor::Cut", "ctrl-a": "editor::SelectAll", "ctrl-z": "editor::Undo" // 等等
```

---

### 八、 Vim 模式专属设置 (Vim Settings)
在 `settings.json` 的 `"vim": { ... }` 对象中配置：
| 属性 | 说明 | 默认值 |
| :--- | :--- | :--- |
| `default_mode` | 启动时的默认模式 (如 `"normal"`, `"insert"`, `"helix_normal"`) | `"normal"` |
| `use_system_clipboard` | 剪贴板策略：`"always"` (总是), `"never"` (从不), `"on_yank"` (仅 yank 时) | `"always"` |
| `use_smartcase_find` | `f` / `t` 查找时，若目标字母为小写则忽略大小写 | `false` |
| `use_regex_search` | 搜索时默认启用正则表达式 | `true` |
| `gdefault` | 若为 true，`:s` 命令默认全局替换（加 `g` 反而只替换第一个） | `false` |
| `toggle_relative_line_numbers` | 普通模式相对行号，插入模式绝对行号 | `false` |
| `highlight_on_yank_duration` | Yank 高亮动画持续时间（毫秒），设为 `0` 禁用 | `200` |
| `custom_digraphs` | 自定义双字符输入（如输入 `ctrl-k f z` 插入 🧟‍♀️） | `{}` |

---

### 九、提升 Vim 体验的 Zed 核心设置
在 `settings.json` 根目录配置以下 Zed 原生属性，可大幅优化 Vim 手感：
```json
{
  "cursor_blink": false,                 // 禁用光标闪烁 (Vim 用户通常不喜欢闪烁)
  "relative_line_numbers": "enabled",    // 启用相对行号 (方便 hjkl 和数字跳转)
  "scrollbar": { "show": "never" },      // 隐藏滚动条
  "scroll_beyond_last_line": "off",      // 禁止滚动超出文件最后一行
  "vertical_scroll_margin": 0,           // 允许光标移动到屏幕最顶端/最底端 (不强制保留边距)
  "gutter": { "line_numbers": false }    // 如果你想彻底隐藏行号
}
```

---

### 十、正则表达式差异 (Regex Differences)
Zed 使用的是 Rust 的 `regex` 引擎，与 Vim 的正则语法有**四大核心差异**：

1. **捕获组 (Capture Groups)**：
   - **Vim**：使用 `\(` 和 `\)` 表示捕获组，`(` 和 `)` 表示字面量括号。
   - **Zed**：使用 `(` 和 `)` 表示捕获组，字面量括号必须转义为 `\(` 和 `\)`。
2. **匹配引用 (Matches)**：
   - **Vim**：使用 `\1`, `\2` 引用捕获组，`\0` 引用全匹配。
   - **Zed**：使用 `$1`, `$2` 引用捕获组，`$0` 引用全匹配。
3. **全局选项 (Global Option)**：
   - **Vim**：默认只替换行内第一个匹配项，需加 `/g` 全局替换。
   - **Zed**：**默认就是全局匹配**。
4. **大小写敏感 (Case Sensitivity)**：
   - **Vim**：在末尾加 `/i`。
   - **Zed**：在模式开头加 `(?i)`，或者在搜索面板按 `Alt-C` 切换。

> **💡 贴心细节**：为了帮助你过渡，当你在命令面板使用 Vim 风格的替换命令（如 `:%s/\(a\)/\1/`）时，Zed 的命令面板会**自动修正**括号和引用语法，将其转换为 Zed 能识别的格式。

---
**总结**：Zed 的 Vim Mode 是一个“披着 Vim 外衣的现代化编辑引擎”。它保留了 Vim 肌肉记忆的核心（模态、操作符、文本对象），但在底层用 Tree-sitter 和多光标替换了 Vim 老旧的文本解析机制。通过合理配置 `keymap.json` 和 `settings.json`，你可以获得远超传统 Vim/Neovim 的流畅开发体验。