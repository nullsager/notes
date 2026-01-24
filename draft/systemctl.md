#linux 

在 Arch Linux 中，“设置了一堆”通常意味着你**启用（enable）**了服务、**修改**了配置文件，或者**创建**了新的服务。

根据你具体想看的内容，有以下几种最常用的查看方式：

### 1. 查看所有“开机自启”的服务 (最常用)
如果你想知道哪些服务被你设置为 `enable`（开机自动运行），可以使用：

```bash
systemctl list-unit-files --state=enabled
```
这个命令会列出所有被启用的单元。
*   **技巧**：如果你只想看服务（排除掉 socket、target 等），可以配合 `grep`：
    ```bash
    systemctl list-unit-files --state=enabled | grep service
    ```

### 2. 查看当前“正在运行”的服务
如果你想知道现在这一刻哪些服务是活跃的（无论是否开机自启）：

```bash
systemctl list-units --type=service --state=running
```
或者查看所有活跃的单元（包括挂载点、定时器等）：
```bash
systemctl list-units --state=active
```

### 3. 查看你“手动修改或创建”的配置 (神器)
Arch 用户最喜欢的命令之一。`systemd-delta` 可以显示系统默认配置和你手动修改/覆盖的配置之间的差异。

```bash
systemd-delta
```
*   **输出解释**：
    *   `[OVERRIDDEN]`：表示你用 drop-in 文件（`.conf`）覆盖了默认设置。
    *   `[EXTENDED]`：表示你添加了额外的配置。
    *   如果你自己写了服务文件放在 `/etc/systemd/system/` 下，这里也会有相关体现。

### 4. 查看你设置的“定时器” (Timers)
如果你用 systemd 替代了 cron 来做定时任务：

```bash
systemctl list-timers
```
这会显示下次运行时间、上次运行时间以及对应的服务。

### 5. 直接查看配置文件夹 (最直观)
当你执行 `systemctl enable xxx` 时，systemd 实际上是在创建软链接。当你修改配置时，文件通常在 `/etc/systemd/system/`。

你可以直接列出这个目录，看看里面有哪些“非系统默认”的东西：

```bash
# 查看你自己创建或修改的服务文件
ls -l /etc/systemd/system/

# 查看你启用的服务（实际上是查看 wants 目录下的软链接）
ls -lR /etc/systemd/system/*.wants/
```

### 6. 如果你使用了 `systemctl --user` (用户级服务)
如果你是以普通用户身份设置的服务（非 root），记得加上 `--user` 参数：

```bash
# 查看用户级已启用的服务
systemctl --user list-unit-files --state=enabled

# 查看用户级正在运行的服务
systemctl --user list-units --type=service --state=running
```

---

### 总结
*   **查开机自启**：`systemctl list-unit-files --state=enabled`
*   **查修改差异**：`systemd-delta` (强烈推荐)
*   **查正在运行**：`systemctl list-units --state=running`

在 Arch Linux (systemd) 中，**“现在的状态”**（运行/停止）和**“未来的状态”**（开机自启/禁止自启）是两个独立的概念。

你需要根据需求组合使用以下命令（大部分需要 `sudo` 权限）：

### 1. 改变“当前”状态（立即生效，不影响重启后）

如果你只想现在让它跑起来，或者现在关掉它：

*   **启动服务**：
    ```bash
    sudo systemctl start 服务名
    ```
*   **停止服务**：
    ```bash
    sudo systemctl stop 服务名
    ```
*   **重启服务**（修改配置文件后常用）：
    ```bash
    sudo systemctl restart 服务名
    ```

### 2. 改变“开机自启”状态（重启后生效，不影响当前）

如果你希望它下次开机自动运行，或者取消自动运行：

*   **开启开机自启**：
    ```bash
    sudo systemctl enable 服务名
    ```
*   **关闭开机自启**：
    ```bash
    sudo systemctl disable 服务名
    ```

---

### 3. ⭐ 组合拳（推荐：既要现在运行，又要开机自启）

在 Arch Linux 中，我们最常使用的其实是 `--now` 参数，它可以把上面两步合并成一步：

*   **设置为开机自启，并且立刻启动它**：
    ```bash
    sudo systemctl enable --now 服务名
    ```
    *(等同于先 enable 再 start)*

*   **取消开机自启，并且立刻停止它**：
    ```bash
    sudo systemctl disable --now 服务名
    ```
    *(等同于先 disable 再 stop)*

---

### 4. 如何确认我设置成功了？

查看服务状态是最稳妥的方法：

```bash
systemctl status 服务名
```

**看输出结果的这两个地方：**

1.  **`Loaded: ...; enabled/disabled; ...`**
    *   `enabled`：表示**开机自启**已开启。
    *   `disabled`：表示**开机自启**已关闭。
2.  **`Active: active (running) / inactive (dead)`**
    *   `active (running)`：表示**现在**正在运行。
    *   `inactive (dead)`：表示**现在**没运行。

### 实战举例

假设你要操作蓝牙服务（`bluetooth`）：

*   **我要用蓝牙，以后开机也要自动开**：
    `sudo systemctl enable --now bluetooth`
*   **我暂时不用蓝牙，关了它，但下次重启还要自动开**：
    `sudo systemctl stop bluetooth`
*   **我以后再也不想用蓝牙了，现在也给我关了**：
    `sudo systemctl disable --now bluetooth`


```
killall clash-verge
pkill clash-verge
```