# linuxserver/booksonic-air

> [!TIP]
>
> 前半部分是翻译官方的文档，最后一部分是我的简单试用（个别软件会深度试用），如果对Docker已经有一定的了解了，可以直接跳转到最后面 [翻译之外](#翻译之外) 这部分来查看。

Booksonic-air → http://booksonic.org/

GitHub → https://github.com/linuxserver/docker-booksonic-air

Docker Hub → https://hub.docker.com/r/linuxserver/booksonic-air

[Booksonic-air](http://booksonic.org/) 是一个可以随时随地访问你的有声读物的平台。该平台由用于流式传输有声读物的服务器 Booksonic Air组成，是基于Booksonic服务器发展而来的。Booksonic App是基于 DSub 的安卓App，用来连接到Booksonic-Air服务器。

------

## 支持的系统架构

得益于docker的跨平台属性，我们的镜像也支持多架构（如，x86-64、arm64、armhf）。

直接拉取 `linuxserver/booksonic-air` 应该就可以自动获取适合你系统架构的版本，当然你也可以通过 tag 获取特定的版本。

| 架构   | Tag            |
| ------ | -------------- |
| x86-64 | amd64-latest   |
| arm64  | arm64v8-latest |
| armhf  | arm32v7-latest |

------

## 版本标签

该镜像可通过不同tag获取不同的版本。最新的tag通常提供了最新的稳定版本，其他的可能是正在开发的版本，需要谨慎使用。

| Tag    | 说明                      |
| ------ | ------------------------- |
| latest | booksonic-air的稳定发行版 |

------

## 拉取镜像

```shell
docker pull linuxserver/booksonic-air
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
  booksonic-air:
    image: linuxserver/booksonic-air
    container_name: booksonic-air
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
      - CONTEXT_PATH=url-base
    volumes:
      - </path/to/appdata/config>:/config
      - </path/to/audiobooks>:/audiobooks
      - </path/to/podcasts>:/podcasts
      - </path/to/othermedia>:/othermedia
    ports:
      - 4040:4040
    restart: unless-stopped
```

### docker cli

```shell
docker create \
  --name=booksonic-air \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Europe/London \
  -e CONTEXT_PATH=url-base \
  -p 4040:4040 \
  -v </path/to/appdata/config>:/config \
  -v </path/to/audiobooks>:/audiobooks \
  -v </path/to/podcasts>:/podcasts \
  -v </path/to/othermedia>:/othermedia \
  --restart unless-stopped \
  linuxserver/booksonic-air
```

## 参数

Docker镜像在使用的时候需要配置一些参数，这些参数使用 `:` 分隔，分别表示 `宿主机:容器内`。例如 `-p 8080:80` 指的是将容器内的`80`端口映射到宿主机上的`8080`端口，即通过宿主机网络访问的话，访问`8080`端口即是访问容器内的`80`端口。

### 端口（`-p`）

| port   | 说明          |
| ------ | ------------- |
| `4040` | 应用的Web界面 |

### 环境变量（`-e`）

| env                     | 说明                                       |
| ----------------------- | ------------------------------------------ |
| `PUID=1000`             | 用户的 UID，详见下面的说明                 |
| `PGID=1000`             | 用户的 GID，详见下面的说明                 |
| `TZ=Europe/London`      | 设置时区，在国内的话可以使用 Asia/Shanghai |
| `CONTEXT_PATH=url-base` | 用于设置反向代理等功能的URL                |

### 卷映射（`-v`）

| volume        | 说明         |
| ------------- | ------------ |
| `/config`     | 配置文件     |
| `/audiobooks` | 有声读物     |
| `/podcasts`   | 播客         |
| `/othermedia` | 其他媒体文件 |

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

该镜像不支持自动升级，建议通过重新安装来升级服务器。默认用户名和密码都是 `admin` 。

------

## 支持

- 进入容器：
  - `docker exec -it booksonic-air /bin/bash`
- 查看容器日志：
  - `docker logs -f booksonic-air`
- 查看容器版本号：
  - `docker inspect -f '{% raw %}{{% endraw %}{ index .Config.Labels "build_version" }}' booksonic-air`
- 查看镜像版本号：
  - `docker inspect -f '{% raw %}{{% endraw %}{ index .Config.Labels "build_version" }}' linuxserver/booksonic-air`

------

## 翻译之外

如果不使用反向代理的话，可以先不设置 `CONTEXT_PATH` ，直接用IP访问，另外在设置这个变量的时候不需要带端口，否则可能会启动服务失败。

界面大家都很熟悉吧？ 跟 Airsonic 是一样的，但在设置里原生却没有英文，使用方法都是一样的。

![image-20201021112628813](https://pic.watercalmx.com/pic/image-20201021112628813.png)

![image-20201021112920694](https://pic.watercalmx.com/pic/image-20201021112920694.png)