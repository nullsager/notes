# 安装运行mysql（通过docker）

## 安装mysql：

```shell
docker pull mysql:latest
```
# 初始化mysql容器

```shell
docker run --name mysql-container -e MYSQL_ROOT_PASSWORD=your_password -d mysql:latest
```

+ --name mysql-container：为容器指定一个名称。
+ -e MYSQL_ROOT_PASSWORD=your_password：设置 MySQL 的 root 用户密码。
+ -d：以分离模式运行容器。

## 查看容器状态

查看正在运行的容器：`docker ps`

查看所有的容器：`docker ps -a`

## 连接到mysql

```bash
docker exec -it mysql-container mysql -uroot -p
```

## 停止，删除，重新启动容器

停止容器：

```bash
docker stop mysql-container
```

删除容器：

```bash
docker rm mysql-container
```

重新启动容器：

```bash
docker start mysql-container
```