# linuxserver/duckdns

> [!TIP]
>
> 前半部分是翻译官方的文档，最后一部分是我的简单试用（个别软件会深度试用），如果对Docker已经有一定的了解了，可以直接跳转到最后面 [翻译之外](#翻译之外) 这部分来查看。

Duckdns → https://duckdns.org/

GitHub → https://github.com/linuxserver/docker-duckdns

Docker Hub → https://hub.docker.com/r/linuxserver/duckdns

[Duckdns](https://duckdns.org/) 是一个免费的DNS服务商。它可以解析duckdns.org的子域名到你指定的IP上。这个服务是完全免费的并且永久有效。

------

## 支持的系统架构

得益于docker的跨平台属性，我们的镜像也支持多架构（如，x86-64、arm64、armhf）。

直接拉取 `ghcr.io/linuxserver/duckdns` 应该就可以自动获取适合你系统架构的版本，当然你也可以通过 tag 获取特定的版本。

| 架构   | Tag            |
| ------ | -------------- |
| x86-64 | amd64-latest   |
| arm64  | arm64v8-latest |
| armhf  | arm32v7-latest |

------

## 拉取镜像

```shell
docker pull ghcr.io/linuxserver/duckdns
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
  duckdns:
    image: linuxserver/duckdns
    container_name: duckdns
    environment:
      - PUID=1000 #optional
      - PGID=1000 #optional
      - TZ=Europe/London
      - SUBDOMAINS=subdomain1,subdomain2
      - TOKEN=token
      - LOG_FILE=false #optional
    volumes:
      - /path/to/appdata/config:/config #optional
    restart: unless-stopped
```

### docker cli

```shell
docker run -d \
  --name=duckdns \
  -e PUID=1000 `#optional` \
  -e PGID=1000 `#optional` \
  -e TZ=Europe/London \
  -e SUBDOMAINS=subdomain1,subdomain2 \
  -e TOKEN=token \
  -e LOG_FILE=false `#optional` \
  -v /path/to/appdata/config:/config `#optional` \
  --restart unless-stopped \
  linuxserver/duckdns
```

## 参数

Docker镜像在使用的时候需要配置一些参数，这些参数使用 `:` 分隔，分别表示 `宿主机:容器内`。例如 `-p 8080:80` 指的是将容器内的`80`端口映射到宿主机上的`8080`端口，即通过宿主机网络访问的话，访问`8080`端口即是访问容器内的`80`端口。

### 端口（`-p`）

| port | 说明 |
| ---- | ---- |
| 无   | 无   |

### 环境变量（`-e`）

| env                                | 说明                                                         |
| ---------------------------------- | ------------------------------------------------------------ |
| `PUID=1000`                        | 用户的 UID，详见下面的说明                                   |
| `PGID=1000`                        | 用户的 GID，详见下面的说明                                   |
| `TZ=Europe/London`                 | 设置时区，在国内的话可以使用 Asia/Shanghai                   |
| `SUBDOMAINS=subdomain1,subdomain2` | 允许多个子域名，用英文逗号分隔，不要出现空格                 |
| `TOKEN=token`                      | DuckDNS的token                                               |
| `LOG_FILE=false`                   | 设置成 `true` 则将日志输出为文件（需要映射 `/config` 文件夹） |

### 卷映射（`-v`）

| volume    | 说明                               |
| --------- | ---------------------------------- |
| `/config` | 配置文件所在路径，配合输出日志使用 |

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

- 在 [duckdns](https://duckdns.org/) 的网站上，注册域名并获取 token
- 用子域名和token创建容器
- 将每5分钟更新你的IP地址到 DuckDNS 上

------

## 支持

- 进入容器：
  - `docker exec -it duckdns /bin/bash`
- 查看容器日志：
  - `docker logs -f duckdns`
- 查看容器版本号：
  - `docker inspect -f '{% raw %}{{% endraw %}{ index .Config.Labels "build_version" }}' duckdns`
- 查看镜像版本号：
  - `docker inspect -f '{% raw %}{{% endraw %}{ index .Config.Labels "build_version" }}' linuxserver/duckdns`

------

## 翻译之外

DuckDNS是一个比较有名的二级域名供应商，国内类似的有花生壳域名。

具体功能在这里就不介绍了，如果不愿意花钱买顶级域名，这个是不错的选择。

用GitHub账号、谷歌账号、twitter账号都能直接登陆。

申请域名也十分简单，只要填写想要的子域名，然后直接 Add 就行，如果没有重复的，你就拥有了一个你自己的二级域名。

这个容器便是可以自动更新当前网络环境的IP到DuckDNS上的工具，使用也是非常简单的，只要设置好环境变量，直接创建容器就行。（因为duckdns在gfw列表中，如果在路由器端开启了科学上网的话，要注意在相关页面进行下设置，不然会一直更新的是科学上网的ip地址，而不是本机真实的ip地址……）

![image-20201029164212382](https://pic.watercalmx.com/pic/image-20201029164212382.png)

![image-20201029164359910](https://pic.watercalmx.com/pic/image-20201029164359910.png)