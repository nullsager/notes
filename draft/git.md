# git 学习

# windows下的gitbash中文乱码

在git bash设置 `git config --global core.quotepath false` 即可

# 在 HTTPS端口下使用 SSH

参考官方文档：[ssh代理](https://docs.github.com/zh/authentication/troubleshooting-ssh/using-ssh-over-the-https-port)

# ssh 连接

`ssh-keygen -t rsa` 创建密钥
将id_rsa.pub文件添加到github中，然后使用 `ssh -T git@github.com` 检测是否连接成功

# 本地已有仓库上传到云端
1. 先在github创建一个仓库，例如名称为Obsidian-Library
2. 在本地的仓库执行`git init`初始化仓库
3. 设置远程URL(使用ssh)：`git remote add origin git@github.com:hacker-dvd/Obsidian-Library.git`
4. 切换到main分支`git branch -M main`
5. 将你的更改提交
6. `git push -u origin main`将修改推送到github
	如果想推送到自己的分支上，使用 `git push 自己分支名`

查看 remote 的详细信息： `git remote -v`

## 一些简单的命令

```bash
alias glg="git log --graph --oneline --decorate --all"
alias gs="git stash push -m"
```

查看远端：

```bash
git remote -v
```



选中分支：在 Local Branches 面板中，使用 ↑ / ↓ 键选中你想切换的目标分支。

执行切换：按 Space (空格键)。

使用 R：raname 分支名

n：创建新分支

fatal：refusing to merge unrelated histories 的解决方案：

```bash
# 这里需要看一看远程的分支名是不是叫main
git pull origin main --allow-unrelated-histories
```

使用 `+` 放大某一个面板的大小

r 会重命名提交

s 能合并提交

d 删除提交

z 撤销


git 中文问题

```
# 让 Git 正确处理中文路径
git config --global core.quotepath false

# 设置 UTF-8 编码
git config --global i18n.commitencoding utf-8
git config --global i18n.logoutputencoding utf-8
```

---

```
git config --global user.name "lc"
git config --global user.email "2981228126@qq.com"
```

