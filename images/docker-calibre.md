# linuxserver/calibre

> [!TIP]
>
> 前半部分是翻译官方的文档，最后一部分是我的简单试用（个别软件会深度试用），如果对Docker已经有一定的了解了，可以直接跳转到最后面 [翻译之外](#翻译之外) 这部分来查看。

Calibre → https://calibre-ebook.com/

GitHub → https://github.com/linuxserver/docker-calibre

Docker Hub → https://hub.docker.com/r/linuxserver/calibre

[Calibre](https://calibre-ebook.com/) 是一款具有强大功能并且简单易用的电子书管理工具，相比于比普通的电子书软件来说，该软件具有很完善的功能，而且它是免费开源的，很适合各类用户使用。

------

## 支持的系统架构

得益于docker的跨平台属性，我们的镜像也支持多架构（如，x86-64、arm64、armhf）。

直接拉取 `ghcr.io/linuxserver/calibre` 应该就可以自动获取适合你系统架构的版本，当然你也可以通过 tag 获取特定的版本。

| 架构   | Tag    |
| ------ | ------ |
| x86-64 | latest |

------

## 拉取镜像

```shell
docker pull ghcr.io/linuxserver/calibre
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
  calibre:
    image: linuxserver/calibre
    container_name: calibre
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
      - GUAC_USER=abc #optional
      - GUAC_PASS=900150983cd24fb0d6963f7d28e17f72 #optional
      - UMASK_SET=022 #optional
      - CLI_ARGS= #optional
    volumes:
      - /path/to/data:/config
    ports:
      - 8080:8080
      - 8081:8081
    restart: unless-stopped
```

### docker cli

```shell
docker run -d \
  --name=calibre \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Europe/London \
  -e GUAC_USER=abc `#optional` \
  -e GUAC_PASS=900150983cd24fb0d6963f7d28e17f72 `#optional` \
  -e UMASK_SET=022 `#optional` \
  -e CLI_ARGS= `#optional` \
  -p 8080:8080 \
  -p 8081:8081 \
  -v /path/to/data:/config \
  --restart unless-stopped \
  linuxserver/calibre
```

## 参数

Docker镜像在使用的时候需要配置一些参数，这些参数使用 `:` 分隔，分别表示 `宿主机:容器内`。例如 `-p 8080:80` 指的是将容器内的`80`端口映射到宿主机上的`8080`端口，即通过宿主机网络访问的话，访问`8080`端口即是访问容器内的`80`端口。

### 端口（`-p`）

| port   | 说明                  |
| ------ | --------------------- |
| `8080` | Calibre界面           |
| `8081` | Calibre网页服务器界面 |

### 环境变量（`-e`）

| env                                          | 说明                                       |
| -------------------------------------------- | ------------------------------------------ |
| `PUID=1000`                                  | 用户的 UID，详见下面的说明                 |
| `PGID=1000`                                  | 用户的 GID，详见下面的说明                 |
| `TZ=Europe/London`                           | 设置时区，在国内的话可以使用 Asia/Shanghai |
| `GUAC_USER=abc`                              | 桌面界面的用户名                           |
| `GUAC_PASS=900150983cd24fb0d6963f7d28e17f72` | 桌面界面的密码（md5加密）                  |
| `UMASK_SET=022`                              | umask的设置，默认为022                     |
| `CLI_ARGS=`                                  | 可选，用其他cli参数启动calibre             |

### 卷映射（`-v`）

| volume    | 说明             |
| --------- | ---------------- |
| `/config` | 配置文件所在路径 |

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

该应用通过Cuacamole服务来实现桌面界面，通过 `http://ip:8080` 访问。

默认情况下，镜像未设置用户名和密码，可以通过docker的环境变量来设置。`GUACPASS` 需要为md5加密的格式（示例中是abc的md5加密值）。可以通过以下任意一条命令来计算md5加密值：

```
echo -n password | openssl md5
```

```
printf '%s' password | md5sum
```

`8081`端口是为Calibre内置Web服务器保留的，可以在桌面界面中的设置启用它，并且只能设置为8081端口，然后再映射到宿主机上的任意端口

可以通过快捷键 `Ctrl + Alt + Shift` 来呼出Guacamole的高级面板来使用复制粘贴功能。

------

## 支持

- 进入容器：
  - `docker exec -it calibre /bin/bash`
- 查看容器日志：
  - `docker logs -f calibre`
- 查看容器版本号：
  - `docker inspect -f '{% raw %}{{% endraw %}{ index .Config.Labels "build_version" }}' calibre`
- 查看镜像版本号：
  - `docker inspect -f '{% raw %}{{% endraw %}{ index .Config.Labels "build_version" }}' ghcr.io/linuxserver/calibre`

------

## 翻译之外

Calibre不用过多介绍，很有名的电子书管理工具，可以把电子书转换成任何格式适配对应的电子书软件或硬件。

可以配合 Calibre-web 使用，来将电子书分享给他人。

![image-20201022115004080](https://pic.watercalmx.com/pic/image-20201022115004080.png)

![image-20201022115325737](https://pic.watercalmx.com/pic/image-20201022115325737.png)