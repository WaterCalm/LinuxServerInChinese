# linuxserver/nano

> [!TIP]
>
> 前半部分是翻译官方的文档，最后一部分是我的简单试用（个别软件会深度试用），如果对Docker已经有一定的了解了，可以直接跳转到最后面 [翻译之外](#翻译之外) 这部分来查看。

Nano → https://nano.org/

GitHub → https://github.com/linuxserver/docker-nano

Docker Hub → https://hub.docker.com/r/linuxserver/nano

[Nano](https://nano.org/) 是一种数字支付协议，旨在实现轻巧易用，重点是消除其他加密货币中存在的低效率问题。通过超快的交易和安全，绿色和分散式网络上的零费用，这使Nano成为日常交易的理想选择。

------

## 支持的系统架构

得益于docker的跨平台属性，我们的镜像也支持多架构（如，x86-64、arm64、armhf）。

直接拉取 `ghcr.io/linuxserver/nano` 应该就可以自动获取适合你系统架构的版本，当然你也可以通过 tag 获取特定的版本。

| 架构   | Tag            |
| ------ | -------------- |
| x86-64 | amd64-latest   |
| arm64  | arm64v8-latest |

### 版本标签

| Tag    | 说明            |
| ------ | --------------- |
| latest | Nano 稳定发行版 |
| beta   | Nano Beta发行版 |



------

## 拉取镜像

```shell
docker pull ghcr.io/linuxserver/nano
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
  nano:
    image: ghcr.io/linuxserver/nano
    container_name: nano
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
      - PEER_HOST=localhost #optional
      - LIVE_GENESIS_PUB=GENESIS_PUBLIC #optional
      - LIVE_GENESIS_ACCOUNT=nano_xxxxxx #optional
      - LIVE_GENESIS_WORK=WORK_FOR_BLOCK #optional
      - LIVE_GENESIS_SIG=BLOCK_SIGNATURE #optional
      - CLI_OPTIONS=--config node.enable_voting=true #optional
      - LMDB_BOOTSTRAP_URL=http://example.com/Nano_64_version_20.7z #optional
    volumes:
      - /path/to/data:/config
    ports:
      - 8075:8075
      - 7076:3000
      - 7077:3001
    restart: unless-stopped
```

### docker cli

```shell
docker run -d \
  --name=nano \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Europe/London \
  -e PEER_HOST=localhost `#optional` \
  -e LIVE_GENESIS_PUB=GENESIS_PUBLIC `#optional` \
  -e LIVE_GENESIS_ACCOUNT=nano_xxxxxx `#optional` \
  -e LIVE_GENESIS_WORK=WORK_FOR_BLOCK `#optional` \
  -e LIVE_GENESIS_SIG=BLOCK_SIGNATURE `#optional` \
  -e CLI_OPTIONS=--config node.enable_voting=true `#optional` \
  -e LMDB_BOOTSTRAP_URL=http://example.com/Nano_64_version_20.7z `#optional` \
  -p 8075:8075 \
  -p 7076:3000 \
  -p 7077:3001 \
  -v /path/to/data:/config \
  --restart unless-stopped \
  ghcr.io/linuxserver/nano
```



------

## 参数

Docker镜像在使用的时候需要配置一些参数，这些参数使用 `:` 分隔，分别表示 `宿主机:容器内`。例如 `-p 8080:80` 指的是将容器内的`80`端口映射到宿主机上的`8080`端口，即通过宿主机网络访问的话，访问`8080`端口即是访问容器内的`80`端口。

### 端口（`-p`）

| port   | 说明                        |
| ------ | --------------------------- |
| `8075` | Nano 通讯端口               |
| `3000` | 通过代理过滤的RPC端口       |
| `3001` | 通过代理过滤的HTTPS RPC端口 |

### 环境变量（`-e`）

| env                                                          | 说明                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `PUID=1000`                                                  | 用户的 UID，详见下面的说明                                   |
| `PGID=1000`                                                  | 用户的 GID，详见下面的说明                                   |
| `TZ=Europe/London`                                           | 设置时区，在国内的话可以使用 Asia/Shanghai                   |
| `PEER_HOST=localhost`                                        | 默认的对等主机（可以通过命令行选项用数组覆盖）               |
| `LIVE_GENESIS_PUB=GENESIS_PUBLIC`                            | 创世块公钥                                                   |
| `LIVE_GENESIS_ACCOUNT=nano_xxxxxx`                           | 创世块帐户                                                   |
| `LIVE_GENESIS_WORK=WORK_FOR_BLOCK`                           | 创世块工作证明                                               |
| `LIVE_GENESIS_SIG=BLOCK_SIGNATURE`                           | 创世块签名                                                   |
| `CLI_OPTIONS=--config node.enable_voting=true`               | 节点运行命令cli args                                         |
| `LMDB_BOOTSTRAP_URL=http://example.com/Nano_64_version_20.7z` | HTTP / HTTPS端点，用于下载带有data.ldb的7z文件以引导至该节点 |

### 卷映射（`-v`）

| volume       | 说明             |
| ------------ | ---------------- |
| `/config`    | 配置文件所在路径 |
| `/comics`    | 漫画文件夹       |
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

详见（比较专业，先不瞎翻译了）：https://docs.linuxserver.io/images/docker-nano#application-setup

------

## 支持

- 进入容器：
  - `docker exec -it nano /bin/bash`
- 查看容器日志：
  - `docker logs -f nano`
- 查看容器版本号：
  - `docker inspect -f '{% raw %}{{% endraw %}{ index .Config.Labels "build_version" }}' nano`
- 查看镜像版本号：
  - `docker inspect -f '{% raw %}{{% endraw %}{ index .Config.Labels "build_version" }}' ghcr.io/linuxserver/nano`

------

## 翻译之外

暂未试用