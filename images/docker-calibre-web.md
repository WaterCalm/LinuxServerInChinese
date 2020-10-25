# linuxserver/calibre-web

Calibre-web → https://github.com/janeczku/calibre-web

GitHub → https://github.com/linuxserver/docker-calibre-web

Docker Hub → https://hub.docker.com/r/linuxserver/calibre-web

[Calibre-web](https://github.com/janeczku/calibre-web) 是一个基于WEB的应用程序，可以通过简洁的界面浏览、阅读、下载Calibre数据库的电子书（需要已存在的Calibre数据库）。也可以通过应用来集成Google Drive并编辑元数据和Calibre数据库。

该软件是 [calibreserver](https://github.com/mutschler/calibreserver) 的分支，使用 GPL v3 许可协议。

------

## 支持的系统架构

得益于docker的跨平台属性，我们的镜像也支持多架构（如，x86-64、arm64、armhf）。

直接拉取 `linuxserver/calibre-web` 应该就可以自动获取适合你系统架构的版本，当然你也可以通过 tag 获取特定的版本。

| 架构   | Tag            |
| ------ | -------------- |
| x86-64 | amd64-latest   |
| arm64  | arm64v8-latest |
| armhf  | arm32v7-latest |

------

## 拉取镜像

```shell
docker pull linuxserver/calibre-web
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
  calibre-web:
    image: linuxserver/calibre-web
    container_name: calibre-web
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
      - DOCKER_MODS=linuxserver/calibre-web:calibre
    volumes:
      - <path to data>:/config
      - <path to calibre library>:/books
    ports:
      - 8083:8083
    restart: unless-stopped
```

### docker cli

```shell
docker run -d \
  --name=calibre-web \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Europe/London \
  -e DOCKER_MODS=linuxserver/calibre-web:calibre \
  -p 8083:8083 \
  -v <path to data>:/config \
  -v <path to calibre library>:/books \
  --restart unless-stopped \
  linuxserver/calibre-web
```

## 参数

Docker镜像在使用的时候需要配置一些参数，这些参数使用 `:` 分隔，分别表示 `宿主机:容器内`。例如 `-p 8080:80` 指的是将容器内的`80`端口映射到宿主机上的`8080`端口，即通过宿主机网络访问的话，访问`8080`端口即是访问容器内的`80`端口。

### 端口（`-p`）

| port   | 说明        |
| ------ | ----------- |
| `8083` | Web管理界面 |

### 环境变量（`-e`）

| env                                           | 说明                                       |
| --------------------------------------------- | ------------------------------------------ |
| `PUID=1000`                                   | 用户的 UID，详见下面的说明                 |
| `PGID=1000`                                   | 用户的 GID，详见下面的说明                 |
| `TZ=Europe/London`                            | 设置时区，在国内的话可以使用 Asia/Shanghai |
| `DOCKER_MODS=linuxserver/calibre-web:calibre` | 可选参数，仅x86-64版本添加了电子书转换功能 |

### 卷映射（`-v`）

| volume    | 说明                |
| --------- | ------------------- |
| `/config` | 配置文件所在路径    |
| `/books`  | Calibre数据库的位置 |

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

可以通过 `http://ip:8083` 访问

在初始化安装界面，输入 `/books` 作为你的Calibre库位置

**默认管理员账号密码**：`admin/admin123`

镜像中默认包含 Unrar ，需要在管理界面中（`Basic Configuration:External Binaries`）填写路径：`/usr/bin/unrar`

**仅限于x86-64：**我们已经实现可以使用Calibre来进行电子书转换，如果需要改功能可以很轻松地配置（不需要的话也可以不使用以节省空间）。当Calibre发布新版本的时候，镜像将自动重新构建。如果要启用该功能，附上相关的环境变量即可。然后在管理页面中（`Basic Configuration:External Binaries`）填写下面的路径：`/usr/bin/ebook-convert`

镜像中包含了 kepubify 用来将epub格式转换为kepub。如果要使用，在管理页面中（`Basic Configuration:External Binaries`）填写下面的路径：`/usr/bin/kepubify`

如果要使用反向代理，我们提供了以下的参考：

```
        location /calibre-web {
                proxy_pass              http://<your-ip>:8083;
                proxy_set_header        Host            $http_host;
                proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header        X-Scheme        $scheme;
                proxy_set_header        X-Script-Name   /calibre-web;
        }
```

------

## 支持

- 进入容器：
  - `docker exec -it calibre-web /bin/bash`
- 查看容器日志：
  - `docker logs -f calibre-web`
- 查看容器版本号：
  - `docker inspect -f '{% raw %}{{% endraw %}{ index .Config.Labels "build_version" }}' calibre-web`
- 查看镜像版本号：
  - `docker inspect -f '{% raw %}{{% endraw %}{ index .Config.Labels "build_version" }}' linuxserver/calibre-web`

------

## 翻译之外

> [!NOTE]
>
> 这个镜像的应用需要已经有一个已经存在的Calibre数据库！如果没有的话可以使用 `linuxserver/calibre` 镜像创建一个，或者下载 Calibre 的客户端创建一个。最终只要将数据库的目录映射给容器里的`/books`目录即可。

总的来说Calibre-web是一个管理图书的工具，它并不能在线阅读，但可以进行下载、分类、转换等操作。可以很方便地将你的收集的电子书与他人分享。建议配合 Calibre 客户端使用，用 Calibre 客户端更新数据库后，Calibre-web 也可以实时更新。

与之相似的镜像还有 [linuxserver/cops](images/docker-cops.md) 

![image-20201022113752658](https://pic.watercalmx.com/pic/image-20201022113752658.png)

![image-20201022115632149](https://pic.watercalmx.com/pic/image-20201022115632149.png)

![image-20201022120035837](https://pic.watercalmx.com/pic/image-20201022120035837.png)