# linuxserver/dokuwiki

> [!TIP]
>
> 前半部分是翻译官方的文档，最后一部分是我的简单试用（个别软件会深度试用），如果对Docker已经有一定的了解了，可以直接跳转到最后面 [翻译之外](#翻译之外) 这部分来查看。

Dokuwiki → https://www.dokuwiki.org/dokuwiki/

GitHub → https://github.com/linuxserver/docker-dokuwiki

Docker Hub → https://hub.docker.com/r/linuxserver/dokuwiki

[Dokuwiki](https://www.dokuwiki.org/dokuwiki/) 是一种易于使用且用途广泛的开源Wiki软件，且它不需要数据库。它以其清晰易读的语法受到用户的喜爱。易于维护、备份和集成使其成为管理员的最爱。内置的访问控制和身份验证连接器使DokuWiki在企业环境中特别有用，并且拥有活跃的社区、大量的插件。

------

## 支持的系统架构

得益于docker的跨平台属性，我们的镜像也支持多架构（如，x86-64、arm64、armhf）。

直接拉取 `ghcr.io/linuxserver/dokuwiki` 应该就可以自动获取适合你系统架构的版本，当然你也可以通过 tag 获取特定的版本。

| 架构   | Tag            |
| ------ | -------------- |
| x86-64 | amd64-latest   |
| arm64  | arm64v8-latest |
| armhf  | arm32v7-latest |


------

## 拉取镜像

```shell
docker pull ghcr.io/linuxserver/dokuwiki
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
  dokuwiki:
    image: linuxserver/dokuwiki
    container_name: dokuwiki
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
    volumes:
      - /path/to/appdata/config:/config
    ports:
      - 80:80
      - 443:443 #optional
    restart: unless-stopped
```

### docker cli

```shell
docker run -d \
  --name=dokuwiki \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Europe/London \
  -p 80:80 \
  -p 443:443 `#optional` \
  -v /path/to/appdata/config:/config \
  --restart unless-stopped \
  linuxserver/dokuwiki
```

## 参数

Docker镜像在使用的时候需要配置一些参数，这些参数使用 `:` 分隔，分别表示 `宿主机:容器内`。例如 `-p 8080:80` 指的是将容器内的`80`端口映射到宿主机上的`8080`端口，即通过宿主机网络访问的话，访问`8080`端口即是访问容器内的`80`端口。

### 端口（`-p`）

| port  | 说明              |
| ----- | ----------------- |
| `80`  | HTTP端口          |
| `443` | HTTPS端口（可选） |

### 环境变量（`-e`）

| env                | 说明                                       |
| ------------------ | ------------------------------------------ |
| `PUID=1000`        | 用户的 UID，详见下面的说明                 |
| `PGID=1000`        | 用户的 GID，详见下面的说明                 |
| `TZ=Europe/London` | 设置时区，在国内的话可以使用 Asia/Shanghai |

### 卷映射（`-v`）

| volume    | 说明             |
| --------- | ---------------- |
| `/config` | 配置文件所在路径 |

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

首次启动容器后，先访问 `http://ip:port/install.php` 进行配置，完成之后重启容器。

之后使用 `admin` 用户登录，并在 `admin/Configuration Settings` 面板设置 URL 相关配置。详细信息可以查看 → [官方文档](https://www.dokuwiki.org/rewrite)。

之后便可以在 `http://ip:port` 访问dokuwiki，可以查看[官方文档](https://www.dokuwiki.org/dokuwiki/)了解更多。

------

## 支持

- 进入容器：
  - `docker exec -it dokuwiki /bin/bash`
- 查看容器日志：
  - `docker logs -f dokuwiki`
- 查看容器版本号：
  - `docker inspect -f '{% raw %}{{% endraw %}{ index .Config.Labels "build_version" }}' dokuwiki`
- 查看镜像版本号：
  - `docker inspect -f '{% raw %}{{% endraw %}{ index .Config.Labels "build_version" }}' ghcr.io/linuxserver/dokuwiki`

------

## 翻译之外

DokuWiki应该比较知名了吧，在知乎上让大家推荐好用的Wiki系统，一半以上的人会推荐DokuWiki。它最大的优势就是足够轻量级，而且不需要数据库。所有的文档都是存储在本地的 txt 文件中，所以很方便迁移和本地保存。

虽然默认版本比较捡漏，但可以去安装插件让它丰富起来。

![image-20201028112811222](https://pic.watercalmx.com/pic/image-20201028112811222.png)

![image-20201028113534105](https://pic.watercalmx.com/pic/image-20201028113534105.png)