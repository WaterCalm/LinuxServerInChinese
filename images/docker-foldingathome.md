# linuxserver/foldingathome

> [!TIP]
>
> 前半部分是翻译官方的文档，最后一部分是我的简单试用（个别软件会深度试用），如果对Docker已经有一定的了解了，可以直接跳转到最后面 [翻译之外](#翻译之外) 这部分来查看。

Folding@home → https://foldingathome.org/

GitHub → https://github.com/linuxserver/docker-foldingathome

Docker Hub → https://hub.docker.com/r/linuxserver/foldingathome

[Folding@home](https://foldingathome.org/) 是一个分布式计算项目，用于模拟蛋白质动力学，包括蛋白质折叠的过程以及与多种疾病有关的蛋白质的运动。它汇集了公民科学家，他们自愿在他们的个人计算机上运行蛋白质动力学的模拟。这些数据的洞察力正在帮助科学家更好地了解生物学，并为开发治疗方法提供了新的机会。

------

## 支持的系统架构

得益于docker的跨平台属性，我们的镜像也支持多架构（如，x86-64、arm64、armhf）。

直接拉取 `ghcr.io/linuxserver/foldingathome` 应该就可以自动获取适合你系统架构的版本，当然你也可以通过 tag 获取特定的版本。

| 架构   | Tag    |
| ------ | ------ |
| x86-64 | latest |

------

## 拉取镜像

```shell
docker pull ghcr.io/linuxserver/foldingathome
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
  foldingathome:
    image: ghcr.io/linuxserver/foldingathome
    container_name: foldingathome
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
    volumes:
      - /path/to/data:/config
    ports:
      - 7396:7396
      - 36330:36330 #optional
    restart: unless-stopped
```

### docker cli

```shell
docker run -d \
  --name=foldingathome \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Europe/London \
  -p 7396:7396 \
  -p 36330:36330 `#optional` \
  -v /path/to/data:/config \
  --restart unless-stopped \
  ghcr.io/linuxserver/foldingathome
```

## 参数

Docker镜像在使用的时候需要配置一些参数，这些参数使用 `:` 分隔，分别表示 `宿主机:容器内`。例如 `-p 8080:80` 指的是将容器内的`80`端口映射到宿主机上的`8080`端口，即通过宿主机网络访问的话，访问`8080`端口即是访问容器内的`80`端口。

### 端口（`-p`）

| port    | 说明                                                 |
| ------- | ---------------------------------------------------- |
| `7369`  | Web界面                                              |
| `36330` | （可选）通过 FAHControl 进行远程连接的端口（无密码） |

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

### 


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

镜像将启动 Folding@home 客户端，访问 `http://ip:7396` 进入界面

内建的web服务器只能进行非常基础的控制（如，GPU只在设置为 Medium 以上时才可用）。如果要进行更多控制，需要在其他设备上用 FAHControl 通过 `36330` 端口连接（无密码）

web界面有以下几个问题：

- 如果尝试通过IP访问时收到“ ERR_EMPTY_RESPONSE”错误，则最有可能是由于Cookie /缓存冲突。尝试在放大窗口中打开。
- 如果您不断刷新窗口但没有显示任何信息，请尝试通过shft-F5或ctrl-F5强制刷新。

### GPU硬件加速

#### Nvidia

Nvidia的硬件加速用户将需要在其主机上安装Nvidia提供的容器运行时，有关说明，请参见：`https://github.com/NVIDIA/nvidia-docker`。

我们自动添加必要的环境变量，该变量将利用所有主机上GPU上可用的功能。在主机上安装nvidia-docker之后，您将需要使用nvidia容器运行时`--runtime = nvidia`重新/创建docker容器，并添加环境变量`-e NVIDIA_VISIBLE_DEVICES = all`（也可以设置为特定GPU的UUID，这可以通过运行`nvidia-smi --query-gpu = gpu_name,gpu_uuid --format = csv`来获取。 NVIDIA会自动将主机中的GPU和驱动程序安装到foldingathome docker容器中。

------

## 支持

- 进入容器：
  - `docker exec -it foldingathome /bin/bash`
- 查看容器日志：
  - `docker logs -f foldingathome`
- 查看容器版本号：
  - `docker inspect -f '{% raw %}{{% endraw %}{ index .Config.Labels "build_version" }}' foldingathome`
- 查看镜像版本号：
  - `docker inspect -f '{% raw %}{{% endraw %}{ index .Config.Labels "build_version" }}' ghcr.io/linuxserver/foldingathome`

------

## 翻译之外

又是一个科研项目，为科研贡献出自己的一份力吧！