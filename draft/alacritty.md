#linux 

windows 路径：

```bash
AppData\Roaming\alacritty
```

使用 `ctrl+shift+f` 进行搜索

配置：

```
window.opacity = 0.7

font.normal = { family = "JetBrainsMono Nerd Font", style = "Regular" }
font.bold = { family = "JetBrainsMono Nerd Font", style = "Bold" }
font.italic = { family = "JetBrainsMono Nerd Font", style = "Italic" }
font.bold_italic = { family = "JetBrainsMono Nerd Font", style = "Bold Italic" }
font.size = 13.0

terminal.shell.program = "wsl.exe"
terminal.shell.args = ["--cd", "~"]

# 设置 Alt + v 进入 Vi 模式
[[keyboard.bindings]]
key = "V"
mods = "Alt"
action = "ToggleViMode"

# 设置 Alt + n 打开新窗口
[[keyboard.bindings]]
key = "N"
mods = "Alt"
action = "CreateNewWindow"
```