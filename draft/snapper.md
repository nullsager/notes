#linux 

安装：
```
sudo pacman -S snapper snap-pac btrfs-assistant
```

自动生成快照启动项：

```
sudo pacman -S grub-btrfs inotify-tools
sudo systemctl enable --now grub-btrfsd
```

生成 grub 菜单入口

```
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

创建的是配置。

```
snapper -c root create-config /
snapper -c home create-config /home
```

配置位置：`/etc/snapper/configs/root`

root?

/@home?

快照位置：`/path/to/subvolume/.snapshots/快照编号/snapshot`

创建快照：

```
snapper -c root create --description "helloworld"
snapper -c home create --description "helloworld"
```

列出快照

```
snapper -c root list
```

回档家目录：

```
snapper -c home undochange 想回档数字..0
```

0 代表当前版本

删除快照：

```
snapper -c root delete 3-6
```

回档根目录：

从 grub 