## 1. 焦点移动（Focus）
| 快捷键 | 命令 | 作用 |
|--------|------|------|
| `alt+h` / `alt+left` | `focus --direction left` | 将焦点向左移动 |
| `alt+l` / `alt+right` | `focus --direction right` | 将焦点向右移动 |
| `alt+k` / `alt+up` | `focus --direction up` | 将焦点向上移动 |
| `alt+j` / `alt+down` | `focus --direction down` | 将焦点向下移动 |

---

## 2. 窗口移动（Move）
| 快捷键 | 命令 | 作用 |
|--------|------|------|
| `alt+shift+h` / `alt+shift+left` | `move --direction left` | 将当前窗口向左移动 |
| `alt+shift+l` / `alt+shift+right` | `move --direction right` | 将当前窗口向右移动 |
| `alt+shift+k` / `alt+shift+up` | `move --direction up` | 将当前窗口向上移动 |
| `alt+shift+j` / `alt+shift+down` | `move --direction down` | 将当前窗口向下移动 |

---

## 3. 调整窗口大小（Resize）
| 快捷键 | 命令 | 作用 |
|--------|------|------|
| `alt+u` | `resize --width -2%` | 将窗口宽度减少 2% |
| `alt+p` | `resize --width +2%` | 将窗口宽度增加 2% |
| `alt+o` | `resize --height +2%` | 将窗口高度增加 2% |
| `alt+i` | `resize --height -2%` | 将窗口高度减少 2% |
| `alt+r` | `wm-enable-binding-mode --name resize` | **进入 resize 模式**（见下方独立章节） |

---

## 4. 窗口状态切换（Toggle）
| 快捷键 | 命令 | 作用 |
|--------|------|------|
| `alt+v` | `toggle-tiling-direction` | 切换平铺方向（决定新窗口插入的位置） |
| `alt+space` | `wm-cycle-focus` | 轮流聚焦平铺窗口 → 浮动窗口 → 全屏窗口 |
| `alt+shift+space` | `toggle-floating --centered` | 将当前窗口切换为浮动状态，并居中显示 |
| `alt+t` | `toggle-tiling` | 将当前窗口切换为平铺状态 |
| `alt+f` | `toggle-fullscreen` | 将当前窗口切换为全屏状态 |
| `alt+m` | `toggle-minimized` | 最小化当前窗口 |
| `alt+shift+space` 已包含 | （同上） | 同上 |

---

## 5. 工作区（Workspace）管理
### 5.1 切换工作区
| 快捷键 | 命令 | 作用 |
|--------|------|------|
| `alt+s` | `focus --next-active-workspace` | 聚焦下一个有窗口的活动工作区 |
| `alt+a` | `focus --prev-active-workspace` | 聚焦上一个有窗口的活动工作区 |
| `alt+d` | `focus --recent-workspace` | 聚焦最近使用过的工作区 |
| `alt+1` … `alt+9` | `focus --workspace 1` … `9` | 直接切换到对应编号的工作区（1~9） |

### 5.2 移动窗口到其他工作区（并跳转过去）
| 快捷键 | 命令 | 作用 |
|--------|------|------|
| `alt+shift+1` … `alt+shift+9` | `move --workspace N` + `focus --workspace N` | 将当前窗口移动到工作区 N，并切换到该工作区 |

### 5.3 将整个工作区移动到另一显示器
| 快捷键 | 命令 | 作用 |
|--------|------|------|
| `alt+shift+a` | `move-workspace --direction left` | 将当前工作区移动到左侧的显示器 |
| `alt+shift+f` | `move-workspace --direction right` | 将当前工作区移动到右侧的显示器 |
| `alt+shift+d` | `move-workspace --direction up` | 将当前工作区移动到上方的显示器 |
| `alt+shift+s` | `move-workspace --direction down` | 将当前工作区移动到下方的显示器 |

---

## 6. 系统与杂项操作
| 快捷键 | 命令 | 作用 |
|--------|------|------|
| `alt+shift+q` | `close` | 关闭当前聚焦的窗口 |
| `alt+shift+e` | `wm-exit` | 安全退出 GlazeWM 进程 |
| `alt+shift+r` | `wm-reload-config` | 重新加载配置文件（热重载） |
| `alt+shift+w` | `wm-redraw` | 强制重绘所有窗口（解决渲染异常） |
| `alt+enter` | `shell-exec cmd` | 启动 CMD 终端（可自行改为 `wt` 启动 Windows Terminal 等） |
| `alt+shift+p` | `wm-toggle-pause` | 暂停 GlazeWM 的所有窗口管理和快捷键（按 `alt+shift+p` 恢复） |

---

## 7. Resize 模式（`alt+r` 进入）
进入该模式后，原来的大部分快捷键会暂时失效，取而代之的是以下调整窗口大小的按键。按 `escape` 或 `enter` 退出 resize 模式。

| 快捷键（resize 模式下） | 命令 | 作用 |
|------------------------|------|------|
| `h` / `left` | `resize --width -2%` | 宽度减少 2% |
| `l` / `right` | `resize --width +2%` | 宽度增加 2% |
| `k` / `up` | `resize --height +2%` | 高度增加 2% |
| `j` / `down` | `resize --height -2%` | 高度减少 2% |
| `escape` / `enter` | `wm-disable-binding-mode --name resize` | 退出 resize 模式，恢复正常快捷键 |

---

`alt+shift+p`：禁用 glazewm 快捷键，来使用软件自带快捷键

