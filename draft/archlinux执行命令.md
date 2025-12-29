#linux

生成 grub 配置文件：

```
grub-mkconfig -o /efi/grub/grub.cfg
```

systemctl 相关命令：

```
systemctl enable --now NetworkManager

```