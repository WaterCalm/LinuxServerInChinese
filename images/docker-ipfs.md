# linuxserver/ipfs

> [!TIP]
>
> 前半部分是翻译官方的文档，最后一部分是我的简单试用（个别软件会深度试用），如果对Docker已经有一定的了解了，可以直接跳转到最后面 [翻译之外](#翻译之外) 这部分来查看。

Ipfs → https://ipfs.io/

GitHub → https://github.com/linuxserver/docker-ipfs

Docker Hub → https://hub.docker.com/r/linuxserver/ipfs

[Ipfs](https://ipfs.io/) 是一个点对点的超媒体协议，其设置初衷是使网络访问更快速、安全、开放。

------

## 支持的系统架构

得益于docker的跨平台属性，我们的镜像也支持多架构（如，x86-64、arm64、armhf）。

直接拉取 `ghcr.io/linuxserver/ipfs` 应该就可以自动获取适合你系统架构的版本，当然你也可以通过 tag 获取特定的版本。

| 架构   | Tag            |
| ------ | -------------- |
| x86-64 | amd64-latest   |
| arm64  | arm64v8-latest |
| armhf  | arm32v7-latest |



------

## 拉取镜像

```shell
docker pull ghcr.io/linuxserver/ipfs
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
  ipfs:
    image: ghcr.io/linuxserver/ipfs
    container_name: ipfs
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
    volumes:
      - /path/to/data:/config
    ports:
      - 80:80
      - 4001:4001
      - 5001:5001
      - 8080:8080
      - 443:443 #optional
    restart: unless-stopped
```

### docker cli

```shell
docker run -d \
  --name=ipfs \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Europe/London \
  -p 80:80 \
  -p 4001:4001 \
  -p 5001:5001 \
  -p 8080:8080 \
  -p 443:443 `#optional` \
  -v /path/to/data:/config \
  --restart unless-stopped \
  ghcr.io/linuxserver/ipfs
```

## 参数

Docker镜像在使用的时候需要配置一些参数，这些参数使用 `:` 分隔，分别表示 `宿主机:容器内`。例如 `-p 8080:80` 指的是将容器内的`80`端口映射到宿主机上的`8080`端口，即通过宿主机网络访问的话，访问`8080`端口即是访问容器内的`80`端口。

### 端口（`-p`）

| port   | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| `80`   | IPFS 的 HTTP 访问端口                                        |
| `4001` | 点对点端口，这个是你应该开放到互联网上的唯一端口             |
| `5001` | API端口，客户端的webUI需要能够从您的Web浏览器所在的任何计算机上与之进行通信 |
| `8080` | 网关端口，实际提供IPFS内容                                   |
| `443`  | IPFS 的 HTTPS 访问端口                                       |

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

为了将文件推送到本地网关之外，您必须确保端口4001已转发到互联网上。 IPFS对等方需要此权限才能进入并获取您的文件，以便公共网关可以为它们提供服务。

通过 `http:// localhost` 访问 webui，也可以使用API方式访问： `http://192.168.1.10:5001`，在这可以上传和管理推送到的文件IPFS。您访问IPFS文件的网关是 `http:// localhost:8080/ipfs/YOUR-FILE-HASH-HERE`。您也可以简单地使用公共IPFS网关，例如：

- Cloudflare https://cloudflare-ipfs.com/ipfs/YOUR-FILE-HASH-HERE
- IPFS.io https://ipfs.io/ipfs/YOUR-FILE-HASH-HERE
- Eternum.io https://ipfs.eternum.io/ipfs/YOUR-FILE-HASH-HERE

Cloudflare是一个可靠的选择，因为它们实际上在CDN上边缘缓存了文件，因此即使您固定该项目的节点关闭了一段时间，其缓存也将持续长达一个月的时间。

有关使用IPFS的更多信息，请在此处阅读文档：https://docs.ipfs.io/

------

## 支持

- 进入容器：
  - `docker exec -it ipfs /bin/bash`
- 查看容器日志：
  - `docker logs -f ipfs`
- 查看容器版本号：
  - `docker inspect -f '{% raw %}{{% endraw %}{ index .Config.Labels "build_version" }}' ipfs`
- 查看镜像版本号：
  - `docker inspect -f '{% raw %}{{% endraw %}{ index .Config.Labels "build_version" }}' ghcr.io/linuxserver/ipfs`

------

## 翻译之外

暂未试用