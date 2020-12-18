# linuxserver/netbootxyz

> [!TIP]
>
> 前半部分是翻译官方的文档，最后一部分是我的简单试用（个别软件会深度试用），如果对Docker已经有一定的了解了，可以直接跳转到最后面 [翻译之外](#翻译之外) 这部分来查看。

Netbootxyz → https://netboot.xyz/

GitHub → https://github.com/linuxserver/docker-netbootxyz

Docker Hub → https://hub.docker.com/r/linuxserver/netbootxyz

[Netbootxyz](https://netboot.xyz/) 是一种从BIOS中的一个位置PXE引导各种操作系统安装程序或实用程序的方法，而无需去检索介质即可运行该工具。 iPXE用于在BIOS中提供用户友好的菜单，使您可以轻松选择所需的操作系统以及任何特定类型的版本或可启动标志。

------

## 支持的系统架构

得益于docker的跨平台属性，我们的镜像也支持多架构（如，x86-64、arm64、armhf）。

直接拉取 `ghcr.io/linuxserver/netbootxyz` 应该就可以自动获取适合你系统架构的版本，当然你也可以通过 tag 获取特定的版本。

| 架构   | Tag            |
| ------ | -------------- |
| x86-64 | amd64-latest   |
| arm64  | arm64v8-latest |
| armhf  | arm32v7-latest |

### 版本标签

| Tag    | 说明                                  |
| ------ | ------------------------------------- |
| latest | 完全自我托管的Web应用程序             |
| tftp   | 仅具有NETBOOT.XYZ启动文件的TFTP服务器 |



------

## 拉取镜像

```shell
docker pull ghcr.io/linuxserver/netbootxyz
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
  netbootxyz:
    image: ghcr.io/linuxserver/netbootxyz
    container_name: netbootxyz
    environment:
      - PUID=1000
      - PGID=1000
      - MENU_VERSION=1.9.9 #optional
    volumes:
      - /path/to/config:/config
      - /path/to/assets:/assets #optional
    ports:
      - 3000:3000
      - 69:69/udp
      - 8080:80 #optional
    restart: unless-stopped
```

### docker cli

```shell
docker run -d \
  --name=netbootxyz \
  -e PUID=1000 \
  -e PGID=1000 \
  -e MENU_VERSION=1.9.9 `#optional` \
  -p 3000:3000 \
  -p 69:69/udp \
  -p 8080:80 `#optional` \
  -v /path/to/config:/config \
  -v /path/to/assets:/assets `#optional` \
  --restart unless-stopped \
  ghcr.io/linuxserver/netbootxyz
```



------

## 参数

Docker镜像在使用的时候需要配置一些参数，这些参数使用 `:` 分隔，分别表示 `宿主机:容器内`。例如 `-p 8080:80` 指的是将容器内的`80`端口映射到宿主机上的`8080`端口，即通过宿主机网络访问的话，访问`8080`端口即是访问容器内的`80`端口。

### 端口（`-p`）

| port     | 说明                      |
| -------- | ------------------------- |
| `3000`   | WebUI访问端口             |
| `69/udp` | TFTP 端口                 |
| `80`     | NGINX服务器，用于托管资源 |

### 环境变量（`-e`）

| env                  | 说明                                                         |
| -------------------- | ------------------------------------------------------------ |
| `PUID=1000`          | 用户的 UID，详见下面的说明                                   |
| `PGID=1000`          | 用户的 GID，详见下面的说明                                   |
| `TZ=Europe/London`   | 设置时区，在国内的话可以使用 Asia/Shanghai                   |
| `MENU_VERSION=1.9.9` | 从NETBOOT.XYZ指定要使用的启动文件的特定版本（未设置将拉动最新的） |

### 卷映射（`-v`）

| volume    | 说明                                            |
| --------- | ----------------------------------------------- |
| `/config` | 配置文件所在路径                                |
| `/assets` | NETBOOT.XYZ可启动资源的存储（活动CD和其他文件） |




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

详见（比较专业，先不瞎翻译了）：https://docs.linuxserver.io/images/docker-netbootxyz#application-setup

------

## 支持

- 进入容器：
  - `docker exec -it mylar3 /bin/bash`
- 查看容器日志：
  - `docker logs -f mylar3`
- 查看容器版本号：
  - `docker inspect -f '{% raw %}{{% endraw %}{ index .Config.Labels "build_version" }}' mylar3`
- 查看镜像版本号：
  - `docker inspect -f '{% raw %}{{% endraw %}{ index .Config.Labels "build_version" }}' ghcr.io/linuxserver/mylar3`

------

## 翻译之外

暂未试用