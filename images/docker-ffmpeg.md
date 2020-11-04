# linuxserver/ffmpeg

> [!TIP]
>
> 前半部分是翻译官方的文档，最后一部分是我的简单试用（个别软件会深度试用），如果对Docker已经有一定的了解了，可以直接跳转到最后面 [翻译之外](#翻译之外) 这部分来查看。

FFmpeg → https://ffmpeg.org/

GitHub → https://github.com/linuxserver/docker-ffmpeg/releases

Docker Hub → https://hub.docker.com/r/linuxserver/ffmpeg

[FFmpeg](https://ffmpeg.org/) 是一个完善的跨平台的录制、转码、串流视频和音频的解决方案。

------

## 支持的系统架构

得益于docker的跨平台属性，我们的镜像也支持多架构（如，x86-64、arm64、armhf）。

直接拉取 `linuxserver/ffmpeg` 应该就可以自动获取适合你系统架构的版本，当然你也可以通过 tag 获取特定的版本。

| 架构   | Tag            |
| ------ | -------------- |
| x86-64 | amd64-latest   |
| arm64  | arm64v8-latest |
| armhf  | arm32v7-latest |

------

## 拉取镜像

```shell
docker pull linuxserver/ffmpeg
```

------

## 使用方法

以下是一些简单的示例。

与我们其他大部分的惊险个不同，该镜像仅通过输入FFmpeg命令在命令行中短暂地运行。为了能很好地使用该镜像，你需要了解Docker的基本使用方法并了解FFmpeg命令。在下面的示例中，我们将通过CLI来映射 /config 作为我们的工作目录并假设 input.mkv 文件在工作目录内。

我们应该使用输入文件的 用户/组 来运行FFmpeg，从而避免权限问题。

该镜像在 x86 平台下，支持硬件加速，具体可以参考下面的示例中的变量。

### 基本使用

```shell
docker run --rm -it \
  -v $(pwd):/config \
  linuxserver/ffmpeg \
  -i /config/input.mkv \
  -c:v libx264 \
  -b:v 4M \
  -vf scale=1280:720 \
  -c:a copy \
  /config/output.mkv
```

### 硬件加速（VAAPI）

```shell
docker run --rm -it \
  --device=/dev/dri:/dev/dri \
  -v $(pwd):/config \
  linuxserver/ffmpeg \
  -vaapi_device /dev/dri/renderD128 \
  -i /config/input.mkv \
  -c:v h264_vaapi \
  -b:v 4M \
  -vf 'format=nv12|vaapi,hwupload,scale_vaapi=w=1280:h=720' \
  -c:a copy \
  /config/output.mkv
```

### Nvidia硬件加速

```shell
docker run --rm -it \
  --runtime=nvidia \
  -v $(pwd):/config \
  linuxserver/ffmpeg \
  -hwaccel nvdec \
  -i /config/input.mkv \
  -c:v h264_nvenc \
  -b:v 4M \
  -vf scale=1280:720 \
  -c:a copy \
  /config/output.mkv
```

### 本地构建

如果你希望本地自定义这些镜像或自定义逻辑

```shell
git clone https://github.com/linuxserver/docker-ffmpeg.git
cd docker-ffmpeg
docker build \
  --no-cache \
  --pull \
  -t linuxserver/ffmpeg:latest .
```

可以使用 `multiarch/qemu-user-static` 在 x86_64 平台上构建 ARM 镜像

```shell
docker run --rm --privileged multiarch/qemu-user-static:register --reset
```

完成后，即可使用 `-f Dockerfile.aarch64` 来构建镜像。

------

## 翻译之外

FFmpeg是比较有名的媒体工具了，但自己对其了解的比较少，暂时就不进行深入的试用了。
