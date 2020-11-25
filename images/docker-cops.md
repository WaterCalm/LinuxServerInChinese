# linuxserver/cops

> [!TIP]
>
> 前半部分是翻译官方的文档，最后一部分是我的简单试用（个别软件会深度试用），如果对Docker已经有一定的了解了，可以直接跳转到最后面 [翻译之外](#翻译之外) 这部分来查看。

Cops → http://blog.slucas.fr/en/oss/calibre-opds-php-server

GitHub → https://github.com/linuxserver/docker-cops

Docker Hub → https://hub.docker.com/r/linuxserver/cops

[Cops](http://blog.slucas.fr/en/oss/calibre-opds-php-server) 由 Sébastien Lucas 编写，Calibre OPDS (和 HTML) 的 php服务。 

COPS 链接到Calibre数据库，允许直接从Web浏览器下载或发送到电子邮箱中，斌给提供OPDS流以链接到你的设备上。

Calibre库中的变化会立即更新到COPS页面上。

可以通过官方介绍了解更多：[COPS's home](http://blog.slucas.fr/en/oss/calibre-opds-php-server)，[WIKI](https://github.com/seblucas/cops/wiki)

------

## 为什么要制作该软件（摘取自作者的网站）

在我看来，Caliber是一个了不起的工具，但是太复杂了并且依赖太多其他应用，无法直接作为服务器使用。

这是我制作这个OPDS服务器的主要原因，我需要一个轻量级的应用安装在小型服务器上。

最初，我想到的是Calibre2OPDS，但由于它生成静态文件，导致无法直接搜索。

后来，我添加了一个简单的HTML目录，该目录可以在我的Kobo上使用。

所有，COPS的主要优点是：

- 不需要太多的依赖
- 不需要大量的CPU和RAM
- 没有过多的代码
- 可以轻松检索
- 借助Dropbox/ownCloud，可以轻松更新OPDS服务器
- 编程很有趣

如果你想要使用 OPDS feed，不要忘记在URL的末尾加上 feed.php

------

## 支持的系统架构

得益于docker的跨平台属性，我们的镜像也支持多架构（如，x86-64、arm64、armhf）。

直接拉取 `ghcr.io/linuxserver/cops` 应该就可以自动获取适合你系统架构的版本，当然你也可以通过 tag 获取特定的版本。

| 架构   | Tag            |
| ------ | -------------- |
| x86-64 | amd64-latest   |
| arm64  | arm64v8-latest |
| armhf  | arm32v7-latest |


------

## 拉取镜像

```shell
docker pull ghcr.io/linuxserver/cops
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
  cops:
    image: linuxserver/cops
    container_name: cops
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
    volumes:
      - <path to data>:/config
      - <path to data>:/books
    ports:
      - 80:80
    restart: unless-stopped
```

### docker cli

```shell
docker run -d \
  --name=cops \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Europe/London \
  -p 80:80 \
  -v <path to data>:/config \
  -v <path to data>:/books \
  --restart unless-stopped \
  linuxserver/cops
```

## 参数

Docker镜像在使用的时候需要配置一些参数，这些参数使用 `:` 分隔，分别表示 `宿主机:容器内`。例如 `-p 8080:80` 指的是将容器内的`80`端口映射到宿主机上的`8080`端口，即通过宿主机网络访问的话，访问`8080`端口即是访问容器内的`80`端口。

### 端口（`-p`）

| port | 说明    |
| ---- | ------- |
| `80` | WEB界面 |

### 环境变量（`-e`）

| env                | 说明                                       |
| ------------------ | ------------------------------------------ |
| `PUID=1000`        | 用户的 UID，详见下面的说明                 |
| `PGID=1000`        | 用户的 GID，详见下面的说明                 |
| `TZ=Europe/London` | 设置时区，在国内的话可以使用 Asia/Shanghai |

### 卷映射（`-v`）

| volume    | 说明                       |
| --------- | -------------------------- |
| `/config` | 配置文件所在路径           |
| `/books`  | Calibre的 metadata.db 位置 |

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

与其他COPS镜像不同，LinuxServer的版本可以直接来修改 `/config` 下面的 `config_local.php` 文件来根据你的需要进行自定义安装，包括设置电子邮件等，还包括可以直接在浏览器中观看epub图书的依赖。

------

## 支持

- 进入容器：
  - `docker exec -it cops /bin/bash`
- 查看容器日志：
  - `docker logs -f cops`
- 查看容器版本号：
  - `docker inspect -f '{% raw %}{{% endraw %}{ index .Config.Labels "build_version" }}' cops`
- 查看镜像版本号：
  - `docker inspect -f '{% raw %}{{% endraw %}{ index .Config.Labels "build_version" }}' ghcr.io/linuxserver/cops`

------

## 翻译之外

> [!NOTE]
>
> 这个镜像的应用需要已经有一个已经存在的Calibre数据库！如果没有的话可以使用 `linuxserver/calibre` 镜像创建一个，或者下载 Calibre 的客户端创建一个。最终只要将数据库的目录映射给容器里的`/books`目录即可。

与之类似的镜像有 [linuxserver/calibre-web](images/docker-calibre-web.md) ，相比来说 Calibre-web 的界面更美观，但 COPS 支持在线浏览书籍而 Calibre-web 却不能。也许COPS更主要的功能是 OPDS 服务器，但自己对这方面了解较少，而且手边也没有相关的设备，也做不了试用……


![image-20201025155956215](https://pic.watercalmx.com/pic/image-20201025155956215.png)

![image-20201025160029384](https://pic.watercalmx.com/pic/image-20201025160029384.png)