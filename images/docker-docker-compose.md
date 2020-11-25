# linuxserver/docker-compose

> [!TIP]
>
> 前半部分是翻译官方的文档，最后一部分是我的简单试用（个别软件会深度试用），如果对Docker已经有一定的了解了，可以直接跳转到最后面 [翻译之外](#翻译之外) 这部分来查看。

Docker-compose → https://github.com/docker/compose

GitHub → https://github.com/linuxserver/docker-docker-compose

Docker Hub → https://hub.docker.com/r/linuxserver/docker-compose

[Docker-compose](https://github.com/docker/compose) 是一个用来定义和运行多个Docker容器的工具。通过Compose，您可以使用Compose文件配置应用程序的服务。然后，使用单个命令，从您的配置中创建并启动所有服务。

------

## 支持的系统架构

得益于docker的跨平台属性，我们的镜像也支持多架构（如，x86-64、arm64、armhf）。

直接拉取 `ghcr.io/linuxserver/docker-compose` 应该就可以自动获取适合你系统架构的版本，当然你也可以通过 tag 获取特定的版本。

| 架构   | Tag            |
| ------ | -------------- |
| x86-64 | amd64-latest   |
| arm64  | arm64v8-latest |
| armhf  | arm32v7-latest |

### 版本标签

| Tag    | 说明             |
| ------ | ---------------- |
| latest | 基于Ubuntu的镜像 |
| alpine | 基于Alpine的镜像 |




------

## 拉取镜像

```shell
docker pull ghcr.io/linuxserver/docker-compose
```

------

## 使用方法

### docker cli

```shell
docker run --rm \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v "$PWD:$PWD" \
  -w="$PWD" \
  linuxserver/docker-compose \
  up
```

其中最后一行可以使用任何 docker-compose 的命令和参数，都将会直接传递给容器里的 docker-compose

### 推荐使用的方法

我们提供了一个非常方便的脚本，该脚本允许docker-compose容器像本地安装一样运行：

```shell
sudo curl -L --fail https://raw.githubusercontent.com/linuxserver/docker-docker-compose/master/run.sh -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

执行完上述两条命令后（事先已安装好docker），可以输入如 `docker-compose up -d` 这样的命令，docker-compose容器将在后台运行。

### 二进制版本

我们还在GitHub上提供了三种系统架构的 docker-compose 和 docker-cli 的二进制版本。你可以下载对应的版本，放到系统的 /usr/local/bin/ 文件夹下。

------

## 支持

- 查看镜像版本号：
  - `docker inspect -f '{% raw %}{{% endraw %}{ index .Config.Labels "build_version" }}' ghcr.io/linuxserver/docker-compose`

### 更新

#### 通过 Docker Cli 更新

- 拉取新镜像：`docker pull linuxserver/docker-compose`
- 删除旧的镜像：`docker image prune`

#### 本地构建

如果要出于开发目的或仅自定义逻辑而对这些映像进行本地修改：

```shell
git clone https://github.com/linuxserver/docker-docker-compose.git
cd docker-docker-compose
docker build \
  --no-cache \
  --pull \
  -t linuxserver/docker-compose:latest .
```

ARM架构可以使用 `multiarch/qemu-user-static`

```shell
docker run --rm --privileged multiarch/qemu-user-static:register --reset
```

完成后，可以通过 `-f Dockerfile.aarch64` 使用指定的dockerfile

------

## 翻译之外

在之前的指引中，已经很详细的介绍过了 → [Docker Compose](general/docker-compose.md)