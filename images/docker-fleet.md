# linuxserver/fleet

> [!TIP]
>
> 前半部分是翻译官方的文档，最后一部分是我的简单试用（个别软件会深度试用），如果对Docker已经有一定的了解了，可以直接跳转到最后面 [翻译之外](#翻译之外) 这部分来查看。

Fleet → https://github.com/linuxserver/fleet

GitHub → https://github.com/linuxserver/docker-fleet

Docker Hub → https://hub.docker.com/r/linuxserver/fleet

[Fleet](https://github.com/linuxserver/fleet) 提供了一个web界面来显示一个或多个镜像仓库的镜像信息。

------

## 支持的系统架构

得益于docker的跨平台属性，我们的镜像也支持多架构（如，x86-64、arm64、armhf）。

直接拉取 `ghcr.io/linuxserver/fleet` 应该就可以自动获取适合你系统架构的版本，当然你也可以通过 tag 获取特定的版本。

| 架构   | Tag            |
| ------ | -------------- |
| x86-64 | amd64-latest   |
| arm64  | arm64v8-latest |
| armhf  | arm32v7-latest |

------

## 拉取镜像

```shell
docker pull ghcr.io/linuxserver/fleet
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
  fleet:
    image: ghcr.io/linuxserver/fleet
    container_name: fleet
    environment:
      - PUID=1000
      - PGID=1000
      - fleet_admin_authentication_type=DATABASE
      - fleet_database_url=jdbc:mariadb://<url>:3306/fleet
      - fleet_database_username=fleet_user
      - fleet_database_password=dbuserpassword
      - fleet_admin_secret=randomstring #optional
    volumes:
      - </path/to/appdata/config>:/config
    ports:
      - 8080:8080
    restart: unless-stopped
```

### docker cli

```shell
docker run -d \
  --name=fleet \
  -e PUID=1000 \
  -e PGID=1000 \
  -e fleet_admin_authentication_type=DATABASE \
  -e fleet_database_url=jdbc:mariadb://<url>:3306/fleet \
  -e fleet_database_username=fleet_user \
  -e fleet_database_password=dbuserpassword \
  -e fleet_admin_secret=randomstring `#optional` \
  -p 8080:8080 \
  -v </path/to/appdata/config>:/config \
  --restart unless-stopped \
  ghcr.io/linuxserver/fleet
```

## 参数

Docker镜像在使用的时候需要配置一些参数，这些参数使用 `:` 分隔，分别表示 `宿主机:容器内`。例如 `-p 8080:80` 指的是将容器内的`80`端口映射到宿主机上的`8080`端口，即通过宿主机网络访问的话，访问`8080`端口即是访问容器内的`80`端口。

### 端口（`-p`）

| port   | 说明    |
| ------ | ------- |
| `8080` | Web界面 |

### 环境变量（`-e`）

| env                                                  | 说明                                                         |
| ---------------------------------------------------- | ------------------------------------------------------------ |
| `PUID=1000`                                          | 用户的 UID，详见下面的说明                                   |
| `PGID=1000`                                          | 用户的 GID，详见下面的说明                                   |
| `TZ=Europe/London`                                   | 设置时区，在国内的话可以使用 Asia/Shanghai                   |
| `fleet_admin_authentication_type=DATABASE`           | 定义Fleet如何管理用户登录的开关。如果设置为DATABASE，请参阅相关的可选参数。可以设置为DATABASE或PROPERTIES。 |
| `fleet_database_url=jdbc:mariadb://<url>:3306/fleet` | 连接到Fleet数据库的JDBC连接字符串                            |
| `fleet_database_username=fleet_user`                 | 具有数据库相关GRANT权限的用户名                              |
| `fleet_database_password=dbuserpassword`             | 数据库用户的密码。                                           |
| `fleet_admin_secret=randomstring`                    | 随机输入的字符串，作为生成密码的加密字符串。                 |

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

访问 `http://ip:8080` 进入首页

如果选择使用 `DATABASE` 作为用户验证程序，访问 `http://ip:8080/setup` 来设置初始用户，完成后该页面也将停用。最好能重新启动容器，这样它将该页面完全删除。

完成后，即可在 `http://ip:8080/setup`  进行登录，来管理镜像仓库。

------

## 支持

- 进入容器：
  - `docker exec -it fleet /bin/bash`
- 查看容器日志：
  - `docker logs -f fleet`
- 查看容器版本号：
  - `docker inspect -f '{% raw %}{{% endraw %}{ index .Config.Labels "build_version" }}' fleet`
- 查看镜像版本号：
  - `docker inspect -f '{% raw %}{{% endraw %}{ index .Config.Labels "build_version" }}' ghcr.io/linuxserver/fleet`

------

## 翻译之外

关于Fleet在 [Fleet](../general/fleet.md) 这篇文章中已经介绍过了，所以就不再试用了，一般来说个人用户应该不需要使用它。
