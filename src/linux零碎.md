# linux 的一些零碎知识点
## pacman 使用
+ 卸载软件：`sudo pacman -Rns` (n 代表删除全局配置文件，s 代表删除依赖)
+ 查看自己安装软件的数目：`pacman -Qe | wc -l`
+ 只显示软件名，不显示版本号：`pacman -Qeq`
+ 查看孤儿(没有任何软件以来它们)：`pacman -Qdt`
+ 删除所有的孤儿：`sudo pacman -R $(pacman -Qdtq)`
## kitty
+ 修改主题：`kitty +kitten themes`
## git 中文乱码问题
[参考资料](https://gist.github.com/nightire/5069597)
## 从源码安装 gcc
从gcc官网选择对应的gcc版本：[官网](https://ftp.gnu.org/gnu/gcc/)，下面以安装 gcc-14.2.0为例(从源码安装gcc仍然先要安装gcc和g++)

安装目录选在`/usr/local/`下
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