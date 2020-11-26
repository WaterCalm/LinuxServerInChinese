# linuxserver/jellyfin

> [!TIP]
>
> 前半部分是翻译官方的文档，最后一部分是我的简单试用（个别软件会深度试用），如果对Docker已经有一定的了解了，可以直接跳转到最后面 [翻译之外](#翻译之外) 这部分来查看。

Jellyfin → https://jellyfin.github.io/

GitHub → https://github.com/linuxserver/docker-jellyfin

Docker Hub → https://hub.docker.com/r/linuxserver/jellyfin

[Jellyfin](https://jellyfin.github.io/) 是一个免费媒体系统，可让您控制媒体的管理和流式传输。它是Emby和Plex的替代产品，可以通过多个应用程序将专用服务器中的媒体提供给最终用户设备。 

Jellyfin是Emby 3.5.2发行版的衍生版本，并移植到.NET Core框架以实现全面的跨平台支持。没有附加条件，没有高级许可或功能，也没有隐藏的功能：只有一个团队想要构建更好的东西，并共同努力实现目标。

------

## 支持的系统架构

得益于docker的跨平台属性，我们的镜像也支持多架构（如，x86-64、arm64、armhf）。

直接拉取 `ghcr.io/linuxserver/jellyfin` 应该就可以自动获取适合你系统架构的版本，当然你也可以通过 tag 获取特定的版本。

| 架构   | Tag            |
| ------ | -------------- |
| x86-64 | amd64-latest   |
| arm64  | arm64v8-latest |
| armhf  | arm32v7-latest |

## 版本标签

| Tag     | 描述                     |
| ------- | ------------------------ |
| latest  | 稳定发行版（基于Focal）  |
| bionic  | 稳定发行版（基于Bionic） |
| nightly | 测试版                   |



------

## 拉取镜像

```shell
docker pull ghcr.io/linuxserver/jellyfin
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
  jellyfin:
    image: ghcr.io/linuxserver/jellyfin
    container_name: jellyfin
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
      - UMASK_SET=<022> #optional
    volumes:
      - /path/to/library:/config
      - /path/to/tvseries:/data/tvshows
      - /path/to/movies:/data/movies
      - /opt/vc/lib:/opt/vc/lib #optional
    ports:
      - 8096:8096
      - 8920:8920 #optional
      - 7359:7359/udp #optional
      - 1900:1900/udp #optional
    devices:
      - /dev/dri:/dev/dri #optional
      - /dev/vcsm:/dev/vcsm #optional
      - /dev/vchiq:/dev/vchiq #optional
      - /dev/video10:/dev/video10 #optional
      - /dev/video11:/dev/video11 #optional
      - /dev/video12:/dev/video12 #optional
    restart: unless-stopped
```

### docker cli

```shell
docker run -d \
  --name=jellyfin \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Europe/London \
  -e UMASK_SET=<022> `#optional` \
  -p 8096:8096 \
  -p 8920:8920 `#optional` \
  -p 7359:7359/udp `#optional` \
  -p 1900:1900/udp `#optional` \
  -v /path/to/library:/config \
  -v /path/to/tvseries:/data/tvshows \
  -v /path/to/movies:/data/movies \
  -v /opt/vc/lib:/opt/vc/lib `#optional` \
  --device /dev/dri:/dev/dri `#optional` \
  --device /dev/vcsm:/dev/vcsm `#optional` \
  --device /dev/vchiq:/dev/vchiq `#optional` \
  --device /dev/video10:/dev/video10 `#optional` \
  --device /dev/video11:/dev/video11 `#optional` \
  --device /dev/video12:/dev/video12 `#optional` \
  --restart unless-stopped \
  ghcr.io/linuxserver/jellyfin
```

## 参数

Docker镜像在使用的时候需要配置一些参数，这些参数使用 `:` 分隔，分别表示 `宿主机:容器内`。例如 `-p 8080:80` 指的是将容器内的`80`端口映射到宿主机上的`8080`端口，即通过宿主机网络访问的话，访问`8080`端口即是访问容器内的`80`端口。

### 端口（`-p`）

| port       | 说明                                              |
| ---------- | ------------------------------------------------- |
| `8096`     | WebUI 访问端口                                    |
| `8920`     | （可选）WebUI 的 HTTPS 访问端口，需要自行准备证书 |
| `7359/udp` | （可选）允许本地网络的客户端发现 Jellyfin         |
| `1900/udp` | （可选）DLNA服务                                  |

### 环境变量（`-e`）

| env                | 说明                                       |
| ------------------ | ------------------------------------------ |
| `PUID=1000`        | 用户的 UID，详见下面的说明                 |
| `PGID=1000`        | 用户的 GID，详见下面的说明                 |
| `TZ=Europe/London` | 设置时区，在国内的话可以使用 Asia/Shanghai |
| `UMASK_SET=<022>`  | Umask设置，默认是022                       |

### 卷映射（`-v`）

| volume          | 说明                                                         |
| --------------- | ------------------------------------------------------------ |
| `/config`       | 配置文件所在路径（这个目录会特别大，对于大的集合来说可能会高达50GB+） |
| `/data/tvshows` | 媒体文件夹，可以根据需求添加如 `/data/movies` 、`/data/tv` 等 |
| `/data/movies`  | 媒体文件夹，可以根据需求添加如 `/data/movies` 、`/data/tv` 等 |
| `/opt/vc/lib`   | （可选）树莓派 OpenMAX 库的路径                              |

### 设备映射（`--device`）

| 参数           | 说明                                                         |
| -------------- | ------------------------------------------------------------ |
| `/dev/dri`     | 如果你希望使用 Intel GPU 来开启硬件加速视频编码（vaapi）则设置该项。 |
| `/dev/vcsm`    | 如果你希望使用树莓派 MMAL 视频编码则设置该项（在GUI中开启 OpenMax H264 编码） |
| `/dev/vchiq`   | 如果你希望使用树莓派 OpenMax 视频编码则设置该项              |
| `/dev/video10` | 如果你希望使用树莓派 V4L2 视频编码则设置该项                 |
| `/dev/video11` | 如果你希望使用树莓派 V4L2 视频编码则设置该项                 |
| `/dev/video12` | 如果你希望使用树莓派 V4L2 视频编码则设置该项                 |

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

## 可选参数

设置端口可以使设备自动发现服务：https://jellyfin.org/docs/general/networking/index.html

服务发现（1900 / udp）：如果配置此选项客户端的自动发现会中断，因此您目前无法在设置中更改此设置。 DLNA也使用此端口，并且必须位于本地子网中。

客户端发现（7359 / udp）：允许客户端发现本地网络上的Jellyfin。使用“Who is Jellyfin Server?”向该端口广播消息。将获得一个包含服务器地址、ID、名称的JSON响应。

```shell
  -p 7359:7359/udp \
  -p 1900:1900/udp \
```



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

访问地址：`http://ip:8069`

更多信息请阅读官方文档：https://jellyfin.org/docs/general/quick-start.html

------

## 硬件加速

### Intel

使用 Intel Quicksync 硬件加速的用户需要在运行或创建容器时通过传递以下命令，将其 `/dev/dri` 视频设备安装在容器中：

```
--device=/dev/dri:/dev/dri
```

我们将自动确保容器内的abc用户具有访问此设备的适当权限。

### Nvidia

使用 Nvidia 硬件加速用户将需要在其主机上安装Nvidia提供的容器运行时，可以在此处找到说明：

https://github.com/NVIDIA/nvidia-docker

我们会自动添加必要的环境变量，该变量将利用主机上GPU上的所有可用功能。在主机上安装nvidia-docker之后，您将需要使用nvidia容器运行时`--runtime = nvidia`重新/创建docker容器，并添加环境变量`-e NVIDIA_VISIBLE_DEVICES = all`（也可以设置为特定GPU的UUID，这可以通过运行`nvidia-smi --query-gpu = gpu_name，gpu_uuid --format = csv`来发现。 NVIDIA将主机中的GPU和驱动程序自动安装到jellyfin docker容器中。

### OpenMAX (Raspberry Pi)

使用树莓派 MMAL / OpenMAX 硬件加速用户将需要在运行或创建容器时通过传递以下选项，将其 `/dev /vcsm` 和 `/dev/vchiq` 视频设备安装在容器及其系统OpenMax库中：

```
--device=/dev/vcsm:/dev/vcsm
--device=/dev/vchiq:/dev/vchiq
-v /opt/vc/lib:/opt/vc/lib
```

### V4L2 (Raspberry Pi)

使用树莓派 V4L2 硬件加速用户将需要在运行或创建容器时通过传递以下选项来将其 `/dev/video1X` 设备安装在容器内：

```
--device=/dev/video10:/dev/video10
--device=/dev/video11:/dev/video11
--device=/dev/video12:/dev/video12
```



------

## 支持

- 进入容器：
  - `docker exec -it jellyfin /bin/bash`
- 查看容器日志：
  - `docker logs -f jellyfin`
- 查看容器版本号：
  - `docker inspect -f '{% raw %}{{% endraw %}{ index .Config.Labels "build_version" }}' jellyfin`
- 查看镜像版本号：
  - `docker inspect -f '{% raw %}{{% endraw %}{ index .Config.Labels "build_version" }}' ghcr.io/linuxserver/jellyfin`

------

## 翻译之外

暂未试用