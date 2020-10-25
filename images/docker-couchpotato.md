# linuxserver/couchpotato

Couchpotato → https://couchpota.to/

GitHub → https://github.com/linuxserver/docker-couchpotato

Docker Hub → https://hub.docker.com/r/linuxserver/couchpotato

[Couchpotato](https://couchpota.to/) 是一个 NZB 和 Torrent 自动下载器。你可以更新 `movies I want` 列表，它将每隔一段时间搜索一次 NZB/Torrent 服务器。如果找到对应的电影，它将发送到 SABnzbd 上或下载 torrent。

------

## 支持的系统架构

得益于docker的跨平台属性，我们的镜像也支持多架构（如，x86-64、arm64、armhf）。

直接拉取 `linuxserver/couchpotato` 应该就可以自动获取适合你系统架构的版本，当然你也可以通过 tag 获取特定的版本。

| 架构   | Tag            |
| ------ | -------------- |
| x86-64 | amd64-latest   |
| arm64  | arm64v8-latest |
| armhf  | arm32v7-latest |


------

## 拉取镜像

```shell
docker pull linuxserver/couchpotato
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
  couchpotato:
    image: linuxserver/couchpotato
    container_name: couchpotato
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
      - UMASK_SET=022
    volumes:
      - /path/to/appdata/config:/config
      - /path/to/downloads:/downloads
      - /path/to/movies:/movies
    ports:
      - 5050:5050
    restart: unless-stopped
```

### docker cli

```shell
docker run -d \
  --name=couchpotato \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Europe/London \
  -e UMASK_SET=022 \
  -p 5050:5050 \
  -v /path/to/appdata/config:/config \
  -v /path/to/downloads:/downloads \
  -v /path/to/movies:/movies \
  --restart unless-stopped \
  linuxserver/couchpotato
```

## 参数

Docker镜像在使用的时候需要配置一些参数，这些参数使用 `:` 分隔，分别表示 `宿主机:容器内`。例如 `-p 8080:80` 指的是将容器内的`80`端口映射到宿主机上的`8080`端口，即通过宿主机网络访问的话，访问`8080`端口即是访问容器内的`80`端口。

### 端口（`-p`）

| port   | 说明    |
| ------ | ------- |
| `5050` | WEB界面 |

### 环境变量（`-e`）

| env                | 说明                                       |
| ------------------ | ------------------------------------------ |
| `PUID=1000`        | 用户的 UID，详见下面的说明                 |
| `PGID=1000`        | 用户的 GID，详见下面的说明                 |
| `TZ=Europe/London` | 设置时区，在国内的话可以使用 Asia/Shanghai |
| `UMASK_SET=022`    | 默认的是022，设置文件的umask               |

### 卷映射（`-v`）

| volume       | 说明             |
| ------------ | ---------------- |
| `/config`    | 配置文件所在路径 |
| `/downloads` | 下载文件夹       |
| `/movies`    | 电影分享文件夹   |

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

访问WEB界面：`http://ip:5050` 

更多信息，请查看官网 → https://couchpota.to/

------

## 支持

- 进入容器：
  - `docker exec -it couchpotato /bin/bash`
- 查看容器日志：
  - `docker logs -f couchpotato`
- 查看容器版本号：
  - `docker inspect -f '{% raw %}{{% endraw %}{ index .Config.Labels "build_version" }}' couchpotato`
- 查看镜像版本号：
  - `docker inspect -f '{% raw %}{{% endraw %}{ index .Config.Labels "build_version" }}' linuxserver/couchpotato`

------

## 翻译之外

首次启动会进行一些设置，主要是设置用户名、密码并选择下载器和搜索的网站，也提示有插件可以使用，插件的功能就是浏览类似于 IMDB 这种网站时，对某个电影感兴趣，可以直接加入清单中。

设置完成后，要重启下容器才生效。

在加入清单之前，会选择清晰度之类的东西，加入清单后就开始搜索。

可能对中文资源支持不是很好，因为目前我还没安装支持的下载器，等后续安装了后再继续试用一下。

![image-20201025161023998](https://pic.watercalmx.com/pic/image-20201025161023998.png)

![image-20201025161137280](https://pic.watercalmx.com/pic/image-20201025161137280.png)

![image-20201025161201560](https://pic.watercalmx.com/pic/image-20201025161201560.png)

![image-20201025161227195](https://pic.watercalmx.com/pic/image-20201025161227195.png)

![image-20201025161627105](https://pic.watercalmx.com/pic/image-20201025161627105.png)

![image-20201025161826467](https://pic.watercalmx.com/pic/image-20201025161826467.png)

![image-20201025162005657](https://pic.watercalmx.com/pic/image-20201025162005657.png)