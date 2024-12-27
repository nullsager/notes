# archlinux 相关

## archwsl 安装

参考教程[Windows11 wsl2 安装archlinux子系统](https://zhuanlan.zhihu.com/p/613454594)

注意修改的点：

1. `pacman -Sy archlinuxcn-keyring`应该修改为
	 `sudo pacman-key --lsign-key "farseerfc@archlinux.org"`

2. 执行`.\Arch.exe config --default-user {username}`时要注意关闭代理，不然报错 

## ssh

安装ssh： `sudo pacman -S openssh`

启动ssh: `sudo systemctl start sshd`

设置SSH服务开机自启: `sudo systemctl enable sshd`

检查SSH服务状态: `sudo systemctl status sshd`

## pacman

`pacman -Rns` 卸载一个程序
`pacman -Q` 查看安装的所有包
`pacman -Qe` 查看显示安装的包
`pacman -Qeq` 只查看显示安装的包的包名
`pacman -Qe | wc -l` 显示安装的包的个数
`pacman -Qn` 从主存储库安装的包
`pacman -Qm` 从aur安装的包
`sudo pacman -Sc` 删除缓存中的所有包

## 安装的软件包

```bash
sudo pacman -S man-db 
```

## 代理配置

```bash
proxy() {
  export hostip=$(cat /etc/resolv.conf | grep -oP '(?<=nameserver\ ).*')
  export https_proxy="http://${hostip}:7890"
  export http_proxy="http://${hostip}:7890"
  echo "Proxy started. https_proxy and http_proxy set to http://${hostip}:7890"
}

unproxy() {
  unset https_proxy
  unset http_proxy
  echo "Proxy stopped. https_proxy and http_proxy variables unset."
}
```

## 零碎问题

* 查看安装包的记录：`grep "installed" /var/log/pacman.log`

* 解决半英半中的问题：`sudo nvim /etc/locale.gen`将`zh_CN.UTF-8 UTF-8`取消注释
 `sudo locale-gen`

* 安装nodejs和npm：`sudo pacman -S nodejs npm`

* 截图:`sudo pacman -S flameshot`
