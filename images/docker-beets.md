# linuxserver/beets

> [!TIP]
>
> 前半部分是翻译官方的文档，最后一部分是我的简单试用（个别软件会深度试用），如果对Docker已经有一定的了解了，可以直接跳转到最后面 [翻译之外](#翻译之外) 这部分来查看。

Beets → http://beets.io/

GitHub → https://github.com/linuxserver/docker-beets

Docker Hub → https://hub.docker.com/r/linuxserver/beets

[Beets](http://beets.io/) 是一个音乐库管理工具，而不仅仅是一个音乐播放器。它包含一个简单的播放器插件和网页播放器，但它更主要的功能是将音乐发送给专业播放设备。

------

## 支持的系统架构

得益于docker的跨平台属性，我们的镜像也支持多架构（如，x86-64、arm64、armhf）。

直接拉取 `linuxserver/beets` 应该就可以自动获取适合你系统架构的版本，当然你也可以通过 tag 获取特定的版本。

| 架构   | Tag            |
| ------ | -------------- |
| x86-64 | amd64-latest   |
| arm64  | arm64v8-latest |
| armhf  | arm32v7-latest |

------

## 版本标签

该镜像可通过不同tag获取不同的版本。最新的tag通常提供了最新的稳定版本，其他的可能是正在开发的版本，需要谨慎使用。

| Tag     | 说明                                                         |
| ------- | ------------------------------------------------------------ |
| latest  | Beets的稳定发行版                                            |
| nightly | 基于最新的Beets的git仓库而构建的，可能是不稳定的版本，但可能是适合高级用户使用的版本 |

------

## 拉取镜像

```shell
docker pull linuxserver/beets
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
  beets:
    image: linuxserver/beets
    container_name: beets
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
    volumes:
      - </path/to/appdata/config>:/config
      - </path/to/music/library>:/music
      - </path/to/ingest>:/downloads
    ports:
      - 8337:8337
    restart: unless-stopped
```

### docker cli

```shell
docker run -d \
  --name=beets \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Europe/London \
  -p 8337:8337 \
  -v </path/to/appdata/config>:/config \
  -v </path/to/music/library>:/music \
  -v </path/to/ingest>:/downloads \
  --restart unless-stopped \
  linuxserver/beets
```

## 参数

Docker镜像在使用的时候需要配置一些参数，这些参数使用 `:` 分隔，分别表示 `宿主机:容器内`。例如 `-p 8080:80` 指的是将容器内的`80`端口映射到宿主机上的`8080`端口，即通过宿主机网络访问的话，访问`8080`端口即是访问容器内的`80`端口。

### 端口（`-p`）

| port   | 说明        |
| ------ | ----------- |
| `8337` | Web管理界面 |

### 环境变量（`-e`）

| env                | 说明                                       |
| ------------------ | ------------------------------------------ |
| `PUID=1000`        | 用户的 UID，详见下面的说明                 |
| `PGID=1000`        | 用户的 GID，详见下面的说明                 |
| `TZ=Europe/London` | 设置时区，在国内的话可以使用 Asia/Shanghai |

### 卷映射（`-v`）

| volume       | 说明         |
| ------------ | ------------ |
| `/config`    | 配置文件     |
| `/music`     | 音乐库       |
| `/downloads` | 未处理的音乐 |

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

无需特殊设置，启动容器后即可使用

------

## 支持

- 进入容器：
  - `docker exec -it beets /bin/bash`
- 查看容器日志：
  - `docker logs -f beets`
- 查看容器版本号：
  - `docker inspect -f '{% raw %}{{% endraw %}{ index .Config.Labels "build_version" }}' beets`
- 查看镜像版本号：
  - `docker inspect -f '{% raw %}{{% endraw %}{ index .Config.Labels "build_version" }}' linuxserver/beets`

------

## 翻译之外

emmm……说实话……用了半天没弄明白怎么用。

打开WebUI是空白的……然后查了官网的文档，看都是用命令行操作的。也试了几首音乐但都无法导入……

看官网的介绍该工具是给痴迷的音乐发烧友用的……可能我不够痴迷和发烧吧……所以……体验失败……

![image-20201020132550745](https://pic.watercalmx.com/pic/image-20201020132550745.png)