# linuxserver/digikam

> [!TIP]
>
> 前半部分是翻译官方的文档，最后一部分是我的简单试用（个别软件会深度试用），如果对Docker已经有一定的了解了，可以直接跳转到最后面 [翻译之外](#翻译之外) 这部分来查看。

digiKam → https://www.digikam.org/

GitHub → https://github.com/linuxserver/docker-digikam

Docker Hub → https://hub.docker.com/r/linuxserver/digikam

[digiKam](https://www.digikam.org/) 是一个开源的专业照片管理工具。

------

## 支持的系统架构

得益于docker的跨平台属性，我们的镜像也支持多架构（如，x86-64、arm64、armhf）。

直接拉取 `linuxserver/digikam` 应该就可以自动获取适合你系统架构的版本，当然你也可以通过 tag 获取特定的版本。

| 架构   | Tag    |
| ------ | ------ |
| x86-64 | latest |


------

## 拉取镜像

```shell
docker pull linuxserver/digikam
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
  digikam:
    image: linuxserver/digikam
    container_name: digikam
    network_mode: host
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/New_York
    volumes:
      - /path/to/config:/config
    ports:
    ports:
      - 3000:3000 #optional
    restart: unless-stopped
```

### docker cli

```shell
docker create \
  --name=digikam \
  --net=host \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=America/New_York \
  -p 3000:3000 `#optional` \
  -v /path/to/config:/config \
  --restart unless-stopped \
  linuxserver/digikam
```

## 参数

Docker镜像在使用的时候需要配置一些参数，这些参数使用 `:` 分隔，分别表示 `宿主机:容器内`。例如 `-p 8080:80` 指的是将容器内的`80`端口映射到宿主机上的`8080`端口，即通过宿主机网络访问的话，访问`8080`端口即是访问容器内的`80`端口。

### 端口（`-p`）

| port   | 说明        |
| ------ | ----------- |
| `3000` | digiKam界面 |

### 网络设置（`--net`）

| network      | 说明                 |
| ------------ | -------------------- |
| `--net=host` | 需要共享宿主机的网络 |

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

访问 `http://ip:3000` ，默认的用户名/密码是 `abc/abc` 

如果要更改密码或手动登陆，通过 `http://ip:3000/?login=true` 访问

------

## 支持

- 进入容器：
  - `docker exec -it digikam /bin/bash`
- 查看容器日志：
  - `docker logs -f digikam`
- 查看容器版本号：
  - `docker inspect -f '{% raw %}{{% endraw %}{ index .Config.Labels "build_version" }}' digikam`
- 查看镜像版本号：
  - `docker inspect -f '{% raw %}{{% endraw %}{ index .Config.Labels "build_version" }}' linuxserver/digikam`

------

## 翻译之外

digiKam是一个专业的图片管理软件，这个镜像就是使其部署在了容器里。

第一次打开进行一些简单的设置，因为没连接外部的数据库，选择 SQLite 即可。

这个软件应该是支持中文的，但似乎容器里没有中文字体，所以在切换语言的时候都显示的小方块。

还好手头有几张Raw格式的照片，放进去也都可以识别，可以进行打标签、分类、面部识别、找类似等处理。

![image-20201027114950535](https://pic.watercalmx.com/pic/image-20201027114950535.png)

![image-20201027115022253](https://pic.watercalmx.com/pic/image-20201027115022253.png)

![image-20201027115139800](https://pic.watercalmx.com/pic/image-20201027115139800.png)

![image-20201027115657042](https://pic.watercalmx.com/pic/image-20201027115657042.png)

![image-20201027115825550](https://pic.watercalmx.com/pic/image-20201027115825550.png)