# linuxserver/mariadb

> [!NOTE]
>
> 因为后续很多开源软件都会使用nginx+mariadb这种组合，所以镜像篇以mariadb为第一篇，关于nginx的可以参考 [部署SWAG](general/swag.md) 这篇文章来了解。

------

MariaDB → https://mariadb.org/

GitHub → https://github.com/linuxserver/docker-mariadb

Docker Hub → https://hub.docker.com/r/linuxserver/mariadb

[MariaDB](https://mariadb.org/)是目前最流行的数据库之一，是MySQL的一个分支。（具体的就不再介绍了，应该都知道吧……网上的教程也很多）

------

## 支持的系统架构

得益于docker的跨平台属性，我们的镜像也支持多架构（如，x86-64、arm64、armhf）。

直接拉取 `linuxserver/mariadb` 应该就可以自动获取适合你系统架构的版本，当然你也可以通过 tag 获取特定的版本。

| 架构   | Tag            |
| ------ | -------------- |
| x86-64 | amd64-latest   |
| arm64  | arm64v8-latest |
| armhf  | arm32v7-latest |

### 拉取镜像

```shell
docker pull linuxserver/mariadb
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
  mariadb:
    image: linuxserver/mariadb
    container_name: mariadb
    environment:
      - PUID=1000
      - PGID=1000
      - MYSQL_ROOT_PASSWORD=ROOT_ACCESS_PASSWORD
      - TZ=Europe/London
      - MYSQL_DATABASE=USER_DB_NAME #optional
      - MYSQL_USER=MYSQL_USER #optional
      - MYSQL_PASSWORD=DATABASE_PASSWORD #optional
      - REMOTE_SQL=http://URL1/your.sql,https://URL2/your.sql #optional
    volumes:
      - path_to_data:/config
    ports:
      - 3306:3306
    restart: unless-stopped
```

### docker cli

```shell
docker run -d \
  --name=mariadb \
  -e PUID=1000 \
  -e PGID=1000 \
  -e MYSQL_ROOT_PASSWORD=ROOT_ACCESS_PASSWORD \
  -e TZ=Europe/London \
  -e MYSQL_DATABASE=USER_DB_NAME `#optional` \
  -e MYSQL_USER=MYSQL_USER `#optional` \
  -e MYSQL_PASSWORD=DATABASE_PASSWORD `#optional` \
  -e REMOTE_SQL=http://URL1/your.sql,https://URL2/your.sql `#optional` \
  -p 3306:3306 \
  -v path_to_data:/config \
  --restart unless-stopped \
  linuxserver/mariadb
```

## 参数

Docker镜像在使用的时候需要配置一些参数，这些参数使用 `:` 分隔，分别表示 `宿主机:容器内`。例如 `-p 8080:80` 指的是将容器内的`80`端口映射到宿主机上的`8080`端口，即通过宿主机网络访问的话，访问`8080`端口即是访问容器内的`80`端口。

### 端口（`-p`）

| port   | 说明              |
| ------ | ----------------- |
| `3306` | mariadb的默认端口 |

### 环境变量（`-e`）

| env                                                     | 说明                                                         |
| ------------------------------------------------------- | ------------------------------------------------------------ |
| `PUID=1000`                                             | 用户的 UID，详见下面的说明                                   |
| `PGID=1000`                                             | 用户的 GID，详见下面的说明                                   |
| `MYSQL_ROOT_PASSWORD=ROOT_ACCESS_PASSWORD`              | 在安装时，设置设置该字段为数据库的root密码。（最少4个字符）  |
| `TZ=Europe/London`                                      | 设置时区，在国内的话可以使用 Asia/Shanghai                   |
| `MYSQL_DATABASE=USER_DB_NAME`                           | 在镜像启动的时候新建的数据库                                 |
| `MYSQL_USER=MYSQL_USER`                                 | 这个用户将拥有 `MYSQL_DATABASE` 设置的数据库的超级权限（不要使用root） |
| `MYSQL_PASSWORD=DATABASE_PASSWORD`                      | `MYSQL_USER` 的密码                                          |
| `REMOTE_SQL=http://URL1/your.sql,https://URL2/your.sql` | 设置通过http/https接受sql文件（用英文逗号分隔）              |

### 卷映射（`-v`）

| volume    | 说明                             |
| --------- | -------------------------------- |
| `/config` | 包含数据库的本身以及所有的设置。 |

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

如果在创建容器时未设置数据库的root密码（日志中会有警告），可以进入Docker容器后使用 `mysqladmin -u root password <PASSWORD>` 设置root密码。

**注意：**如果在容器初始化结束后，通过环境变量 `MYSQL_ROOT_PASSWORD` 修改数据库的root密码将无效！请进入容器后使用 `mysqladmin` 工具更改root密码。

**注意：**如果要使用`MYSQL_DATABASE`、`MYSQL_USER`、`MYSQL_PASSWORD`的话，必须完整设置这三个变量！建议在设置完成后，删除对该变量的引用。

配置文件：`/config/custom.cnf` ；数据库：`/config/databases` ；日志：`/config/log/mysql`

### 从文件加载用户名和密码

`MYSQL_ROOT_PASSWORD` `MYSQL_DATABASE` `MYSQL_USER` `MYSQL_PASSWORD` `REMOTE_SQL`这几个变量可以通过文件设置：

```
/config/env
```

使用下面这种格式：

```
MYSQL_ROOT_PASSWORD="ROOT_ACCESS_PASSWORD"
MYSQL_DATABASE="USER_DB_NAME"
MYSQL_USER="MYSQL_USER"
MYSQL_PASSWORD="DATABASE_PASSWORD"
REMOTE_SQL="http://URL1/your.sql,https://URL2/your.sql"
```

可以根据需要同时使用 `-e` 和通过文件设置，但是将优先文件中的设置。

### 引导新的实例

我们支持使用自定义sql文件进行初始化，将 `*.sql` 文件放在：

```
/config/initdb.d/
```

这将与 `REMOTE_SQL` 有同样的效果，只在容器首次运行的时候生效。

------

## 支持

- 进入容器：
  - `docker exec -it mariadb /bin/bash`
- 查看容器日志：
  - `docker logs -f mariadb`
- 查看容器版本号：
  - `docker inspect -f '{% raw %}{{% endraw %}{ index .Config.Labels "build_version" }}' mariadb`
- 查看镜像版本号：
  - `docker inspect -f '{% raw %}{{% endraw %}{ index .Config.Labels "build_version" }}' linuxserver/mariadb`

------

## 翻译之外

有很多mysql的管理工具，都可以对mariadb进行管理。比如phpmyadmin，其官方也提供了docker镜像，我们可以使用phpmyadmin对mariadb进行管理，只要将他们加入同一个虚拟网络并做相应配置即可。

官网 → https://www.phpmyadmin.net/

Docker Hub → https://hub.docker.com/r/phpmyadmin/phpmyadmin

### 拉取镜像

```shell
docker pull phpmyadmin/phpmyadmin
```

### 创建容器

通过这一条命令即可：

```shell
docker run -d \
    --name phpmyadmin \
    -p 8080:80 \
    phpmyadmin/phpmyadmin
```

但这样直接创建的容器，每次打开的时候还需要设置数据库地址、用户名、密码等来登陆。

也可以在创建容器的时候通过 `-e` 把这些都设置了，这样打开phpmyadmin便可以直接进行管理。

```shell
docker run -d \
    --name phpmyadmin \
    -e PMA_HOST=<数据库IP> \
    -e PMA_PORT=<数据库端口> \
    -e PMA_USER=<用户名> \
    -e PMA_PASSWORD=<密码> \
    -p 8080:80 \
    phpmyadmin/phpmyadmin
```

> [!TIP]
>
> 如果mariadb和phpmyadmin都在同一个虚拟网络下的话，那便不需要将mariadb的3306端口映射到宿主机上。因为在同一个虚拟网络下，不同容器之间是可以互相进行网络通信的。

![image-20201019155431457](https://pic.watercalmx.com/pic/image-20201019155431457.png)