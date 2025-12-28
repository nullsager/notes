# neovim 相关

## 从源码构建安装neovim

```bash
sudo apt install gettext
git clone https://github.com/neovim/neovim
git checkout stable
cd neovim/
make CMAKE_BUILD_TYPE=RelWithDebInfo
sudo make install
```

## 快捷键映射

`vim.keymap.set` 函数： `vim.keymap.set({mode}, {lhs}, {rhs}, {opts})`

| n                | 普通模式    |
| ---------------- | ------- |
| i                | 插入模式    |
| v                | 可视模式    |
| t                | 终端模式    |
| x                | 可视模式    |
| s                | 选择模式    |
| c                | 命令模式    |
| o                | 操作符等待模式 |
| "" 或者 {"n", "v"} | 多种模式的组合 |

示例：
```lua
vim.keymap.set("n", "<leader>w", ":w<CR>", { silent = true, desc = "保存文件" })
vim.keymap.set("n", "<leader>t", function()
  print("Hello from Lua!")
end, { desc = "打印消息" })
vim.keymap.set({"n", "v"}, "<leader>y", '"+y', { desc = "复制到系统剪贴板" })
```

[同时使用多套nvim配置](https://michaeluloth.com/neovim-switch-configs/)
