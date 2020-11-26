# linuxserver/jackett

> [!TIP]
>
> 前半部分是翻译官方的文档，最后一部分是我的简单试用（个别软件会深度试用），如果对Docker已经有一定的了解了，可以直接跳转到最后面 [翻译之外](#翻译之外) 这部分来查看。

Jackett → https://github.com/Jackett/Jackett

GitHub → https://github.com/linuxserver/docker-jackett

Docker Hub → https://hub.docker.com/r/linuxserver/jackett

[Jackett](https://github.com/Jackett/Jackett) 充当代理服务器：它将来自应用程序（Sonarr、SickRage、CouchPotato、Mylar等）的查询转换为特定于跟踪器站点的http查询，解析html响应，然后将结果发送回请求软件。这样可以获取最近上传的内容（例如RSS）并执行搜索。 Jackett是维护索引器抓取和翻译逻辑的单一存储库-减轻了其他应用程序的负担。

------

## 支持的系统架构

得益于docker的跨平台属性，我们的镜像也支持多架构（如，x86-64、arm64、armhf）。

直接拉取 `ghcr.io/linuxserver/jackett` 应该就可以自动获取适合你系统架构的版本，当然你也可以通过 tag 获取特定的版本。

| 架构   | Tag            |
| ------ | -------------- |
| x86-64 | amd64-latest   |
| arm64  | arm64v8-latest |
| armhf  | arm32v7-latest |

## 版本标签

| Tag         | 描述       |
| ----------- | ---------- |
| latest      | 稳定发行版 |
| development | 最新版本   |



------

## 拉取镜像

```shell
docker pull ghcr.io/linuxserver/jackett
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
  jackett:
    image: ghcr.io/linuxserver/jackett
    container_name: jackett
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
      - AUTO_UPDATE=true #optional
      - RUN_OPTS=<run options here> #optional
    volumes:
      - <path to data>:/config
      - <path to blackhole>:/downloads
    ports:
      - 9117:9117
    restart: unless-stopped
```

### docker cli

```shell
docker run -d \
  --name=jackett \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Europe/London \
  -e AUTO_UPDATE=true `#optional` \
  -e RUN_OPTS=<run options here> `#optional` \
  -p 9117:9117 \
  -v <path to data>:/config \
  -v <path to blackhole>:/downloads \
  --restart unless-stopped \
  ghcr.io/linuxserver/jackett
```

## 参数

Docker镜像在使用的时候需要配置一些参数，这些参数使用 `:` 分隔，分别表示 `宿主机:容器内`。例如 `-p 8080:80` 指的是将容器内的`80`端口映射到宿主机上的`8080`端口，即通过宿主机网络访问的话，访问`8080`端口即是访问容器内的`80`端口。

### 端口（`-p`）

| port   | 说明          |
| ------ | ------------- |
| `9117` | WebUI访问端口 |

### 环境变量（`-e`）

| env                           | 说明                                                         |
| ----------------------------- | ------------------------------------------------------------ |
| `PUID=1000`                   | 用户的 UID，详见下面的说明                                   |
| `PGID=1000`                   | 用户的 GID，详见下面的说明                                   |
| `TZ=Europe/London`            | 设置时区，在国内的话可以使用 Asia/Shanghai                   |
| `AUTO_UPDATE=true`            | 允许Jackett在容器内部更新（Jackett当前建议并默认启用）       |
| `RUN_OPTS=<run options here>` | （可选）指定要传递的其他参数。（如，`--ProxyConnection=10.0.0.100:1234`） |

### 卷映射（`-v`）

| volume       | 说明             |
| ------------ | ---------------- |
| `/config`    | 配置文件所在路径 |
| `/downloads` | 下载目录         |




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

访问地址：`http://ip:9117`

访问上面的地址，然后配置各种跟踪器以及与其他应用程序的链接。更多信息请查看：https://github.com/Jackett/Jackett

在WebUI中禁用自动更新，可以防止 jeckett 崩溃，当新版本发布时镜像也会更新。

------

## 支持

- 进入容器：
  - `docker exec -it jackett /bin/bash`
- 查看容器日志：
  - `docker logs -f jackett`
- 查看容器版本号：
  - `docker inspect -f '{% raw %}{{% endraw %}{ index .Config.Labels "build_version" }}' jackett`
- 查看镜像版本号：
  - `docker inspect -f '{% raw %}{{% endraw %}{ index .Config.Labels "build_version" }}' ghcr.io/linuxserver/jackett`

------

## 翻译之外

暂未试用