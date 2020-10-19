# linuxserver/airsonic

Airsonic → https://github.com/airsonic/airsonic

GitHub → https://github.com/linuxserver/docker-airsonic

Docker Hub → https://hub.docker.com/r/linuxserver/airsonic

[Airsonic](https://github.com/airsonic/airsonic)是一个免费的基于web的媒体流媒体程序，可以让你在任何地方聆听你的音乐。你可以随时随地地听音乐并分享给你的朋友。也可以同时流式传输给多个音乐播放器，比如一个在厨房，一个在客厅。

------

## 支持的系统架构

得益于docker的跨平台属性，我们的镜像也支持多架构（如，x86-64、arm64、armhf）。

直接拉取 `linuxserver/airsonic` 应该就可以自动获取适合你系统架构的版本，当然你也可以通过 tag 获取特定的版本。

| 架构   | Tag            |
| ------ | -------------- |
| x86-64 | amd64-latest   |
| arm64  | arm64v8-latest |
| armhf  | arm32v7-latest |

### 拉取镜像

```shell
docker pull linuxserver/airsonic
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
  airsonic:
    image: linuxserver/airsonic
    container_name: airsonic
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
      - CONTEXT_PATH=<URL_BASE> #optional
      - JAVA_OPTS=<options> #optional
    volumes:
      - </path/to/config>:/config
      - </path/to/music>:/music
      - </path/to/playlists>:/playlists
      - </path/to/podcasts>:/podcasts
      - </path/to/other media>:/media #optional
    ports:
      - 4040:4040
    devices:
      - /dev/snd:/dev/snd #optional
    restart: unless-stopped
```

### docker cli

```shell
docker run -d \
  --name=airsonic \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Europe/London \
  -e CONTEXT_PATH=<URL_BASE> `#optional` \
  -e JAVA_OPTS=<options> `#optional` \
  -p 4040:4040 \
  -v </path/to/config>:/config \
  -v </path/to/music>:/music \
  -v </path/to/playlists>:/playlists \
  -v </path/to/podcasts>:/podcasts \
  -v </path/to/other media>:/media `#optional` \
  --device /dev/snd:/dev/snd `#optional` \
  --restart unless-stopped \
  linuxserver/airsonic
```

## 参数

Docker镜像在使用的时候需要配置一些参数，这些参数使用 `:` 分隔，分别表示 `宿主机:容器内`。例如 `-p 8080:80` 指的是将容器内的`80`端口映射到宿主机上的`8080`端口，即通过宿主机网络访问的话，访问`8080`端口即是访问容器内的`80`端口。

### 端口（`-p`）

| port   | 说明    |
| ------ | ------- |
| `4040` | Web界面 |

### 环境变量（`-e`）

| env                       | 说明                                       |
| ------------------------- | ------------------------------------------ |
| `PUID=1000`               | 用户的 UID，详见下面的说明                 |
| `PGID=1000`               | 用户的 GID，详见下面的说明                 |
| `TZ=Europe/London`        | 设置时区，在国内的话可以使用 Asia/Shanghai |
| `CONTEXT_PATH=<URL_BASE>` | 用于在设置反向代理时候的重定向url          |
| `JAVA_OPTS=<options>`     | 传递其他的Java配置                         |

### 卷映射（`-v`）

| volume       | 说明               |
| ------------ | ------------------ |
| `/config`    | 配置文件的位置     |
| `/music`     | 音乐文件夹         |
| `/playlists` | 播放列表保存的位置 |
| `/podcasts`  | 播客的目录         |
| `/media`     | 其他媒体文件的位置 |

### 设备映射（`--device`）

| 设备     | 说明                                   |
| -------- | -------------------------------------- |
| /dev/snd | 映射 Airsonic 的 Java jukebox 播放器。 |



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

## 安装应用

Web管理界面： <ip地址>:4040

默认用户名和密码都是：admin

可以通过 `JAVA_OPTS` 传递额外的Java配置参数，如`-e JAVA_OPTS="-Xmx256m -Xms256m"`。

对于反向代理，可能需要传递 `JAVA_OPTS=-Dserver.use-forward-headers=true` 来让airsonic正常使用。

注意：如果你希望使用 [Airsonic 的 Java jukebox 播放器](https://airsonic.github.io/docs/jukebox/)，需要让 PGID 和硬件声音设备相匹配（如，/dev/snd）

------

## 支持

- 进入容器：
  - `docker exec -it airsonic /bin/bash`
- 查看容器日志：
  - `docker logs -f airsonic`
- 查看容器版本号：
  - `docker inspect -f '{{ index .Config.Labels "build_version" }}' airsonic`
- 查看镜像版本号：
  - `docker inspect -f '{{ index .Config.Labels "build_version" }}' linuxserver/airsonic`

------

## 翻译之外

简单试用了一下Airsonic，整体的UI风格较为复古，可以设置为中文面板，虽然并没有100%汉化，但已经不影响使用。

目前在线播放音乐的在线服务商已经很多了（如，QQ音乐、网易云音乐等），所以感觉这个软件对于普通只是听音乐的用户来说用处不是非常大。除非你喜欢收集音乐，但似乎自动下载歌词的功能对中文歌的支持也不是很友好，随便放了几首歌进去，都无法自动搜索歌词。

airsonic可以创建多个播放器和播放列表并且可以开启DLAN服务，感觉这些功能更适用于餐厅、茶室这种有针对不同区域播放不同音乐的需求。

因为airsonic也是一个流媒体服务器，如果你喜欢的音乐的版权分别属于不同的服务商……那么可以通过某些手段都下载到airsonic上并生成一个m3u格式的播放列表，放到自己的播放器里播放。

还有在线转码、播客、在线收音机等功能，但未进行深入体验，感兴趣的朋友可能自己尝试一下。

![image-20201019145822703](https://pic.watercalmx.com/pic/image-20201019145822703.png)

![image-20201019145913922](https://pic.watercalmx.com/pic/image-20201019145913922.png)

![image-20201019152528107](https://pic.watercalmx.com/pic/image-20201019152528107.png)