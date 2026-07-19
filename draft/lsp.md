#linux 

从 **Neovim 0.10+** 开始，官方为原生 LSP 和诊断（Diagnostics）引入了一套全新的、无需任何配置即可开箱即用的**默认全局快捷键**。这省去了以前大家必须在配置里写一大堆 `buf_set_keymap` 的麻烦。

“现代 Neovim” 原生自带的 LSP 相关核心快捷键如下：

### 1. LSP 代码核心操作

新版的快捷键大多以 `gr`（Go to/Do with References 或类似语义）开头：

* **`gra`**：选择当前位置可用的 **代码操作（Code Action）**。（*对应旧版的 `<leader>ca*`）
* **`grn`**：重命名当前符号 **（Rename）**。（*对应旧版的 `<leader>rn*`）
* **`grr`**：列出当前符号的所有 **引用（References）**，并在 Quickfix 窗口展示。（*对应旧版的 `gr*`）
* **`gri`**：列出当前符号的所有 **实现（Implementation）**。
* **`grt`**：跳转到 **类型定义（Type Definition）**。
* **`K`**：在浮动窗口中显示当前符号的 **悬浮提示（Hover）**。
* **`Ctrl + s`**（插入模式下）：显示 **签名帮助（Signature Help）**。

### 2. 代码跳转与格式化

这些功能利用了 Neovim 内置的 `tagfunc` 和 `formatexpr`：

* **`Ctrl + ]`**：跳转到符号的 **定义（Definition）**。（*相当于旧版的 `gd*`）
* **`Ctrl + t`**：从定义中 **跳回** 上一级。
* **`gq`**（可在普通模式或可视模式下组合使用）：使用 LSP **格式化代码（Format）**。

### 3. 代码诊断（Diagnostics / 报错与警告）

* **`[d`**：跳转到上一个诊断错误/警告。
* **`]d`**：跳转到下一个诊断错误/警告。
* **`Ctrl + w + d`**：在浮动窗口中显示当前行的诊断详细信息。

---

> 💡 **💡 小提示**：如果你习惯了以前通过第三方插件（如 `lsp-zero` 或老版 `nvim-lspconfig` 教程）推荐的 `gd`（跳转定义）或 `<leader>ca` 等键位，官方现在的内置键位（比如用 `Ctrl + ]` 代替 `gd`，用 `gra` 代替代码操作）可能会需要适应一下。如果你不喜欢官方这套，依然可以通过 `vim.keymap.set` 在你的 `init.lua` 中自由覆盖它们。