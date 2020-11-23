# linuxserver/boinc

> [!TIP]
>
> 前半部分是翻译官方的文档，最后一部分是我的简单试用（个别软件会深度试用），如果对Docker已经有一定的了解了，可以直接跳转到最后面 [翻译之外](#翻译之外) 这部分来查看。

BOINC → https://boinc.berkeley.edu/

GitHub → https://github.com/linuxserver/docker-boinc

Docker Hub → https://hub.docker.com/r/linuxserver/boinc

[BOINC](https://boinc.berkeley.edu/) 是用于大规模（成百上千台计算机）高吞吐量的计算平台。它可以让志愿者（）或网格计算（）贡献其计算资源。它支持虚拟化、parallel、基于GPU的应用。

------

## 支持的系统架构

得益于docker的跨平台属性，我们的镜像也支持多架构（如，x86-64、arm64、armhf）。

直接拉取 `ghcr.io/linuxserver/beets` 应该就可以自动获取适合你系统架构的版本，当然你也可以通过 tag 获取特定的版本。

| 架构   | Tag            |
| ------ | -------------- |
| x86-64 | amd64-latest   |
| arm64  | arm64v8-latest |
| armhf  | arm32v7-latest |

------

## 拉取镜像

```shell
docker pull ghcr.io/linuxserver/boinc
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
  boinc:
    image: linuxserver/boinc
    container_name: boinc
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
      - GUAC_USER=abc #optional
      - GUAC_PASS=900150983cd24fb0d6963f7d28e17f72 #optional
    volumes:
      - /path/to/data:/config
    ports:
      - 8080:8080
    devices:
      - /dev/dri:/dev/dri #optional
    restart: unless-stopped
```

### docker cli

```shell
docker run -d \
  --name=boinc \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Europe/London \
  -e GUAC_USER=abc `#optional` \
  -e GUAC_PASS=900150983cd24fb0d6963f7d28e17f72 `#optional` \
  -p 8080:8080 \
  -v /path/to/data:/config \
  --device /dev/dri:/dev/dri `#optional` \
  --restart unless-stopped \
  linuxserver/boinc
```

## 参数

Docker镜像在使用的时候需要配置一些参数，这些参数使用 `:` 分隔，分别表示 `宿主机:容器内`。例如 `-p 8080:80` 指的是将容器内的`80`端口映射到宿主机上的`8080`端口，即通过宿主机网络访问的话，访问`8080`端口即是访问容器内的`80`端口。

### 端口（`-p`）

| port   | 说明        |
| ------ | ----------- |
| `8080` | Boinc的界面 |

### 环境变量（`-e`）

| env                                        | 说明                                       |
| ------------------------------------------ | ------------------------------------------ |
| `PUID=1000`                                | 用户的 UID，详见下面的说明                 |
| `PGID=1000`                                | 用户的 GID，详见下面的说明                 |
| `TZ=Europe/London`                         | 设置时区，在国内的话可以使用 Asia/Shanghai |
| GUAC_USER=abc                              | BOINC桌面客户端的用户名                    |
| GUAC_PASS=900150983cd24fb0d6963f7d28e17f72 | BOINC桌面客户端的密码（md5加密）           |

### 卷映射（`-v`）

| volume    | 说明                |
| --------- | ------------------- |
| `/config` | BOINC的数据库及配置 |

### 设备映射（`--device`）

| 参数     | 说明                                       |
| -------- | ------------------------------------------ |
| /dev/dri | 如果你想使用你的Intel GPU(vaapi)则设置该项 |



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

该镜像将搭建BOINC客户端，并在浏览器通过Guacamole服务器来进行可视化管理。可视化管理界面地址：`http://ip:8080`

默认情况下，不会新建用户。可以通过在创建容器的时候的环境变量来新建用户。要注意的是， GUACPASS 只接受经过md5加密过后的密码（示例中的是 abc 经过加密后的字符串）。可以通过下面这条命令来任意一条命令来进行md5加密：

```shell
echo -n password | openssl md5
```

```c
printf '%s' password | md5sum
```

在访问Guacamole远程桌面时，可以使用 `ctrl + alt + shift` 来查看高级功能（远程复制/粘贴以及设置语言）。

推荐在菜单中点击 `Advanced View` 来启用高级功能，因为在 `Simple View` 下是没有 `Computing Preferences`的。

有时，弹出的窗口可能会在屏幕左上角的小框中，只要拖拽更改其大小即可。

------

## GPU硬件加速

### Intel

对于要使用 Intel Quicksync 硬件加速的用户来说，需要使用下面这条配置将显卡挂载到容器里：`--device=/dev/dri:/dev/dri` 。镜像将自动把相关的权限分配给用户abc。

### Nvidia

对于要使用 Nvidia 硬件加速的用户来说，需要安装Nvidia提供的相关驱动（详见：https://github.com/NVIDIA/nvidia-docker）。如果宿主机上有支持的硬件设备，镜像会自动添加必要的环境变量。安装完 nvidia-docker 后，必须要使用 `--runtime=nvidia` 以及添加一条 `-e NVIDIA_VISIBLE_DEVICES=all` （也可以指定某一个gpu UUID，可以通过 `nvidia-smi --query-gpu=gpu_name,gpu_uuid --format=csv` 来查询）环境变量来重新创建容器。NVIDIA将自动把GPU挂载到BOINC容器内。

------

## 支持

- 进入容器：
  - `docker exec -it boinc /bin/bash`
- 查看容器日志：
  - `docker logs -f boinc`
- 查看容器版本号：
  - `docker inspect -f '{% raw %}{{% endraw %}{ index .Config.Labels "build_version" }}' boinc`
- 查看镜像版本号：
  - `docker inspect -f '{% raw %}{{% endraw %}{ index .Config.Labels "build_version" }}' linuxserver/boinc`

------

## 翻译之外

BOINC是分布式计算平台，可以在其中选择喜欢的项目，并贡献出来自己闲置计算机的计算资源。

关于BOINC的具体内容可以自行百度，也有专门的贴吧来讨论，官网也有中文介绍。

https://boinc.berkeley.edu/download.php 这里也有专门的客户端。

如果家里有闲置的电脑，跑跑BOINC似乎也是不错的选择，为科研贡献一份力嘛！

![image-20201020133452580](https://pic.watercalmx.com/pic/image-20201020133452580.png)

![image-20201020133921621](https://pic.watercalmx.com/pic/image-20201020133921621.png)

![image-20201020134250892](https://pic.watercalmx.com/pic/image-20201020134250892.png)

![image-20201020134308819](https://pic.watercalmx.com/pic/image-20201020134308819.png)

![image-20201020134322702](https://pic.watercalmx.com/pic/image-20201020134322702.png)

![image-20201020134351262](https://pic.watercalmx.com/pic/image-20201020134351262.png)

![image-20201020134935816](https://pic.watercalmx.com/pic/image-20201020134935816.png)