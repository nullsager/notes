#linux

生成 grub 配置文件：

```
grub-mkconfig -o /efi/grub/grub.cfg
```

systemctl 相关命令：

```
systemctl enable --now NetworkManager

```

中文字体：

```
adobe-source-han-sans-cn-fonts
wqy-zenhei
```

安装 nerd font：

```
sudo pacman -S ttf-jetbrains-mono-nerd
fc-cache -fv
fc-list
```

