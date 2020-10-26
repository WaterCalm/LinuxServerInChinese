# linuxserver/clarkson

> [!TIP]
>
> 前半部分是翻译官方的文档，最后一部分是我的简单试用（个别软件会深度试用），如果对Docker已经有一定的了解了，可以直接跳转到最后面 [翻译之外](#翻译之外) 这部分来查看。

Clarkson → https://github.com/linuxserver/Clarkson

GitHub → https://github.com/linuxserver/docker-clarkson

Docker Hub → https://hub.docker.com/r/linuxserver/clarkson

[Clarkson](https://github.com/linuxserver/Clarkson) 是一个基于WEB的仪表盘，它用简洁的界面记录你所有车辆的加油记录。这个应用支持多用户以及每位用户的多辆车。每当加完油后，你可以保留收据并将数据记录到Clarkson中。

------

## 支持的系统架构

得益于docker的跨平台属性，我们的镜像也支持多架构（如，x86-64、arm64、armhf）。

直接拉取 `linuxserver/clarkson` 应该就可以自动获取适合你系统架构的版本，当然你也可以通过 tag 获取特定的版本。

| 架构   | Tag            |
| ------ | -------------- |
| x86-64 | amd64-latest   |
| arm64  | arm64v8-latest |
| armhf  | arm32v7-latest |

------

## 拉取镜像

```shell
docker pull linuxserver/clarkson
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
  clarkson:
    image: linuxserver/clarkson
    container_name: clarkson
    environment:
      - PUID=1000
      - PGID=1000
      - MYSQL_HOST=<mysql_host>
      - MYSQL_USERNAME=<mysql_username>
      - MYSQL_PASSWORD=<mysql_password>
      - ENABLE_REGISTRATIONS=<true/false>
      - TZ=Europe/London
    ports:
      - 3000:3000
    restart: unless-stopped
```

### docker cli

```shell
docker run -d \
  --name=clarkson \
  -e PUID=1000 \
  -e PGID=1000 \
  -e MYSQL_HOST=<mysql_host> \
  -e MYSQL_USERNAME=<mysql_username> \
  -e MYSQL_PASSWORD=<mysql_password> \
  -e ENABLE_REGISTRATIONS=<true/false> \
  -e TZ=Europe/London \
  -p 3000:3000 \
  --restart unless-stopped \
  linuxserver/clarkson
```

## 参数

Docker镜像在使用的时候需要配置一些参数，这些参数使用 `:` 分隔，分别表示 `宿主机:容器内`。例如 `-p 8080:80` 指的是将容器内的`80`端口映射到宿主机上的`8080`端口，即通过宿主机网络访问的话，访问`8080`端口即是访问容器内的`80`端口。

### 端口（`-p`）

| port   | 说明    |
| ------ | ------- |
| `3000` | WEB界面 |

### 环境变量（`-e`）

| env                                 | 说明                                            |
| ----------------------------------- | ----------------------------------------------- |
| `PUID=1000`                         | 用户的 UID，详见下面的说明                      |
| `PGID=1000`                         | 用户的 GID，详见下面的说明                      |
| `TZ=Europe/London`                  | 设置时区，在国内的话可以使用 Asia/Shanghai      |
| `MYSQL_HOST=<mysql_host>`           | MySQL数据库的地址                               |
| `MYSQL_USERNAME=<mysql_username>`   | 对 clarkson 有权限的用户名                      |
| `MYSQL_PASSWORD=<mysql_password>`   | 数据库的密码                                    |
| `ENABLE_REGISTRATIONS=<true/false>` | 默认为false，如果设置为true，将允许新用户注册。 |

### 卷映射（`-v`）

| volume | 说明           |
| ------ | -------------- |
| 无     | 无需要映射的卷 |

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

Clarkson 需要 v5.7.* 版本的MySQL数据库，请确保数据库已经启动

开始之前请确保已经创建了 clarkson 数据库，并创建对数据库有所有权限的用户。可以用这几条命令来创建数据库：

```sql
CREATE SCHEMA `clarkson`;
CREATE USER 'clarkson_user' IDENTIFIED BY 'supersecretpassword';
GRANT ALL ON `clarkson`.* TO 'clarkson_user';
```

运行后，容器将开始迁移数据到你的数据库中，完成后应用便会自动启动。你需要注册一个管理员用户，之后再注册的用户都是普通标准用户。通过 `ENABLE_REGISTRATIONS` 可以设置是否允许新用户注册，但这并不会隐藏注册按钮，只会禁用注册功能。

------

## 支持

- 进入容器：
  - `docker exec -it clarkson /bin/bash`
- 查看容器日志：
  - `docker logs -f clarkson`
- 查看容器版本号：
  - `docker inspect -f '{% raw %}{{% endraw %}{ index .Config.Labels "build_version" }}' clarkson`
- 查看镜像版本号：
  - `docker inspect -f '{% raw %}{{% endraw %}{ index .Config.Labels "build_version" }}' linuxserver/clarkson`

------

## 翻译之外

首次启动的时候，已经要记得启用注册……否则连管理用户都注册不了……（`-e ENABLE_REGISTRATIONS=true`）

整体来说使用很简单，虽然没有中文面板，但操作很简单：先添加车辆，然后添加加油记录就好了，可以记录加了多少油、跑了多少公里、用了多少钱之类的。

在开始记录前，一定要去设置里把各个单位都设置好，不然以后更改的时候无法更新之前已经输入的记录的单位（货币的单位只有美元和欧元这些，没有人民币……）。

这个可能更适合……车老板使用吧……



![image-20201023111321246](https://pic.watercalmx.com/pic/image-20201023111321246.png)

![image-20201023111334298](https://pic.watercalmx.com/pic/image-20201023111334298.png)

![image-20201023111748666](https://pic.watercalmx.com/pic/image-20201023111748666.png)

![image-20201023111842487](https://pic.watercalmx.com/pic/image-20201023111842487.png)

![image-20201023111918378](https://pic.watercalmx.com/pic/image-20201023111918378.png)