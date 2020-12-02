# linuxserver/minetest

> [!TIP]
>
> 前半部分是翻译官方的文档，最后一部分是我的简单试用（个别软件会深度试用），如果对Docker已经有一定的了解了，可以直接跳转到最后面 [翻译之外](#翻译之外) 这部分来查看。

Minetest → http://www.minetest.net/

GitHub → https://github.com/linuxserver/docker-minetest

Docker Hub → https://hub.docker.com/r/linuxserver/minetest

[Minetest](http://www.minetest.net/)（服务器） 是一款近乎无限世界的区块沙盒游戏和游戏引擎，灵感来自InfiniMiner，Minecraft等。

------

## 支持的系统架构

得益于docker的跨平台属性，我们的镜像也支持多架构（如，x86-64、arm64、armhf）。

直接拉取 `ghcr.io/linuxserver/minetest` 应该就可以自动获取适合你系统架构的版本，当然你也可以通过 tag 获取特定的版本。

| 架构   | Tag            |
| ------ | -------------- |
| x86-64 | amd64-latest   |
| arm64  | arm64v8-latest |
| armhf  | arm32v7-latest |



------

## 拉取镜像

```shell
docker pull ghcr.io/linuxserver/minetest
```

------

## 使用方法

以下是一些简单的示例。

### docker-compose

兼容docker-compose v2

```yaml
---
version: "2.1"
services:
  minetest:
    image: ghcr.io/linuxserver/minetest
    container_name: minetest
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
      - CLI_ARGS="--gameid minetest" #optional
    volumes:
      - <path to data>:/config/.minetest
    ports:
      - 30000:30000/udp
    restart: unless-stopped
```

### docker cli

```shell
docker run -d \
  --name=minetest \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Europe/London \
  -e CLI_ARGS="--gameid minetest" `#optional` \
  -p 30000:30000/udp \
  -v <path to data>:/config/.minetest \
  --restart unless-stopped \
  ghcr.io/linuxserver/minetest
```

## 参数

Docker镜像在使用的时候需要配置一些参数，这些参数使用 `:` 分隔，分别表示 `宿主机:容器内`。例如 `-p 8080:80` 指的是将容器内的`80`端口映射到宿主机上的`8080`端口，即通过宿主机网络访问的话，访问`8080`端口即是访问容器内的`80`端口。

### 端口（`-p`）

| port        | 说明               |
| ----------- | ------------------ |
| `30000/udp` | Minetest的监听端口 |

### 环境变量（`-e`）

| env                            | 说明                                        |
| ------------------------------ | ------------------------------------------- |
| `PUID=1000`                    | 用户的 UID，详见下面的说明                  |
| `PGID=1000`                    | 用户的 GID，详见下面的说明                  |
| `TZ=Europe/London`             | 设置时区，在国内的话可以使用 Asia/Shanghai  |
| `CLI_ARGS="--gameid minetest"` | （可选）指定要用于启动应用程序的任何CLI变量 |

### 卷映射（`-v`）

| volume              | 说明                 |
| ------------------- | -------------------- |
| `/config/.minetest` | 存储配置文件和地图等 |




## 从文件加载环境变量

可以使用前缀名为 `FILE__` 的文件来加载环境变量。

例：

```
-e FILE__PASSWORD=/run/secrets/mysecretpassword
```

将把 `/run/secrets/mysecretpassword` 文件中的内容作为 `PASSWORD` 变量的值。

------

## 运行程序的umask（文件掩码）

我们所有的镜像都可以使用 -e UMASK=022 来设置覆盖容器内服务的umask。关于Linux umask可以通过[这里](https://en.wikipedia.org/wiki/Umask)了解，或自行百度学习。

------

## UID和GID

当使用 `-v` 映射卷的时候，宿主机和容器内会出现关于权限的问题，我们的镜像可以通过指定 `PUID` 和 `GUID` 来避免此类问题。

可以使用你目前正在使用的用户的UID和GID进行设置，这样就不会存在权限问题。

可以通过如下的方式查看当前用户的UID和GID：

```shell
  $ id $user
    uid=1000(dockeruser) gid=1000(dockergroup) groups=1000(dockergroup)
```

> [!NOTE]
>
> 如果没特殊需求，可以将后续所有容器的id都设置为当前非root用户的id，这样会省去解决很多关于文件权限的问题（可以查看 [什么是PUID和PGID](general/understanding-puid-and-pgid.md) 这篇文章了解更多）。当然，如果有特殊需要可以给不同的容器设置不同的id，即分配给不同的用户。使用 `useradd` 命令即可添加用户。通常来说，除root账户外，第一个建立的用户的UID和GID都是1000，然后递增生成。

------

## 安装说明

可以在`/config/.minetest`中找到地图、mod和配置文件。

客户端和服务器的版本必须相同，请浏览此处的标签以拉出适合您服务器的版本：https://hub.docker.com/r/linuxserver/minetest/tags

------

## 支持

- 进入容器：
  - `docker exec -it minetest /bin/bash`
- 查看容器日志：
  - `docker logs -f minetest`
- 查看容器版本号：
  - `docker inspect -f '{% raw %}{{% endraw %}{ index .Config.Labels "build_version" }}' minetest`
- 查看镜像版本号：
  - `docker inspect -f '{% raw %}{{% endraw %}{ index .Config.Labels "build_version" }}' ghcr.io/linuxserver/minetest`

------

## 翻译之外

暂未试用