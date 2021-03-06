# linuxserver/librespeed

> [!TIP]
>
> 前半部分是翻译官方的文档，最后一部分是我的简单试用（个别软件会深度试用），如果对Docker已经有一定的了解了，可以直接跳转到最后面 [翻译之外](#翻译之外) 这部分来查看。

Librespeed → https://github.com/librespeed/speedtest

GitHub → https://github.com/linuxserver/docker-librespeed

Docker Hub → https://hub.docker.com/r/linuxserver/librespeed

[Librespeed](https://github.com/librespeed/speedtest) 是一个非常轻量级的测速工具，基于 Javascript ，使用 XMLHttpRequest 和 Web Workers 。没有Flash，没有Java，没有Websocket。

------

## 支持的系统架构

得益于docker的跨平台属性，我们的镜像也支持多架构（如，x86-64、arm64、armhf）。

直接拉取 `ghcr.io/linuxserver/librespeed` 应该就可以自动获取适合你系统架构的版本，当然你也可以通过 tag 获取特定的版本。

| 架构   | Tag            |
| ------ | -------------- |
| x86-64 | amd64-latest   |
| arm64  | arm64v8-latest |
| armhf  | arm32v7-latest |



------

## 拉取镜像

```shell
docker pull ghcr.io/linuxserver/librespeed
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
  librespeed:
    image: ghcr.io/linuxserver/librespeed
    container_name: librespeed
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
      - PASSWORD=PASSWORD
      - CUSTOM_RESULTS=true #optional
      - DB_TYPE=sqlite #optional
      - DB_NAME=DB_NAME #optional
      - DB_HOSTNAME=DB_HOSTNAME #optional
      - DB_USERNAME=DB_USERNAME #optional
      - DB_PASSWORD=DB_PASSWORD #optional
    volumes:
      - /path/to/appdata/config:/config
    ports:
      - 80:80
    restart: unless-stopped
```

### docker cli

```shell
docker run -d \
  --name=librespeed \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Europe/London \
  -e PASSWORD=PASSWORD \
  -e CUSTOM_RESULTS=true `#optional` \
  -e DB_TYPE=sqlite `#optional` \
  -e DB_NAME=DB_NAME `#optional` \
  -e DB_HOSTNAME=DB_HOSTNAME `#optional` \
  -e DB_USERNAME=DB_USERNAME `#optional` \
  -e DB_PASSWORD=DB_PASSWORD `#optional` \
  -p 80:80 \
  -v /path/to/appdata/config:/config \
  --restart unless-stopped \
  ghcr.io/linuxserver/librespeed
```

## 参数

Docker镜像在使用的时候需要配置一些参数，这些参数使用 `:` 分隔，分别表示 `宿主机:容器内`。例如 `-p 8080:80` 指的是将容器内的`80`端口映射到宿主机上的`8080`端口，即通过宿主机网络访问的话，访问`8080`端口即是访问容器内的`80`端口。

### 端口（`-p`）

| port | 说明          |
| ---- | ------------- |
| `80` | WebUI访问端口 |

### 环境变量（`-e`）

| env                       | 说明                                                         |
| ------------------------- | ------------------------------------------------------------ |
| `PUID=1000`               | 用户的 UID，详见下面的说明                                   |
| `PGID=1000`               | 用户的 GID，详见下面的说明                                   |
| `TZ=Europe/London`        | 设置时区，在国内的话可以使用 Asia/Shanghai                   |
| `PASSWORD=PASSWORD`       | 设置结果数据库的密码                                         |
| `CUSTOM_RESULTS=true`     | （可选）启用`/config/www/results/index.php`中的自定义结果页面 |
| `DB_TYPE=sqlite`          | 默认为`sqlite`，也可以设置为`mysql`或`postgresql`            |
| `DB_NAME=DB_NAME`         | 数据库名称，对于`mysql`和`pgsql`是必需的                     |
| `DB_HOSTNAME=DB_HOSTNAME` | 数据库地址，对于`mysql`和`pgsql`是必需的                     |
| `DB_USERNAME=DB_USERNAME` | 数据库用户名，对于`mysql`和`pgsql`是必需的                   |
| `DB_PASSWORD=DB_PASSWORD` | 数据库密码，对于`mysql`和`pgsql`是必需的                     |

### 卷映射（`-v`）

| volume    | 说明             |
| --------- | ---------------- |
| `/config` | 配置文件所在路径 |




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

访问地址：`http://ip:9117`，可以使用设置的密码从 `http://SERVERIP/results/stats.php` 访问结果数据库。

默认的模板基于`example-singleServer-full.html`，但 `/config/www/` 提供了所有模板供参考。随意自定义`/config/www/index.html` 文件。若要恢复默认模板，删除该文件即可。

您可以选择将自定义的`speedtest.js`和`speedtest_worker.js`文件放在 `/config/www` 下，它们将在容器启动后取代默认设置。请记住，一旦您这样做，它们将不再被更新。但可以删除它们并重新创建容器以返回到镜像的默认值。

如果要设置mysql或postgresql数据库，则首先需要按照以下链接中的描述将表导入数据库中：https://github.com/librespeed/speedtest/blob/master/doc.md#creating-the-database

要启用自定义的结果页面，请设置环境变量`CUSTOM_RESULTS = true`并至少一次启动（或重新启动）容器，以创建 `/config/www/results/index.php` 并根据您的喜好修改此文件。

------

## 支持

- 进入容器：
  - `docker exec -it librespeed /bin/bash`
- 查看容器日志：
  - `docker logs -f librespeed`
- 查看容器版本号：
  - `docker inspect -f '{% raw %}{{% endraw %}{ index .Config.Labels "build_version" }}' librespeed`
- 查看镜像版本号：
  - `docker inspect -f '{% raw %}{{% endraw %}{ index .Config.Labels "build_version" }}' ghcr.io/linuxserver/librespeed`

------

## 翻译之外

暂未试用