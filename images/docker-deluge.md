# linuxserver/deluge

> [!TIP]
>
> 前半部分是翻译官方的文档，最后一部分是我的简单试用（个别软件会深度试用），如果对Docker已经有一定的了解了，可以直接跳转到最后面 [翻译之外](#翻译之外) 这部分来查看。

Deluge → http://deluge-torrent.org/

GitHub → https://github.com/linuxserver/docker-deluge

Docker Hub → https://hub.docker.com/r/linuxserver/deluge

[Deluge](http://deluge-torrent.org/) 是一个轻量级、免费的跨平台BitTorrent客户端。

- 完全加密
- Web界面
- 插件系统
- 更多...

------

## 支持的系统架构

得益于docker的跨平台属性，我们的镜像也支持多架构（如，x86-64、arm64、armhf）。

直接拉取 `ghcr.io/linuxserver/deluge` 应该就可以自动获取适合你系统架构的版本，当然你也可以通过 tag 获取特定的版本。

| 架构   | Tag            |
| ------ | -------------- |
| x86-64 | amd64-latest   |
| arm64  | arm64v8-latest |
| armhf  | arm32v7-latest |


------

## 拉取镜像

```shell
docker pull ghcr.io/linuxserver/deluge
```

------

## 使用方法

以下是一些简单的示例。

### docker-compose（[推荐](general/docker-compose.md)）

兼容docker-compose v2

```yaml
---
version: "2.1"
services:
  deluge:
    image: linuxserver/deluge
    container_name: deluge
    network_mode: host
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
      - UMASK_SET=022 #optional
      - DELUGE_LOGLEVEL=error #optional
    volumes:
      - /path/to/deluge/config:/config
      - /path/to/your/downloads:/downloads
    restart: unless-stopped
```

### docker cli

```shell
docker run -d \
  --name=deluge \
  --net=host \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Europe/London \
  -e UMASK_SET=022 `#optional` \
  -e DELUGE_LOGLEVEL=error `#optional` \
  -v /path/to/deluge/config:/config \
  -v /path/to/your/downloads:/downloads \
  --restart unless-stopped \
  linuxserver/deluge
```

## 参数

Docker镜像在使用的时候需要配置一些参数，这些参数使用 `:` 分隔，分别表示 `宿主机:容器内`。例如 `-p 8080:80` 指的是将容器内的`80`端口映射到宿主机上的`8080`端口，即通过宿主机网络访问的话，访问`8080`端口即是访问容器内的`80`端口。

### 端口（`-p`）

| port | 说明 |
| ---- | ---- |
| 无   | 无   |

### 网络设置（`--net`）

| network      | 说明                       |
| ------------ | -------------------------- |
| `--net=host` | 需要共享宿主机的网络，必填 |

### 环境变量（`-e`）

| env                     | 说明                                                         |
| ----------------------- | ------------------------------------------------------------ |
| `PUID=1000`             | 用户的 UID，详见下面的说明                                   |
| `PGID=1000`             | 用户的 GID，详见下面的说明                                   |
| `TZ=Europe/London`      | 设置时区，在国内的话可以使用 Asia/Shanghai                   |
| `UMASK_SET=022`         | deluge的umask设置，默认为 022                                |
| `DELUGE_LOGLEVEL=error` | 设置Deluge运行时的日志输出，默认是输出deluged的info和delgued-web的warning日志。 |

### 卷映射（`-v`）

| volume       | 说明             |
| ------------ | ---------------- |
| `/config`    | 配置文件所在路径 |
| `/downloads` | 种子下载文件集   |

------

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

管理界面位于 `http://ip:8112` ，默认的用户名/密码是：`admin/deluge`

更改登陆密码（推荐）：`Preferences->Interface->Password`

更改下载位置：`Preferences->Downloads`，使用 `/downloads`

------

## 支持

- 进入容器：
  - `docker exec -it deluge /bin/bash`
- 查看容器日志：
  - `docker logs -f deluge`
- 查看容器版本号：
  - `docker inspect -f '{% raw %}{{% endraw %}{ index .Config.Labels "build_version" }}' deluge`
- 查看镜像版本号：
  - `docker inspect -f '{% raw %}{{% endraw %}{ index .Config.Labels "build_version" }}' ghcr.io/linuxserver/deluge`

------

## 翻译之外

比较常见的种子下载工具，手头没种子，未来可以尝试一下。

用默认密码 deluge 登录后即可使用，可以切换中文。

![image-20201026112351607](https://pic.watercalmx.com/pic/image-20201026112351607.png)

![image-20201026112446877](https://pic.watercalmx.com/pic/image-20201026112446877.png)