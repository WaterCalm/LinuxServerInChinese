# linuxserver/domoticz

> [!TIP]
>
> 前半部分是翻译官方的文档，最后一部分是我的简单试用（个别软件会深度试用），如果对Docker已经有一定的了解了，可以直接跳转到最后面 [翻译之外](#翻译之外) 这部分来查看。

Domoticz → https://www.domoticz.com/

GitHub → https://github.com/linuxserver/docker-domoticz

Docker Hub → https://hub.docker.com/r/linuxserver/domoticz

[Domoticz](https://www.domoticz.com/) 是一个家庭自动化系统，可让您监视和配置各种设备，例如：灯、开关、各种传感器/仪表（如温度、雨水、风、紫外线、燃气等）。也可以将通知/警报可以发送到任何移动设备。

------

## 支持的系统架构

得益于docker的跨平台属性，我们的镜像也支持多架构（如，x86-64、arm64、armhf）。

直接拉取 `ghcr.io/linuxserver/domoticz` 应该就可以自动获取适合你系统架构的版本，当然你也可以通过 tag 获取特定的版本。

| 架构   | Tag            |
| ------ | -------------- |
| x86-64 | amd64-latest   |
| arm64  | arm64v8-latest |
| armhf  | arm32v7-latest |

### 版本标签

| Tag           | 说明                                                        |
| ------------- | ----------------------------------------------------------- |
| latest        | 目前GitHub上的最新版本 https://github.com/domoticz/domoticz |
| stable        | 最新的稳定版本                                              |
| stable-4.9700 | 旧版本，不再更新                                            |
| stable-3.815  | 旧版本，不再更新                                            |
| stable-3.5877 | 旧版本，不再更新                                            |




------

## 拉取镜像

```shell
docker pull ghcr.io/linuxserver/domoticz
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
  domoticz:
    image: linuxserver/domoticz
    container_name: domoticz
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
      - WEBROOT=domoticz #optional
    volumes:
      - <path to data>:/config
    ports:
      - 8080:8080
      - 6144:6144
      - 1443:1443
    devices:
      - <path to device>:<path to device>
    restart: unless-stopped
```

### docker cli

```shell
docker create \
  --name=domoticz \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Europe/London \
  -e WEBROOT=domoticz `#optional` \
  -p 8080:8080 \
  -p 6144:6144 \
  -p 1443:1443 \
  -v <path to data>:/config \
  --device <path to device>:<path to device> \
  --restart unless-stopped \
  linuxserver/domoticz
```

### 直通USB设备

为了更好地使用 Domoticz，你可能会需要直通USB设备到Domoticz容器内。为了能定位USB设备，你需要将USB设备连接到设备上，并用 `dmesg` 命令查看设备节点。使用 `dmesg | tail` 后，你可能会看到如下的输出：

```shell
usb 1-1.2: new full-speed USB device number 7 using ehci-pci
ftdi_sio 1-1.2:1.0: FTDI USB Serial Device converter detected
usb 1-1.2: Detected FT232RL
usb 1-1.2: FTDI USB Serial Device converter now attached to ttyUSB0
```

如上所述，USB的设备节点是 `ttyUSB0` ，通常情况下都是在 `/dev/` 目录下。所以要把USB设备直通到容器内的话，可以在创建容器的时候使用 `--device /dev/ttyUSB0:/dev/ttyUSB0`

## 参数

Docker镜像在使用的时候需要配置一些参数，这些参数使用 `:` 分隔，分别表示 `宿主机:容器内`。例如 `-p 8080:80` 指的是将容器内的`80`端口映射到宿主机上的`8080`端口，即通过宿主机网络访问的话，访问`8080`端口即是访问容器内的`80`端口。

### 端口（`-p`）

| port   | 说明               |
| ------ | ------------------ |
| `8080` | Web界面            |
| `6144` | Domoticz的通讯端口 |
| `1443` | Domoticz的通讯端口 |

### 环境变量（`-e`）

| env                | 说明                                                         |
| ------------------ | ------------------------------------------------------------ |
| `PUID=1000`        | 用户的 UID，详见下面的说明                                   |
| `PGID=1000`        | 用户的 GID，详见下面的说明                                   |
| `TZ=Europe/London` | 设置时区，在国内的话可以使用 Asia/Shanghai                   |
| `WEBROOT=domoticz` | 设置子文件夹反向代理时候的网站路径。如果不使用反向代理，则不需要。 |

### 卷映射（`-v`）

| volume    | 说明             |
| --------- | ---------------- |
| `/config` | 配置文件所在路径 |

### 设备映射（`--device`）

| 参数     | 说明                    |
| -------- | ----------------------- |
| 设备路径 | 直通USB设备到容器内使用 |



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

访问 `http://ip:8080` 来配置Domoticz，在 `Setup > Hardware` 中添加硬件。更多信息请访问 https://www.domoticz.com/

------

## 支持

- 进入容器：
  - `docker exec -it domoticz /bin/bash`
- 查看容器日志：
  - `docker logs -f domoticz`
- 查看容器版本号：
  - `docker inspect -f '{% raw %}{{% endraw %}{ index .Config.Labels "build_version" }}' domoticz`
- 查看镜像版本号：
  - `docker inspect -f '{% raw %}{{% endraw %}{ index .Config.Labels "build_version" }}' linuxserver/domoticz`

------

## 翻译之外

首次启动需要等待的时间比较长，需耐心等。（可能根据配置等待的时间不同，我大概等了4-5分钟）

整体界面可以更改为中文

当然，要把设备接入到Domoticz中必须得是智能设备，比如我添加了一个yeelight的吸顶灯，然后就可以在界面中控制灯的开关和亮度。

在添加设备的时候看还支持小米的网关，要想接入自然还得需要获取到网关的IP和token，这个在这里就不介绍了。

至于智能家居系统，除了Domoticz外，还有比较知名的HomeAssistant。

![image-20201028114624701](https://pic.watercalmx.com/pic/image-20201028114624701.png)

![image-20201028115422282](https://pic.watercalmx.com/pic/image-20201028115422282.png)