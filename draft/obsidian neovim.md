在这套配置下，Neovim 和 Obsidian 直接编辑同一个 Vault：

```text
/home/lc/Documents/notes
├── draft/       普通新笔记
├── dailynote/   每日笔记
├── images/      图片附件
└── README.md
```

两边不需要导入或同步：Neovim 保存文件后，Obsidian 会自动检测变化；反之亦然。

## 基本工作流

建议先在 Neovim 中按：

```text
<Space>oo
```

这会用 netrw 打开整个笔记库。由于你的 Leader 是空格，下文中的 `<leader>` 都表示 `<Space>`。

在 netrw 中：

- `Enter`：打开文件或展开目录
- `-`：返回上一级
- `I`：显示或隐藏 netrw 顶部信息
- `<Space>e`：打开或关闭左侧 netrw 文件树

## 查找笔记

### 按笔记名称查找

```text
<Space>of
```

对应：

```vim
:Obsidian quick_switch
```

使用 `fzf-lua` 模糊搜索笔记名称，适合已知笔记大致叫什么的情况。

### 全文搜索

```text
<Space>os
```

对应：

```vim
:Obsidian search
```

搜索所有笔记的正文内容。

例如搜索 `pacman`，可以找出所有提到它的笔记。

## 新建笔记

按：

```text
<Space>on
```

对应：

```vim
:Obsidian new
```

新建的普通笔记默认放在：

```text
/home/lc/Documents/notes/draft
```

这是根据你的 Obsidian `newFileFolderPath` 配置设置的。

也可以直接指定名称：

```vim
:Obsidian new CachyOS配置
```

保存后，Obsidian 会自动发现这个文件。

## 每日笔记

按：

```text
<Space>ot
```

对应：

```vim
:Obsidian today
```

它会打开或创建当天的笔记，例如：

```text
dailynote/2026-08-11.md
```

其他相关命令包括：

```vim
:Obsidian yesterday
:Obsidian tomorrow
```

## 笔记链接

你的配置使用标准 Markdown 链接：

```markdown
[Arch Linux](draft/archlinux.md)
```

而不是默认 Wiki 链接：

```markdown
[[archlinux]]
```

这是为了与 Obsidian 当前的 `useMarkdownLinks: true` 保持一致。

常见操作：

- 将光标放在链接上按 `gf`：跳转到对应文件。
- 返回上一个位置：按 `<C-o>`。
- 查看当前笔记中的链接：

```vim
:Obsidian links
```

- 查看有哪些笔记链接到了当前笔记：

```text
<Space>ob
```

对应：

```vim
:Obsidian backlinks
```

### 给选中文字创建链接

可视模式选中文字，然后执行：

```vim
:Obsidian link
```

如果目标笔记还不存在，可以执行：

```vim
:Obsidian link_new
```

## 重命名笔记

在要重命名的笔记中执行：

```vim
:Obsidian rename
```

相比直接用文件管理器重命名，这个命令的重要优势是会尝试更新 Vault 中引用该笔记的链接。

## 粘贴图片

先把图片复制到系统剪贴板，然后在笔记中按：

```text
<Space>op
```

对应：

```vim
:Obsidian paste_img
```

图片会保存到：

```text
/home/lc/Documents/notes/images
```

并在当前位置插入 Markdown 图片链接。

Wayland/CachyOS 下需要安装剪贴板工具：

```bash
sudo pacman -S wl-clipboard
```

## 在 Obsidian 桌面程序中打开

编辑某篇笔记时执行：

```vim
:Obsidian open
```

插件会尝试让 Obsidian 桌面程序打开当前笔记。

如果系统没有正确注册 `obsidian://` URL scheme，这项功能可能无法使用，但不会影响其他功能。

## Markdown 编辑体验

你现有的 `render-markdown.nvim` 会负责显示效果：

- 标题带图标和层级样式
- 代码块美化
- Markdown 表格美化
- LaTeX 公式渲染
- 普通模式显示渲染结果
- 插入模式显示原始 Markdown，方便编辑
- 光标所在行显示原始语法

因此通常是：

1. 按 `i` 进入插入模式，编辑原始 Markdown。
2. 按 `Esc` 返回普通模式。
3. 当前内容自动恢复为渲染后的样式。

之前配置的 fcitx5 同步也会生效：离开插入模式自动切英文，再次进入时恢复之前的中文状态。

## 推荐的日常流程

```text
<Space>oo    进入整个笔记库
<Space>of    找到已有笔记
<Space>on    创建普通笔记
<Space>ot    写今天的日记
<Space>os    全文搜索知识库
<Space>ob    查看谁引用了当前笔记
<Space>op    粘贴截图或图片
:w           保存，Obsidian 自动看到变化
```

需要查看插件所有功能时，可以输入：

```vim
:Obsidian
```

然后按空格或 `Tab` 查看可用子命令补全。