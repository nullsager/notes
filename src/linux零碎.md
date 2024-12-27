# linux 的一些零碎知识点

## pacman 使用

* 卸载软件：`sudo pacman -Rns` (n 代表删除全局配置文件，s 代表删除依赖)
* 查看自己安装软件的数目：`pacman -Qe | wc -l`
* 只显示软件名，不显示版本号：`pacman -Qeq`
* 查看孤儿(没有任何软件以来它们)：`pacman -Qdt`
* 删除所有的孤儿：`sudo pacman -R $(pacman -Qdtq)`

## kitty

* 修改主题：`kitty +kitten themes`

## git 中文乱码问题

[参考资料](https://gist.github.com/nightire/5069597)

## git错误

在git clone时，出现报错： `git clone Recv failure: Connection was reset`

解决方案：

```bash
git config --global http.proxy http://127.0.0.1:10809
git config --global https.proxy http://127.0.0.1:10811
```

## 从源码安装 gcc

从gcc官网选择对应的gcc版本：[官网](https://ftp.gnu.org/gnu/gcc/)，下面以安装 gcc-14.2.0为例(从源码安装gcc仍然先要安装gcc和g++)

安装目录选在 `/usr/local/` 下

```bash
sudo wget https://ftp.gnu.org/gnu/gcc/gcc-14.2.0/gcc-14.2.0.tar.gz
sudo tar -xzf gcc-14.2.0.tar.gz
sudo rm gcc-14.2.0.tar.gz
cd gcc-14.2.0/
sudo apt install bzip2
sudo ./contrib/download_prerequisites
sudo mkdir build
cd build/

../configure --prefix=/usr/local/gcc-14.2.0 \
--enable-languages=c,c++ \
--disable-multilib \
--enable-threads=posix \
--disable-bootstrap

make -j$(nproc)
sudo make install

# 在.bashrc中添加
# 添加新安装的GCC到PATH
export PATH=/usr/local/gcc-14.2.0/bin:$PATH

# 配置库路径
export LD_LIBRARY_PATH=/usr/local/gcc-14.2.0/lib64:$LD_LIBRARY_PATH
```

## 统计代码行数

```bash
# 统计单个文件
wc -l user.py
# 统计当前目录所有文件行数
wc -l *       
# 统计当前目录所有.h文件行数
wc -l *.h

# 统计所有py文件的行数
wc -l `find . -name "*.py"`
# 统计.h和.cpp文件行数
wc -l `find . -name "*.h";find . -name "*.cpp"`
# 统计.py和.html文件行数, 多扩展名的正则写法
wc -l `find -E . -regex '.*\.(py|html)'`

# 统计当前目录下所有文件行数
find  . * | xargs wc -l
# 统计java文件的代码行数
find . -name *.java | xargs wc -l
# 统计文件个数，不xargs，带上就是统计代码行数了
find . -name "*.h" | wc -l

# 统计目录并按行数排序
find . -name *.java | xargs wc -l | sort -n
# 统计py和html行数，并按照第二列(文件名列)倒序排序
wc -l `find -E . -regex '.*\.(py|html)'` | sort -r -k2

# 统计行数，去除空行
find . \( -name "*.h" -o -name "*.c" \) |xargs cat|grep -v ^$|wc -l
# 统计行数，去除注释
find . -name "*.java"|xargs cat|grep -v -e ^$ -e ^\s*\/\/.*$|wc -l
```

## tmux

文档参考：[oh my tmux](https://github.com/gpakosz/.tmux)
快捷键：
* `<prefix> -` 竖直分屏
* `<prefix> _` 水平分屏
* `<prefix> h`,      `<prefix> j`,      `<prefix> k`,  `<prefix> l` 上下左右切换pane
* `<prefix> H`,      `<prefix> J`,      `<prefix> K`,  `<prefix> L` 调整pane的大小
* `<prefix> <`,  `<prefix> >` 交换pane的位置
* `<prefix> +` 将当前所在pane最大化/恢复原大小
* `<prefix> q` 根据给出的数字在pane之间进行移动
* `<prefix> m` 打开/关闭鼠标模式
* `<prefix> c` 创建一个 window
* `<prefix> 数字`：切换 window
* `<prefix> Tab` 切换到最近一个打开的window
* `<prefix> &` 杀死当前window
* `<prefix> ,` 重命名当前window
* `<prefix> C-c` 创建一个新的session
* `<prefix> $` 重命名当前session
* `tmux kill-session -t 0` 杀死 0 号session
* `tmux kill-server` 杀死所有session

## linux 命令

查看文件夹大小： `du -sh 文件夹路径`

如果你希望查看指定目录下每个子目录的大小，可以使用 `--max-depth` 选项: `du -h --max-depth=1 文件夹路径`

## 安装nodejs

参考资料：
1. [nodejs](https://nodejs.org/en/download/package-manager)
2. [nvm](https://github.com/nvm-sh/nvm)

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
nvm install 22
```

## archlinux 代理配置

`sudo pacman -S v2ray` 安装 v2ray 内核

`yay -S v2raya-bin` 安装 v2raya

启动服务

```
systemctl enable --now v2raya.service
systemctl restart v2raya.service
```

参考官方教程[v2rayA](https://v2raya.org/docs/prologue/quick-start/)配置好 v2rayA

然后在 firefox 里配置

```
127.0.0.1   20171  HTTP Proxy
127.0.0.1   20171  HTTPS Proxy
127.0.0.1   20170  SOCKS Host
```
