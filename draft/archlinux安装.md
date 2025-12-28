基本安装：

```bash
systemctl stop reflector.service

iwctl
station wlan0 get-networks
station wlan0 connect <wifi name>
exit
ping www.baidu.com -n

vim /etc/pacman.d/mirrorlist
sudo pacman -Sy

#查看磁盘分区
lsblk
cfdisk /dev/nvme0n1
# 512M EFI System
# 其余全是根目录占用

cfdisk /dev/nvme1n1

mkfs.fat -F32 /dev/nvme0n1p1
mkfs.ext4 /dev/nvme0n1p2
mkfs.ext4 /dev/nvme1n1p1


# 挂载
mount /dev/nvme0n1p2 /mnt
mkdir /mnt/boot
mount /dev/nvme0n1p1 /mnt/boot
mkdir /mnt/home
mount /dev/nvme1n1p1 /mnt/home
archinstall
```
---

安装中文 locale：

```bash
sudo /etc/locale.gen
```

将下面两行的注释取消：

```bash
en_US.UTF-8 UTF-8
zh_CN.UTF-8 UTF-8
```

执行命令：

```bash
sudo locale-gen
```

---

添加archlinuxcn源：

```bash
# 在 /etc/pacman.conf 文件末尾添加以下两行
[archlinuxcn]
Server = https://repo.archlinuxcn.org/$arch

sudo pacman -Syu
sudo pacman -S archlinuxcn-keyring
```

---

安装中文字体：

```bash
sudo pacman -S noto-fonts-cjk
```

参考[字体配置/中文](https://wiki.archlinuxcn.org/wiki/%E5%AD%97%E4%BD%93%E9%85%8D%E7%BD%AE/%E4%B8%AD%E6%96%87)添加配置win样式字体

创建`~/.config/fontconfig/conf.d/64-language-selector-prefer.conf`，添加如下内容：

```
<?xml version="1.0"?>
<fontconfig>
  <alias>
    <family>sans-serif</family>
    <prefer>
      <family>Noto Sans</family>
      <family>Noto Sans CJK SC</family>
      <family>Noto Sans CJK TC</family>
      <family>Noto Sans CJK JP</family>
      <family>Noto Sans CJK KR</family>
      <family>Noto Color Emoji</family>
      <family>Noto Emoji</family>
    </prefer>
  </alias>
  <alias>
    <family>serif</family>
    <prefer>
      <family>Noto Serif</family>
      <family>Noto Serif CJK SC</family>
      <family>Noto Serif CJK TC</family>
      <family>Noto Serif CJK JP</family>
      <family>Noto Serif CJK KR</family>
      <family>Noto Color Emoji</family>
      <family>Noto Emoji</family>
    </prefer>
  </alias>
  <alias>
    <family>monospace</family>
    <prefer>
      <family>Hack</family>
      <family>Noto Sans Mono CJK SC</family>
      <family>Noto Sans Mono CJK TC</family>
      <family>Noto Sans Mono CJK JP</family>
      <family>Noto Sans Mono CJK KR</family>
      <family>Noto Color Emoji</family>
      <family>Noto Emoji</family>
    </prefer>
  </alias>
</fontconfig>
```

编辑完成后刷新系统缓存：`fc-cache -fv`

---

中文输入法安装：

```bash
sudo pacman -S fcitx5-im fcitx5-chinese-addons
```
在kde wayland中，前往系统设置 > 输入设备 > 虚拟键盘，选择 Fcitx 5

在`/etc/environment`中添加：

```
XMODIFIERS=@im=fcitx
SDL_IM_MODULE=fcitx
GLFW_IM_MODULE=ibus
```

然后在kde设置界面添加输入法pinyin

---

我在安装 archlinux 时使用 archinstall 安装时驱动有两个选项：

第一个是 Nvidia (open kernel module for newer GPUs, Turing+)
包括 dkms, libva-nvidia-driver, nvidia-open-dkms, xorg-server, xorg-xinit
适合在显卡为 20 版本以上安装。

第二个是 Nvidia (proprietary)
包括 dkms, libva-nvidia-driver, nvidia-dkms, xorg-server, xorg-xinit

