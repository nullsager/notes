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
```

配置位置：`/etc/snapper/configs/root`

root?

/@home?

快照位置：`/path/to/subvolume/.snapshots/快照编号/snapshot`

创建快照：

```
snapper -c root create --description "helloworld"
```