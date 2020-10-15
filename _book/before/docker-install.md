# 安装Docker

## 使用脚本安装Docker

### Linux安装

Docker官方为了简化安装流程，提供了一套安装脚本，可以直接在终端中执行下面这条命令

```shell
curl -sSL https://get.docker.com/ | sh
```

如果在国内直接这么用可能会被感人的网速感动到，可以在后面加上阿里云的镜像。

```shell
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
```

当然，也可以用DaoCloud提供的安装脚本

```shell
curl -sSL https://get.daocloud.io/docker | sh
```

阿里云的官方也提供了一个安装脚本

```shell
curl -sSL http://acs-public-mirror.oss-cn-hangzhou.aliyuncs.com/docker-engine/internet | sh
```

安装过程中只需要静静地等待即可……

### Windows & macOS安装

对于Windows和macOS来说，只要去docker的官网下载对应系统的版本就好。 → [Docker Get Started](https://www.docker.com/get-started)

但我强烈不建议在Windows上安装docker，总会遇到各种各样奇奇怪怪的问题。我曾在两台不同的Windows电脑上安装过Docker，最后都毅然决然地卸载了。如果在Windows平台，可以尝试使用Windows的Linux子系统功能来体验Docker。 →  [适用于 Linux 的 Windows 子系统文档](https://docs.microsoft.com/zh-cn/windows/wsl/)

## 配置镜像加速

安装完成后，我是都习惯于修改下镜像源，不然有可能遇到网络问题。

可以使用DaoCloud提供的脚本来添加镜像源，可以把最后的加速地址换成自己的

```shell
curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://f1361db2.m.daocloud.io
```

也可以参照阿里云提供的方式手动设置（[阿里云容器镜像服务](https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors)）

```shell
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://3swlt11f.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

其实很简单，docker会读取在/etc/docker目录下的daemon.json文件作为配置。默认是没有这个文件的，新建一个就好，之后再用docker pull镜像的时候，就会从镜像源去拉取，提升了速度与稳定性。

到此为止，docker的安装就完成了。