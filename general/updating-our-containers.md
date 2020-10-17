# 更新容器

当程序依赖和应用更新或改变时，我们就会更新镜像。确保你始终运行的是最新版本，因为他们可能包含重要的错误修复和新功能。

------

## 更新所需的步骤

大部分的Docker容器是无法改变的，这意味这容器一旦创建便无法轻易修改如卷、端口映射等重要配置。所以为了升级容器，你必须重新创建容器。

### 停止容器

首先，你需要先停止容器。

```shell
docker stop <container_name>
```

### 删除容器

容器停止后，删除容器。

> **重要**：你应该还记得创建容器时，映射的 `/config` 卷吧？ 如果当初没有将这个卷映射到宿主机上，那么删除容器也将删除容器的全部数据！[看这篇文章了解更多](general/volumes.md)。

```shell
docker rm <container_name>
```

### 拉取最新版本

从Docker Hub拉取镜像的最新版本。

```shell
docker pull linuxserver/<image_name>
```

### 重新创建容器

最后，重新创建容器。因为重新创建时需要使用跟最初创建时使用相同的参数，所以这可能稍微有点麻烦。但你可以使用Docker Compose来操作。参考[这篇文章了解docker-compose](general/docker-compose.md)。

```shell
docker create \
    --name=<container_name> \
    -v <path_to_data>:/config \
    -e PUID=<uid> \
    -e PGID=<gid> \
    -p <host_port>:<app_port> \
    linuxserver/<image_name>
```

------

## Docker Compose

可以使用Docker Compose更新单个容器：

```shell
docker-compose pull linuxserver/<image_name>
docker-compose up -d <container_name>
```

也可以以此更新所有的容器：

```shell
docker-compose pull
docker-compose up -d
```

------

## 删除旧的容器

当Docker镜像升级后，系统将下载新版本的镜像并存储在宿主机上，但升级并不会自动删除旧版本的镜像，这些旧镜像将占用你的磁盘空间。你可以使用 `prune` 删除不使用的镜像清理磁盘空间：

```shell
docker image prune
```

