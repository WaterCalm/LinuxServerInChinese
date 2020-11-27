# linuxserver/ldap-auth

> [!TIP]
>
> 前半部分是翻译官方的文档，最后一部分是我的简单试用（个别软件会深度试用），如果对Docker已经有一定的了解了，可以直接跳转到最后面 [翻译之外](#翻译之外) 这部分来查看。

Ldap-auth → https://github.com/nginxinc/nginx-ldap-auth

GitHub → https://github.com/linuxserver/docker-ldap-auth

Docker Hub → https://hub.docker.com/r/linuxserver/ldap-auth

[Ldap-auth](https://github.com/nginxinc/nginx-ldap-auth) 用于验证从nginx代理服务器请求受保护资源的用户的身份。它包括一个与身份验证服务器通信的守护程序（ldap-auth）和一个基于用户凭据生成身份验证cookie的网络服务器守护程序。这些守护程序是用Python编写的，可与轻型目录访问协议（LDAP）身份验证服务器（OpenLDAP或Microsoft Windows Active Directory 2003和2012）一起使用。

------

## 支持的系统架构

得益于docker的跨平台属性，我们的镜像也支持多架构（如，x86-64、arm64、armhf）。

直接拉取 `ghcr.io/linuxserver/ldap-auth` 应该就可以自动获取适合你系统架构的版本，当然你也可以通过 tag 获取特定的版本。

| 架构   | Tag            |
| ------ | -------------- |
| x86-64 | amd64-latest   |
| arm64  | arm64v8-latest |
| armhf  | arm32v7-latest |



------

## 拉取镜像

```shell
docker pull ghcr.io/linuxserver/ldap-auth
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
  ldap-auth:
    image: ghcr.io/linuxserver/ldap-auth
    container_name: ldap-auth
    environment:
      - TZ=Europe/London
      - FERNETKEY= #optional
      - CERTFILE= #optional
      - KEYFILE= #optional
    ports:
      - 8888:8888
      - 9000:9000
    restart: unless-stopped
```

### docker cli

```shell
docker run -d \
  --name=ldap-auth \
  -e TZ=Europe/London \
  -e FERNETKEY= `#optional` \
  -e CERTFILE= `#optional` \
  -e KEYFILE= `#optional` \
  -p 8888:8888 \
  -p 9000:9000 \
  --restart unless-stopped \
  ghcr.io/linuxserver/ldap-auth
```

## 参数

Docker镜像在使用的时候需要配置一些参数，这些参数使用 `:` 分隔，分别表示 `宿主机:容器内`。例如 `-p 8080:80` 指的是将容器内的`80`端口映射到宿主机上的`8080`端口，即通过宿主机网络访问的话，访问`8080`端口即是访问容器内的`80`端口。

### 端口（`-p`）

| port   | 说明           |
| ------ | -------------- |
| `8888` | 守护程序的端口 |
| `9000` | 登录页面的端口 |

### 环境变量（`-e`）

| env                | 说明                                                         |
| ------------------ | ------------------------------------------------------------ |
| `TZ=Europe/London` | 设置时区，在国内的话可以使用 Asia/Shanghai                   |
| `FERNETKEY=`       | （可选）定义一个自定义的Fernet密钥，必须是base64编码的32字节（仅在频繁重新创建容器或使用多节点设置，从而使先前的验证无效的情况下才需要） |
| `CERTFILE=`        | 指向证书文件以为ldap auth守护程序启用SSL上的HTTP（HTTPS）    |
| `KEYFILE=`         | 将此指向私钥文件，与CERTFILE中引用的证书文件匹配             |

### 卷映射（`-v`）

| volume | 说明 |
| ------ | ---- |
| 无     | 无   |




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

- 该容器本身没有任何设置，它依赖于传入请求的http标头中传递的相关信息。确保您的网络服务器使用正确的配置进行设置。
- 这是一个示例配置：[nginx-ldap-auth.conf](https://github.com/nginxinc/nginx-ldap-auth/blob/master/nginx-ldap-auth.conf)
- 与上游项目不同，此镜像使用容器创建期间（或用户定义）使用随机生成的密钥，通过Fernet对cookie信息进行编码。
- 不同于上游项目，此映像在 `/ldaplogin`（以及 `/login`）提供登录页面，以防止与也可能使用 `/login` 进行内部身份验证的反向代理应用程序发生冲突。

------

## 支持

- 进入容器：
  - `docker exec -it ldap-auth /bin/bash`
- 查看容器日志：
  - `docker logs -f ldap-auth`
- 查看容器版本号：
  - `docker inspect -f '{% raw %}{{% endraw %}{ index .Config.Labels "build_version" }}' ldap-auth`
- 查看镜像版本号：
  - `docker inspect -f '{% raw %}{{% endraw %}{ index .Config.Labels "build_version" }}' ghcr.io/linuxserver/ldap-auth`

------

## 翻译之外

暂未试用