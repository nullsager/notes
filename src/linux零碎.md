# linux 的一些零碎知识点
## pacman 使用
+ 卸载软件：`sudo pacman -Rns` (n 代表删除全局配置文件，s 代表删除依赖)
+ 查看自己安装软件的数目：`pacman -Qe | wc -l`
+ 只显示软件名，不显示版本号：`pacman -Qeq`
+ 查看孤儿(没有任何软件以来它们)：`pacman -Qdt`
+ 删除所有的孤儿：`sudo pacman -R $(pacman -Qdtq)`
## kitty
+ 修改主题：`kitty +kitten themes`