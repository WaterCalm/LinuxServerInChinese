# linuxserver/code-server

> [!TIP]
>
> 前半部分是翻译官方的文档，最后一部分是我的简单试用（个别软件会深度试用），如果对Docker已经有一定的了解了，可以直接跳转到最后面 [翻译之外](#翻译之外) 这部分来查看。

Code-server → https://coder.com/

GitHub → https://github.com/linuxserver/docker-code-server

Docker Hub → https://hub.docker.com/r/linuxserver/code-server

[Code-server](https://coder.com/) 是在远程服务器上运行的VS Code，通过浏览器访问。

- 在你的Chromebook、平板、笔记本上用一样的环境来进行开发
- 如果你有Windows或Mac工作站，也可以轻松地在Linux上开发
- 利用云服务器来加快测试、编译、下载等操作
- 不论在哪永远在线
- 所有的计算都在云端进行
- 不需要再运行过多的Chrome实例

------

## 支持的系统架构

得益于docker的跨平台属性，我们的镜像也支持多架构（如，x86-64、arm64、armhf）。

直接拉取 `ghcr.io/linuxserver/code-server` 应该就可以自动获取适合你系统架构的版本，当然你也可以通过 tag 获取特定的版本。

| 架构   | Tag            |
| ------ | -------------- |
| x86-64 | amd64-latest   |
| arm64  | arm64v8-latest |
| armhf  | arm32v7-latest |

------

## 版本标签

该镜像可通过不同tag获取不同的版本。最新的tag通常提供了最新的稳定版本，其他的可能是正在开发的版本，需要谨慎使用。

| Tag         | 说明                 |
| ----------- | -------------------- |
| latest      | 稳定发行版           |
| development | GitHub上的预览发行版 |

------

## 拉取镜像

```shell
docker pull ghcr.io/linuxserver/code-server
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
  code-server:
    image: linuxserver/code-server
    container_name: code-server
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
      - PASSWORD=password #optional
      - SUDO_PASSWORD=password #optional
      - PROXY_DOMAIN=code-server.my.domain #optional
    volumes:
      - /path/to/appdata/config:/config
    ports:
      - 8443:8443
    restart: unless-stopped
```

### docker cli

```shell
docker run -d \
  --name=code-server \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Europe/London \
  -e PASSWORD=password `#optional` \
  -e SUDO_PASSWORD=password `#optional` \
  -e PROXY_DOMAIN=code-server.my.domain `#optional` \
  -p 8443:8443 \
  -v /path/to/appdata/config:/config \
  --restart unless-stopped \
  linuxserver/code-server
```

## 参数

Docker镜像在使用的时候需要配置一些参数，这些参数使用 `:` 分隔，分别表示 `宿主机:容器内`。例如 `-p 8080:80` 指的是将容器内的`80`端口映射到宿主机上的`8080`端口，即通过宿主机网络访问的话，访问`8080`端口即是访问容器内的`80`端口。

### 端口（`-p`）

| port   | 说明    |
| ------ | ------- |
| `8443` | WEB界面 |

### 环境变量（`-e`）

| env                                  | 说明                                                         |
| ------------------------------------ | ------------------------------------------------------------ |
| `PUID=1000`                          | 用户的 UID，详见下面的说明                                   |
| `PGID=1000`                          | 用户的 GID，详见下面的说明                                   |
| `TZ=Europe/London`                   | 设置时区，在国内的话可以使用 Asia/Shanghai                   |
| `PASSWORD=password`                  | Web界面的密码，如果不设置则不会启用认证                      |
| `SUDO_PASSWORD=password`             | 如果设置了该项，则在服务器的终端上用该密码使用sudo           |
| `PROXY_DOMAIN=code-server.my.domain` | 反向代理相关的设置，[具体参阅这里](https://github.com/cdr/code-server/blob/master/doc/FAQ.md#sub-domains) |

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

访问WEB界面：`http://ip:8443` 

如果要使用GitHub，把ssh密钥放在 /config/.ssh 文件夹

然后在菜单中打开终端，设置github的用户名和邮箱

```
git config --global user.name "username"
git config --global user.email "email address"
```



------

## 支持

- 进入容器：
  - `docker exec -it code-server /bin/bash`
- 查看容器日志：
  - `docker logs -f code-server`
- 查看容器版本号：
  - `docker inspect -f '{% raw %}{{% endraw %}{ index .Config.Labels "build_version" }}' code-server`
- 查看镜像版本号：
  - `docker inspect -f '{% raw %}{{% endraw %}{ index .Config.Labels "build_version" }}' ghcr.io/linuxserver/code-server`

------

## 翻译之外

和code9一样，都是基于WEB的远程开发IDE

但VS Code本体现在已经有了远程开发的插件，这个的用处已经不大了吧？

emmm……倒也不是……可以用手机访问……然后……

![image-20201024135232779](https://pic.watercalmx.com/pic/image-20201024135232779.png)

![image-20201024143009029](https://pic.watercalmx.com/pic/image-20201024143009029.png)