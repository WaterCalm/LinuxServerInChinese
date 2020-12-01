# linuxserver/lidarr

> [!TIP]
>
> 前半部分是翻译官方的文档，最后一部分是我的简单试用（个别软件会深度试用），如果对Docker已经有一定的了解了，可以直接跳转到最后面 [翻译之外](#翻译之外) 这部分来查看。

Lidarr → https://github.com/lidarr/Lidarr

GitHub → https://github.com/linuxserver/docker-lidarr

Docker Hub → https://hub.docker.com/r/linuxserver/lidarr

[Lidarr](https://github.com/lidarr/Lidarr) 是 Usenet 和 BitTorrent 的音乐收集管理器。它可以监视多个 RSS feed，以查找您最喜欢的艺术家的新曲目，并对其进行抓取、排序和重命名。它还可以配置为自动升级更高质量的音频文件。

------

## 支持的系统架构

得益于docker的跨平台属性，我们的镜像也支持多架构（如，x86-64、arm64、armhf）。

直接拉取 `ghcr.io/linuxserver/lidarr` 应该就可以自动获取适合你系统架构的版本，当然你也可以通过 tag 获取特定的版本。

| 架构   | Tag            |
| ------ | -------------- |
| x86-64 | amd64-latest   |
| arm64  | arm64v8-latest |
| armhf  | arm32v7-latest |

## 版本标签

| Tag     | 描述       |
| ------- | ---------- |
| latest  | 稳定发行版 |
| nightly | 测试版     |



------

## 拉取镜像

```shell
docker pull ghcr.io/linuxserver/lidarr
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
  lidarr:
    image: ghcr.io/linuxserver/lidarr
    container_name: lidarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
      - UMASK_SET=022 #optional
    volumes:
      - /path/to/appdata/config:/config
      - /path/to/music:/music
      - /path/to/downloads:/downloads
    ports:
      - 8686:8686
    restart: unless-stopped
```

### docker cli

```shell
docker run -d \
  --name=lidarr \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Europe/London \
  -e UMASK_SET=022 `#optional` \
  -p 8686:8686 \
  -v /path/to/appdata/config:/config \
  -v /path/to/music:/music \
  -v /path/to/downloads:/downloads \
  --restart unless-stopped \
  ghcr.io/linuxserver/lidarr
```

## 参数

Docker镜像在使用的时候需要配置一些参数，这些参数使用 `:` 分隔，分别表示 `宿主机:容器内`。例如 `-p 8080:80` 指的是将容器内的`80`端口映射到宿主机上的`8080`端口，即通过宿主机网络访问的话，访问`8080`端口即是访问容器内的`80`端口。

### 端口（`-p`）

| port   | 说明          |
| ------ | ------------- |
| `8686` | WebUI访问端口 |

### 环境变量（`-e`）

| env                | 说明                                       |
| ------------------ | ------------------------------------------ |
| `PUID=1000`        | 用户的 UID，详见下面的说明                 |
| `PGID=1000`        | 用户的 GID，详见下面的说明                 |
| `TZ=Europe/London` | 设置时区，在国内的话可以使用 Asia/Shanghai |
| `UMASK_SET=022`    | 文件的Umask设置，默认为022                 |

### 卷映射（`-v`）

| volume       | 说明             |
| ------------ | ---------------- |
| `/config`    | 配置文件所在路径 |
| `/music`     | 音乐文件夹       |
| `/downloads` | 下载文件夹       |




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

访问地址：`http://ip:8686`，更多信息可以查看：https://github.com/lidarr/Lidarr

特别注意：由于我们的文件夹结构不同，因此遵循我们当前的文件夹结构将无法从下载内容硬链接到“音乐”文件夹。为了支持硬链接，只要确保“音乐”和“下载”数据在单个卷上即可。例如，如果您有 `/mnt/storage/Music` 和 `/mnt/storage/downloads/completed/Music` 这样的文件夹，则需要 `/mnt/storage:/media` 来映射卷，然后可以使用 `/media/downloads/completed` 硬链接到 `/media/Music`。

另外一点是，在 lidarr 本身中，您应该将下载客户端的下载文件夹映射到 lidarr 文件夹中 ：`Settings -> Download Client -> advanced -> remote path mappings`。比如，我输入了下载客户端的下载路径为 `/downloads/Music`（内部容器路径），相对应的本地路径为 `/media/downloads/completed/Music`。

------

## 支持

- 进入容器：
  - `docker exec -it lidarr /bin/bash`
- 查看容器日志：
  - `docker logs -f lidarr`
- 查看容器版本号：
  - `docker inspect -f '{% raw %}{{% endraw %}{ index .Config.Labels "build_version" }}' lidarr`
- 查看镜像版本号：
  - `docker inspect -f '{% raw %}{{% endraw %}{ index .Config.Labels "build_version" }}' ghcr.io/linuxserver/lidarr`

------

## 翻译之外

暂未试用