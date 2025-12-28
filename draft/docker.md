# docker 学习

## 资料

* [Docker 快速入门](https://docker.easydoc.net/doc/81170005/cCewZWoN/lTKfePfP)
* [docker教程](https://www.quanxiaoha.com/docker/docker-tutorial.html)
* [Docker — 从入门到实践](https://yeasy.gitbook.io/docker_practice)

## 安装

```bash
sudo pacman -S docker
sudo systemctl enable --now docker.socket # 在首次需要使用 Docker 时才启动它

# 验证是否安装成功
sudo docker info
```

## 基础配置

* 将自己添加到docker用户组中：`sudo usermod -aG docker your_username`

## 基本命令

* 查看现在有哪些活动的容器：`docker ps`
* 查看所有的容器： `docker ps -a`
* 删除一个 docker 容器：先使用 `docker stop 容器名` 停用容器, 然后使用 `docker rm 容器名` 删除容器
* docker删除镜像：`docker rmi ID`
* docker登录：`docker login`
* 停止所有运行中的容器：`docker stop $(docker ps -a -q)`
* 强制停止所有容器：`docker kill $(docker ps -a -q)`
* 删除所有容器（包括已停止的）：`docker rm $(docker ps -a -q)`
* 停止并删除所有容器（组合命令）：`docker stop $(docker ps -a -q) && docker rm $(docker ps -a -q)`
* 一次性清理所有容器、网络、镜像和缓存：`docker system prune -a`
* 重新启动并进入容器: `docker start -i <container_id_or_name>`

*注意：-q 参数表示只返回容器的 ID*