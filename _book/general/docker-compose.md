# Docker Compose

## 介绍

Compose是用于定义和运行多个docker应用程序的工具，你可以使用yaml文件来配置应用程序的服务，只用一条命令便可以启动配置中的所有服务。

------

## 安装

### 方式一（推荐）：

你可以通过运行我们的脚本基于我们的docker-compose镜像来安装docker-compose。只要下面这两条简单的命令，你就可以在你的系统上使用docker-compose：

```shell
sudo curl -L --fail https://raw.githubusercontent.com/linuxserver/docker-docker-compose/master/run.sh -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

可以用下面的命令来更新docker-compose：

```shell
docker pull linuxserver/docker-compose:"${DOCKER_COMPOSE_IMAGE_TAG:-latest}"
docker image prune -f
```

如果要使用基于微型系统alpine制作的docker-compose镜像，你可以通过修改 `.profile` 文件，在其中设置一条环境变量： `DOCKER_COMPOSE_IMAGE_TAG=alpine` 。当然，你也可以把变量设置为 `version-1.27.4` 或 `version-alpine-1.27.4` 来获取特定版本的docker-compose。



### 方式二：

我们也在 [这里](https://github.com/linuxserver/docker-docker-compose/releases) 发布了docker-compose的二进制版本。其中有两个版本：一个是基于glibc系统的（如Ubuntu、Debian）；一个是基于musl系统的（如Alpine，被标记为 `alpine` 标签）。每一个版本都包含了 amd64、armhf、arm64的二进制文件。你可以通过下面这条命令把二进制文件下载到你的系统里并安装docker-compose：

```shell
sudo curl -L --fail https://github.com/linuxserver/docker-docker-compose/releases/download/1.27.4-ls17/docker-compose-amd64 -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

------

## 启动单服务的方式

下面是用docker-compose部署linuxserver制作的容器的样例：

```yaml
version: "2.1"
services:
  heimdall:
    image: linuxserver/heimdall
    container_name: heimdall
    volumes:
      - /home/user/appdata/heimdall:/config
    environment:
      - PUID: 1000
      - PGID: 1000
      - TZ=Europe/London
    ports:
      - 80:80
      - 443:443
    restart: unless-stopped
```

如果你将以上代码片段保存到名为 `docker-compose.yml` 的文件中，你可以在与文件相同的目录下使用 `docker-compose up -d`  来部署服务。其中 heimdall 镜像将被自动拉取，容器将被自动创建并启动。其中，`up` 意味着启动服务，`-d` 则意味着在后台运行。

如果你希望用其他目录或其他名称的yaml文件（如 `heimdall.yml` ），那么你可以在 `-f` 后指定文件的位置：`docker-compose -f /path/to/heimdall.yml up -d`

如果你希望停止服务，则使用 `docker-compose down` 或 `docker-compose -f /path/to/heimdall.yml down` 。所有通过该配置文件部署的容器将被停止并删除。

------

## 启动多服务的方式

你可以使用一个compose的yaml文件管理多个服务。

复制配置文件中 `services:` 下的内容到同一个文件中并使用 `docker-compose up/down` 命令，将可以启动/停止文件中的所有服务。

例：`docker-compose.yml`

```yaml
version: "2.1"
services:
  heimdall:
    image: linuxserver/heimdall
    container_name: heimdall
    volumes:
      - /home/user/appdata/heimdall:/config
    environment:
      - PUID: 1000
      - PGID: 1000
      - TZ=Europe/London
    ports:
      - 80:80
      - 443:443
    restart: unless-stopped
  nginx:
    image: linuxserver/nginx
    container_name: nginx
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
    volumes:
      - /home/user/appdata/nginx:/config
    ports:
      - 81:80
      - 444:443
    restart: unless-stopped
  mariadb:
    image: linuxserver/mariadb
    container_name: mariadb
    environment:
      - PUID=1000
      - PGID=1000
      - MYSQL_ROOT_PASSWORD=ROOT_ACCESS_PASSWORD
      - TZ=Europe/London
    volumes:
      - /home/user/appdata/mariadb:/config
    ports:
      - 3306:3306
    restart: unless-stopped
```

这样，你就可以一下启动 heimdall、nginx、mariadb 这三个服务。

当你执行 docker-compose up -d 时，将首先使用本地的镜像，若本地中没有，则会自动拉取相关的镜像。然后将创建并启动这三个容器。执行 docker-compose down 时，所有的三个服务将被停止并删除容器（数据将被保留，因数据都以映射到本地保存）。



------

## 更新升级

如果你希望更新镜像并用相同的参数重新创建容器，那么使用docker-compose便会异常轻松。首先，我们使用 `docker-compose pull` 来更新所有的镜像。然后，执行 `docker-compose up -d` 后，将自动用更新后的镜像重新创建容器。如果某个容器已经是最新版，那么它将不会被更新，将保持不变。

同样的，如果你修改了yaml文件中的内容，并执行 `docker-compose up -d` ，那么只会改变yaml文件中配置的容器，其他容器不变。

将部署容器的代码定义在服务器上，是“Devops”的核心宗旨。使用docker-compose，可以有效避免直接使用`docker run` 命令时忘记某个参数的尴尬情况。

------

## 小贴士

`docker-compose` 默认使用当前目录下的 `docker-compose.yml` 文件，若没有该文件则会报错。我们可以使用bash aliases来提高效率。

创建 `~/.bash_aliases` 文件并写入下面的内容：

```shell
alias dcp='docker-compose -f /opt/docker-compose.yml '
alias dcpull='docker-compose -f /opt/docker-compose.yml pull'
alias dclogs='docker-compose -f /opt/docker-compose.yml logs -tf --tail="50" '
alias dtail='docker logs -tf --tail="50" "$@"'
```

再将下面的代码加入 `~/.bashrc` 中以用来启用上面的文件：

```shell
if [ -f ~/.bash_aliases ]; then
    . ~/.bash_aliases
fi
```

配置完成后，注销再登录后。你就可以使用 `dcpull` 或 `dcp up -d` 来管理你的容器了。