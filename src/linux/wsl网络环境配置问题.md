使用clash-verge+archlinux

参考文献：[使用 WSL 访问网络应用程序](https://learn.microsoft.com/zh-cn/windows/wsl/networking)

首先，将wslconfig中无关的内容清空，网络模式设置为最传统的NAT，而不是mirrored

clash-verge设置（注意关闭虚拟网卡模式，打开系统代理）：

![](Pasted%20image%2020250819214215.png)

在wsl中使用：

```
ip route show | grep -i default | awk '{ print $3}'
```

获取windows主机IP地址

在.bashrc中配置：

```bash
export http_proxy="http://172.19.16.1:7897"
export https_proxy="http://172.19.16.1:7897"
export all_proxy="socks5://172.19.16.1:7897"
```

其中`172.19.16.1`是主机ip地址，7897是clash-verge端口号