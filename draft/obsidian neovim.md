
- Vault：`/home/lc/Documents/notes`
- 新笔记目录：`draft`
- 日记目录：`dailynote`
- 图片目录：`images`
- 链接格式：标准 Markdown 链接
- 搜索界面：复用 `fzf-lua`
- Markdown 渲染：继续使用现有 `render-markdown.nvim`
- 不自动给旧笔记添加 YAML frontmatter

常用快捷键：

| 快捷键 | 功能 |
|---|---|
| `<leader>oo` | 打开笔记库 |
| `<leader>of` | 按文件名快速查找笔记 |
| `<leader>os` | 全文搜索所有笔记 |
| `<leader>ob` | 查看当前笔记的反向链接 |
| `<leader>ot` | 打开或创建今天的日记 |
| `<leader>on` | 新建笔记 |
| `<leader>op` | 从剪贴板粘贴图片到 `images` |

在笔记中还可以使用：

- `gf`：跳转到光标下的 Markdown/Wiki 链接。
- `:Obsidian links`：查看当前笔记中的链接。
- `:Obsidian tags`：搜索标签。
- `:Obsidian rename`：重命名笔记并更新引用。
- `:Obsidian toc`：显示当前笔记目录。
- `:Obsidian open`：在 Obsidian 桌面程序中打开当前笔记。

第一次重启 Neovim 时，`vim.pack` 会自动下载插件。配置已经通过 Lua 语法检查。

图片粘贴功能在 Wayland 下通常需要 `wl-clipboard`：

```bash
sudo pacman -S wl-clipboard
```

之后建议先按 `<leader>oo` 进入 Vault，再使用其余 Obsidian 快捷键。