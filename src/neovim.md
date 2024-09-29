# neovim 相关
## 从源码构建安装neovim
```bash
sudo apt install gettext
git clone https://github.com/neovim/neovim
git checkout stable
cd neovim/
make CMAKE_BUILD_TYPE=RelWithDebInfo
sudo make install
```