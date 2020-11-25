# linuxserver/diskover

> [!TIP]
>
> 前半部分是翻译官方的文档，最后一部分是我的简单试用（个别软件会深度试用），如果对Docker已经有一定的了解了，可以直接跳转到最后面 [翻译之外](#翻译之外) 这部分来查看。

diskover → https://github.com/shirosaidev/diskover

GitHub → https://github.com/linuxserver/docker-diskover

Docker Hub → https://hub.docker.com/r/linuxserver/diskover

[diskover](https://github.com/shirosaidev/diskover) 是一个文件系统搜寻器和磁盘空间管理软件。它使用 Elasticsearch 在跨异构存储系统中索引和管理数据。

------

## 支持的系统架构

得益于docker的跨平台属性，我们的镜像也支持多架构（如，x86-64、arm64、armhf）。

直接拉取 `ghcr.io/linuxserver/diskover` 应该就可以自动获取适合你系统架构的版本，当然你也可以通过 tag 获取特定的版本。

| 架构   | Tag            |
| ------ | -------------- |
| x86-64 | amd64-latest   |
| arm64  | arm64v8-latest |
| armhf  | arm32v7-latest |


------

## 拉取镜像

```shell
docker pull ghcr.io/linuxserver/diskover
```

------

## 使用方法

以下是一些简单的示例。

### docker-compose（[推荐](general/docker-compose.md)）

兼容docker-compose v2

```yaml
version: '2'
services:
  diskover:
    image: linuxserver/diskover
    container_name: diskover
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
      - REDIS_HOST=redis
      - REDIS_PORT=6379
      - ES_HOST=elasticsearch
      - ES_PORT=9200
      - ES_USER=elastic
      - ES_PASS=changeme
      - RUN_ON_START=true
      - USE_CRON=true
    volumes:
      - /path/to/diskover/config:/config
      - /path/to/diskover/data:/data
    ports:
      - 80:80
      - 9181:9181
      - 9999:9999
    mem_limit: 4096m
    restart: unless-stopped
    depends_on:
      - elasticsearch
      - redis
  elasticsearch:
    container_name: elasticsearch
    image: docker.elastic.co/elasticsearch/elasticsearch:5.6.9
    volumes:
      - ${DOCKER_HOME}/elasticsearch/data:/usr/share/elasticsearch/data
    environment:
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms2048m -Xmx2048m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
  redis:
    container_name: redis
    image: redis:alpine
    volumes:
      - ${HOME}/docker/redis:/data
```

### docker cli

```shell
docker run -d \
  --name=diskover \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Europe/London \
  -e REDIS_HOST=redis \
  -e REDIS_PORT=6379 \
  -e ES_HOST=elasticsearch \
  -e ES_PORT=9200 \
  -e ES_USER=elastic \
  -e ES_PASS=changeme \
  -e INDEX_NAME=diskover- \
  -e DISKOVER_OPTS= \
  -e WORKER_OPTS= \
  -e RUN_ON_START=true \
  -e USE_CRON=true \
  -p 80:80 \
  -p 9181:9181 \
  -p 9999:9999 \
  -v /path/to/diskover/config:/config \
  -v /path/to/diskover/data:/data \
  --restart unless-stopped \
  linuxserver/diskover
```

## 参数

Docker镜像在使用的时候需要配置一些参数，这些参数使用 `:` 分隔，分别表示 `宿主机:容器内`。例如 `-p 8080:80` 指的是将容器内的`80`端口映射到宿主机上的`8080`端口，即通过宿主机网络访问的话，访问`8080`端口即是访问容器内的`80`端口。

### 端口（`-p`）

| port   | 说明                  |
| ------ | --------------------- |
| `80`   | diskover Web界面      |
| `9181` | rq-dashboard Web界面  |
| `9999` | diskover socket服务器 |

### 环境变量（`-e`）

| env                     | 说明                                       |
| ----------------------- | ------------------------------------------ |
| `PUID=1000`             | 用户的 UID，详见下面的说明                 |
| `PGID=1000`             | 用户的 GID，详见下面的说明                 |
| `TZ=Europe/London`      | 设置时区，在国内的话可以使用 Asia/Shanghai |
| `REDIS_HOST=redis`      | Redis 地址（可选）                         |
| `REDIS_PORT=6379`       | Redis 端口（可选）                         |
| `ES_HOST=elasticsearch` | ElasticSearch 地址（可选）                 |
| `ES_PORT=9200`          | ElasticSearch 端口（可选）                 |
| `ES_USER=elastic`       | ElasticSearch 用户名（可选）               |
| `ES_PASS=changeme`      | ElasticSearch 密码（可选）                 |
| `INDEX_NAME=diskover-`  | 索引名称前缀（可选）                       |
| `DISKOVER_OPTS=`        | 传递给 diskover 搜寻器的参数（可选）       |
| `WORKER_OPTS=`          | 传递给 diskover bots 启动器的参数（可选）  |
| `RUN_ON_START=true`     | 容器启动时，启动搜寻器（可选）             |
| `USE_CRON=true`         | 使 搜寻器 作为Cron启动（可选）             |

### 卷映射（`-v`）

| volume    | 说明               |
| --------- | ------------------ |
| `/config` | 配置文件所在路径   |
| `/data`   | 搜寻器的默认挂载点 |

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

访问地址 `http://ip`

初始化将花费一段时间，如果打开页面无响应，请稍后刷新重试。

我们强烈建议使用 docker compose 进行部署，因为它需要链接到多个数据库后端。

如果你在宿主机上安装的 elasticsearch 和 redis，那么他们不支持设置自定义的uid和gid，他们将默认运行为：

- Redis UID=999  GID=999
- Elasticsearch UID=1000 GID=1000

ElasticSearch还需要主机上的sysctl设置才能正常运行。使用 `sysctl -w vm.max_map_count=262144` 会解决这个问题。要使此设置在重新启动后保持不变，请在 `/etc/sysctl.conf` 中设置此值。

如果仅是希望应用程序正常运行，则可以将它们安装到具有`0777`权限的文件夹中，否则，您将需要创建这些用户的主机级别并正确设置文件夹所有权。

上面 compose 的示例中，是指向单个目录的，并且你传递给diskover容器的UID和GID需要与该文件夹的所有权相匹配。如果这些是具有许多所有者的共享文件夹，则索引编制可能会失败。

在您的环境中特殊问题或设置，请参阅[项目的Github页面](https://github.com/shirosaidev/diskover)。

------

## 支持

- 进入容器：
  - `docker exec -it diskover /bin/bash`
- 查看容器日志：
  - `docker logs -f diskover`
- 查看容器版本号：
  - `docker inspect -f '{% raw %}{{% endraw %}{ index .Config.Labels "build_version" }}' diskover`
- 查看镜像版本号：
  - `docker inspect -f '{% raw %}{{% endraw %}{ index .Config.Labels "build_version" }}' ghcr.io/linuxserver/diskover`

------

## 翻译之外

说实话……不知道这个东西咋用……

