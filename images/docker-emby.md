# linuxserver/emby

> [!TIP]
>
> 前半部分是翻译官方的文档，最后一部分是我的简单试用（个别软件会深度试用），如果对Docker已经有一定的了解了，可以直接跳转到最后面 [翻译之外](#翻译之外) 这部分来查看。

Emby → https://emby.media/

GitHub → https://github.com/linuxserver/docker-emby

Docker Hub → https://hub.docker.com/r/linuxserver/emby

[Emby](https://emby.media/) 可以管理个人媒体库中的视频、音乐、在线电视和照片，并把它们串流到智能电视、串流盒子和移动设备上。该容器打包了独立的 emby Media Server 。

------

## 支持的系统架构

得益于docker的跨平台属性，我们的镜像也支持多架构（如，x86-64、arm64、armhf）。

直接拉取 `ghcr.io/linuxserver/emby` 应该就可以自动获取适合你系统架构的版本，当然你也可以通过 tag 获取特定的版本。

| 架构   | Tag            |
| ------ | -------------- |
| x86-64 | amd64-latest   |
| arm64  | arm64v8-latest |
| armhf  | arm32v7-latest |

### 版本标签

| Tag    | 说明              |
| ------ | ----------------- |
| latest | emby 的稳定发行版 |
| beta   | emby 的beta发行版 |


------

## 拉取镜像

```shell
docker pull ghcr.io/linuxserver/emby
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
  emby:
    image: ghcr.io/linuxserver/emby
    container_name: emby
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
      - UMASK_SET=<022> #optional
    volumes:
      - /path/to/library:/config
      - /path/to/tvshows:/data/tvshows
      - /path/to/movies:/data/movies
      - /path/for/transcoding:/transcode #optional
      - /opt/vc/lib:/opt/vc/lib #optional
    ports:
      - 8096:8096
      - 8920:8920 #optional
    devices:
      - /dev/dri:/dev/dri #optional
      - /dev/vchiq:/dev/vchiq #optional
      - /dev/video10:/dev/video10 #optional
      - /dev/video11:/dev/video11 #optional
      - /dev/video12:/dev/video12 #optional
    restart: unless-stopped
```

### docker cli

```shell
docker run -d \
  --name=emby \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Europe/London \
  -e UMASK_SET=<022> `#optional` \
  -p 8096:8096 \
  -p 8920:8920 `#optional` \
  -v /path/to/library:/config \
  -v /path/to/tvshows:/data/tvshows \
  -v /path/to/movies:/data/movies \
  -v /path/for/transcoding:/transcode `#optional` \
  -v /opt/vc/lib:/opt/vc/lib `#optional` \
  --device /dev/dri:/dev/dri `#optional` \
  --device /dev/vchiq:/dev/vchiq `#optional` \
  --device /dev/video10:/dev/video10 `#optional` \
  --device /dev/video11:/dev/video11 `#optional` \
  --device /dev/video12:/dev/video12 `#optional` \
  --restart unless-stopped \
  ghcr.io/linuxserver/emby
```

## 参数

Docker镜像在使用的时候需要配置一些参数，这些参数使用 `:` 分隔，分别表示 `宿主机:容器内`。例如 `-p 8080:80` 指的是将容器内的`80`端口映射到宿主机上的`8080`端口，即通过宿主机网络访问的话，访问`8080`端口即是访问容器内的`80`端口。

### 端口（`-p`）

| port   | 说明                          |
| ------ | ----------------------------- |
| `8096` | Web界面（HTTP访问）           |
| `8920` | HTTPS访问端口（需要安装证书） |

### 环境变量（`-e`）

| env                | 说明                                       |
| ------------------ | ------------------------------------------ |
| `PUID=1000`        | 用户的 UID，详见下面的说明                 |
| `PGID=1000`        | 用户的 GID，详见下面的说明                 |
| `TZ=Europe/London` | 设置时区，在国内的话可以使用 Asia/Shanghai |
| `UMASK_SET=<022>`  | Emby的umask设置，默认为022                 |

### 卷映射（`-v`）

| volume          | 说明                                                         |
| --------------- | ------------------------------------------------------------ |
| `/config`       | 配置文件所在路径                                             |
| `/data/tvshows` | 媒体文件夹，可以按需要添加，如 `/data/movies`，`/data/tv` 等 |
| `/data/movies`  | 媒体文件夹，可以按需要添加，如 `/data/movies`，`/data/tv` 等 |
| `/transcode`    | （可选）转码文件夹                                           |
| `/opt/vc/lib`   | （可选）Raspberry Pi OpenMAX库的路径                         |

### 设备映射（`--device`）

| 参数          | 说明                                                         |
| --------------- | ------------------------------------------------------------ |
| `/dev/dri` | 如果要使用Intel或AMD GPU硬件加速转码（vaapi）时设置        |
| `/dev/vchiq` | 如果要使用Raspberry Pi OpenMax 转码时设置 |
| `/dev/video10` | 如果要使用Raspberry Pi V4L2 转码时设置 |
| `/dev/video11` | 如果要使用Raspberry Pi V4L2 转码时设置        |
| `/dev/video12` | 如果要使用Raspberry Pi V4L2 转码时设置 |


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

访问 `http://ip:8096` 进入Web管理页面

访问 [这里](https://github.com/MediaBrowser/Wiki/wiki) 获取完整的Emby文档

使用 Intel Quicksync 和 AMD VAAPI 硬件加速的用户，需要在创建容器的时候把 /dev/dri 下的设备映射到容器内：

需要确保容器内的abc用户有权限访问设备.

Nvidia的硬件加速需要到下面的网址中下载运行时:
https://github.com/NVIDIA/nvidia-docker

我们会自动添加必要的环境变量，该变量将利用主机上GPU上的所有可用功能。在主机上安装nvidia-docker之后，需要使用nvidia容器运行时 `--runtime = nvidia` 创建docker容器，并添加环境变量`-e NVIDIA_VISIBLE_DEVICES = all`（也可以设置为特定的GPU的UUID，可以通过运行`nvidia-smi --query-gpu = gpu_name，gpu_uuid --format = csv`来获取） NVIDIA将主机上的GPU和驱动程序自动映射到容器内。



### OpenMAX (Raspberry Pi)

要使用 Raspberry Pi OpenMAX 来进行硬件加速的用户，需要映射 /dev/vchiq下的设备和替换系统中的 OpenMAX 库。在创建或运行容器时用下面这条命令完成上述步骤：

```
--device=/dev/vchiq:/dev/vchiq -v /opt/vc/lib:/opt/vc/lib
```



### V4L2 (Raspberry Pi)

要使用Raspberry Pi V4L2 来进行硬件加速的用户，需要映射 /dev/video1X下的设备。在创建或运行容器时用下面这条命令完成上述步骤：

```
--device=/dev/video10:/dev/video10 --device=/dev/video11:/dev/video11 --device=/dev/video12:/dev/video12
```



------

## 支持

- 进入容器：
  - `docker exec -it emby /bin/bash`
- 查看容器日志：
  - `docker logs -f emby`
- 查看容器版本号：
  - `docker inspect -f '{% raw %}{{% endraw %}{ index .Config.Labels "build_version" }}' emby`
- 查看镜像版本号：
  - `docker inspect -f '{% raw %}{{% endraw %}{ index .Config.Labels "build_version" }}' ghcr.io/linuxserver/emby`

------

## 翻译之外

比较知名的开源软件了，首次打开的时候设置用户名和密码、媒体库等信息

之后，就可以用刚刚的用户名和密码登录了

添加媒体库的时候，支持smb协议，手动输入地址就可以了，这就意味着你可以直接把NAS上的影片弄到emby上。

emby还可以把媒体推送到设备上播放

而且还支持 webhooks，这就意味着可以实现播放电影的时候，家里的灯光等设备可以自动进行变化。

![image-20201104123408879](https://pic.watercalmx.com/pic/image-20201104123408879.png)

![image-20201104124240319](https://pic.watercalmx.com/pic/image-20201104124240319.png)

![image-20201104131155104](https://pic.watercalmx.com/pic/image-20201104131155104.png)

![image-20201104131408756](https://pic.watercalmx.com/pic/image-20201104131408756.png)