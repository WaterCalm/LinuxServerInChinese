# linuxserver/bookstack

> [!TIP]
>
> 前半部分是翻译官方的文档，最后一部分是我的简单试用（个别软件会深度试用），如果对Docker已经有一定的了解了，可以直接跳转到最后面 [翻译之外](#翻译之外) 这部分来查看。

Bookstack → https://github.com/BookStackApp/BookStack

GitHub → https://github.com/linuxserver/docker-bookstack

Docker Hub → https://hub.docker.com/r/linuxserver/bookstack

[Bookstack](https://github.com/BookStackApp/BookStack) 是一个可以创建漂亮文档的免费开源的Wiki系统。它能让团队通过一个简单的所见即所得的编辑器创建适用的文档。

得益于SQL数据库和Markdown编辑器，BookStack是一个让编写文档变为一件并不繁琐的趣事。

可以访问他们的网站来获取更多信息：https://www.bookstackapp.com

------

## 支持的系统架构

得益于docker的跨平台属性，我们的镜像也支持多架构（如，x86-64、arm64、armhf）。

直接拉取 `ghcr.io/linuxserver/bookstack` 应该就可以自动获取适合你系统架构的版本，当然你也可以通过 tag 获取特定的版本。

| 架构   | Tag            |
| ------ | -------------- |
| x86-64 | amd64-latest   |
| arm64  | arm64v8-latest |
| armhf  | arm32v7-latest |

------

## 版本标签

该镜像可通过不同tag获取不同的版本。最新的tag通常提供了最新的稳定版本，其他的可能是正在开发的版本，需要谨慎使用。

| Tag        | 说明                   |
| ---------- | ---------------------- |
| latest     | booksonic 的稳定发行版 |
| prerelease | booksonic的预览发行版  |

------

## 拉取镜像

```shell
docker pull ghcr.io/linuxserver/bookstack
```

------

## 使用方法

以下是一些简单的示例。

### docker-compose（[推荐](general/docker-compose.md)）

兼容docker-compose v2

```yaml
---
version: "2"
services:
  bookstack:
    image: linuxserver/bookstack
    container_name: bookstack
    environment:
      - PUID=1000
      - PGID=1000
      - DB_HOST=bookstack_db
      - DB_USER=bookstack
      - DB_PASS=<yourdbpass>
      - DB_DATABASE=bookstackapp
    volumes:
      - /path/to/data:/config
    ports:
      - 6875:80
    restart: unless-stopped
    depends_on:
      - bookstack_db
  bookstack_db:
    image: linuxserver/mariadb
    container_name: bookstack_db
    environment:
      - PUID=1000
      - PGID=1000
      - MYSQL_ROOT_PASSWORD=<yourdbpass>
      - TZ=Europe/London
      - MYSQL_DATABASE=bookstackapp
      - MYSQL_USER=bookstack
      - MYSQL_PASSWORD=<yourdbpass>
    volumes:
      - /path/to/data:/config
    restart: unless-stopped
```

### docker cli

```shell
docker run -d \
  --name=bookstack \
  -e PUID=1000 \
  -e PGID=1000 \
  -e DB_HOST=<yourdbhost> \
  -e DB_USER=<yourdbuser> \
  -e DB_PASS=<yourdbpass> \
  -e DB_DATABASE=bookstackapp \
  -e APP_URL=http://your.site.here.xyz `#optional` \
  -p 6875:80 \
  -v /path/to/data:/config \
  --restart unless-stopped \
  linuxserver/bookstack
```

## 参数

Docker镜像在使用的时候需要配置一些参数，这些参数使用 `:` 分隔，分别表示 `宿主机:容器内`。例如 `-p 8080:80` 指的是将容器内的`80`端口映射到宿主机上的`8080`端口，即通过宿主机网络访问的话，访问`8080`端口即是访问容器内的`80`端口。

### 端口（`-p`）

| port | 说明                                                         |
| ---- | ------------------------------------------------------------ |
| `80` | 建议将容器内的80端口映射到宿主机的其他端口上去（上例中映射到6875端口） |

### 环境变量（`-e`）

| env                                 | 说明                                       |
| ----------------------------------- | ------------------------------------------ |
| `PUID=1000`                         | 用户的 UID，详见下面的说明                 |
| `PGID=1000`                         | 用户的 GID，详见下面的说明                 |
| `TZ=Europe/London`                  | 设置时区，在国内的话可以使用 Asia/Shanghai |
| `DB_HOST=<yourdbhost>`              | 数据库的地址                               |
| `DB_USER=<yourdbuser>`              | 数据库的用户名                             |
| `DB_PASS=<yourdbpass>`              | 数据库的密码                               |
| `DB_DATABASE=bookstackapp`          | 数据库的名字                               |
| `APP_URL=http://your.site.here.xyz` | 应用的访问地址（使用反向代理时要正确设置） |

### 卷映射（`-v`）

| volume    | 说明                     |
| --------- | ------------------------ |
| `/config` | 存储上传的文件和数据文件 |

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

默认的用户名是 `admin@admin.com` 对应的密码是 `password` 。可以通过 `http://ip:6875` 来访问应用（使用你实际映射出来的端口）。

该应用需要MySQL数据库，可以使用我们的MariaDB镜像。

如果你打算使用反向代理，请设置 `APP_URL` 环境变量，否则可能会设置失败。

BookStack的更多文档介绍：https://www.bookstackapp.com/docs/

### 高级用法（通过 .env 文件设置参数）

如果你希望使用BookStack的其他功能（如，email、Memcache、LDAP等），你需要根据官方的文档来创建 .env 文件。

当你创建容器的时候，不要设置任何SQL数据库的参数或`APP_URL`，容器将把 `.env` 文件复制到 `/config/www/.env` ，你可以直接编辑该文件。

### 渲染PDF

[wkhtmltopdf](https://wkhtmltopdf.org/) 可用于替代原本的PDF渲染器。具体参考：https://www.bookstackapp.com/docs/admin/pdf-rendering/

该镜像中包含 wkhtmltopdf 的路径是 `/usr/bin/wkhtmltopdf` （配置 `.env` 文件用）

------

## 支持

- 进入容器：
  - `docker exec -it bookstack /bin/bash`
- 查看容器日志：
  - `docker logs -f bookstack`
- 查看容器版本号：
  - `docker inspect -f '{% raw %}{{% endraw %}{ index .Config.Labels "build_version" }}' bookstack`
- 查看镜像版本号：
  - `docker inspect -f '{% raw %}{{% endraw %}{ index .Config.Labels "build_version" }}' linuxserver/bookstack`

------

## 翻译之外

BookStack我曾经用过一段时间，整体来说风格很简介，而且功能也够用。

设置语言是在右上角的编辑用户信息中修改，支持中文，而且还有夜晚模式。

总体分为书架 → 图书 → 章节 → 页面，可以很方便地将不同的知识分门别类地进行记录和管理。

拥有一个很强大的编辑器，有代码块和代码高亮，可以插入图片和视频，图片可以直接粘贴到编辑器上自动上传，可以给文章不同的标签，使用标签管理文章，甚至可以在线画流程图。

也拥有比较简单的权限管理，可以给不同类别的用户分配不同的权限。

起初本说明也是打算用BookStack去搭建的，但为了能在GitHub上与大佬交流，才选择了GitBook。

但现在GitBook开始逐渐发展自己的平台，已经不再更新 gitbook-cli 了，也许 BookStack 还真的是不错的替代品。

最后补充一点，BookStack是可以高度定制的，所以你可以做任何事。

![image-20201021125919918](https://pic.watercalmx.com/pic/image-20201021125919918.png)

<img src="https://pic.watercalmx.com/pic/image-20201021130205045.png" alt="image-20201021130205045" style="zoom: 50%;" /><img src="https://pic.watercalmx.com/pic/image-20201021130236141.png" alt="image-20201021130236141" style="zoom: 50%;" />

![image-20201021130524910](https://pic.watercalmx.com/pic/image-20201021130524910.png)

![image-20201021130629963](https://pic.watercalmx.com/pic/image-20201021130629963.png)

![image-20201021130700975](https://pic.watercalmx.com/pic/image-20201021130700975.png)

![image-20201021131254338](https://pic.watercalmx.com/pic/image-20201021131254338.png)

![image-20201021131531114](https://pic.watercalmx.com/pic/image-20201021131531114.png)

![image-20201021131935233](https://pic.watercalmx.com/pic/image-20201021131935233.png)