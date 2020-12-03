# linuxserver/musicbrainz

> [!TIP]
>
> 前半部分是翻译官方的文档，最后一部分是我的简单试用（个别软件会深度试用），如果对Docker已经有一定的了解了，可以直接跳转到最后面 [翻译之外](#翻译之外) 这部分来查看。

MusicBrainz → https://musicbrainz.org/

GitHub → https://github.com/linuxserver/docker-musicbrainz

Docker Hub → https://hub.docker.com/r/linuxserver/musicbrainz

[MusicBrainz](https://musicbrainz.org/) 是一个开放的音乐百科全书，收集音乐元数据并将其提供给公众。

------

## 支持的系统架构

得益于docker的跨平台属性，我们的镜像也支持多架构（如，x86-64、arm64、armhf）。

直接拉取 `ghcr.io/linuxserver/musicbrainz` 应该就可以自动获取适合你系统架构的版本，当然你也可以通过 tag 获取特定的版本。

| 架构   | Tag            |
| ------ | -------------- |
| x86-64 | amd64-latest   |
| arm64  | arm64v8-latest |
| armhf  | arm32v7-latest |



------

## 拉取镜像

```shell
docker pull ghcr.io/linuxserver/musicbrainz
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
  musicbrainz:
    image: ghcr.io/linuxserver/musicbrainz
    container_name: musicbrainz
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
      - BRAINZCODE=<code from MusicBrainz>
      - WEBADDRESS=<ip of host>
      - NPROC=<parameter> #optional
    volumes:
      - </path/to/appdata/config>:/config
      - </path/to/appdata/config>:/data
    ports:
      - 5000:5000
    restart: unless-stopped
```

### docker cli

```shell
docker run -d \
  --name=musicbrainz \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Europe/London \
  -e BRAINZCODE=<code from MusicBrainz> \
  -e WEBADDRESS=<ip of host> \
  -e NPROC=<parameter> `#optional` \
  -p 5000:5000 \
  -v </path/to/appdata/config>:/config \
  -v </path/to/appdata/config>:/data \
  --restart unless-stopped \
  ghcr.io/linuxserver/musicbrainz
```



------

## 参数

Docker镜像在使用的时候需要配置一些参数，这些参数使用 `:` 分隔，分别表示 `宿主机:容器内`。例如 `-p 8080:80` 指的是将容器内的`80`端口映射到宿主机上的`8080`端口，即通过宿主机网络访问的话，访问`8080`端口即是访问容器内的`80`端口。

### 端口（`-p`）

| port   | 说明          |
| ------ | ------------- |
| `5000` | WebUI访问端口 |

### 环境变量（`-e`）

| env                                  | 说明                                        |
| ------------------------------------ | ------------------------------------------- |
| `PUID=1000`                          | 用户的 UID，详见下面的说明                  |
| `PGID=1000`                          | 用户的 GID，详见下面的说明                  |
| `TZ=Europe/London`                   | 设置时区，在国内的话可以使用 Asia/Shanghai  |
| `BRAINZCODE=<code from MusicBrainz>` | 输入MusicBrainz代码。请参阅设置应用程序     |
| `WEBADDRESS=<ip of host>`            | 设置主机ip以正常显示css，请不要输入端口号。 |
| `NPROC=<parameter>`                  | 设置处理数量，如果未设置，则默认为5。       |

### 卷映射（`-v`）

| volume    | 说明                    |
| --------- | ----------------------- |
| `/config` | 配置文件所在路径        |
| `/data`   | musicbrainz的数据文件夹 |




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

- 对于所有更新，您应该提取最新的映像，清除 `/config` 和 `/data` 中的所有文件和文件夹，并通过（重新）启动docker重新启动数据库导入。我们不正式支持使用现有数据集在适当位置升级此容器。
- 如果未设置 `WEBADDRESS` 环境变量，则在完成初始化之后，您需要编辑 `/config/DBDefs.pm` 文件中的 `sub WEB_SERVER { "localhost:5000" }`，将 `localhost` 更改为主机的 `ip` ，以用来正常显示css
- 必须在注册才能接收 MusicBrainz 代码并允许您接收数据库更新，它是免费的：https://metabrainz.org/supporters/account-type
- 数据库的初始导入和设置可能会花费很长时间，具体取决于您的下载速度等，请耐心等待并且不要在完成容器之前重新启动容器。
- 使用 `/mnt/cache/appdata` 时可能有些问题，可以使用 `/mnt/user/cache/appdata` 代替它

------

## 支持

- 进入容器：
  - `docker exec -it musicbrainz /bin/bash`
- 查看容器日志：
  - `docker logs -f musicbrainz`
- 查看容器版本号：
  - `docker inspect -f '{% raw %}{{% endraw %}{ index .Config.Labels "build_version" }}' musicbrainz`
- 查看镜像版本号：
  - `docker inspect -f '{% raw %}{{% endraw %}{ index .Config.Labels "build_version" }}' ghcr.io/linuxserver/musicbrainz`

------

## 翻译之外

暂未试用