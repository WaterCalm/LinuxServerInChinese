# 关于Docker的几点常识

## Docker几个概念的简单介绍

为了方便小白，简单介绍下后面文章中会提到的几个概念：

| 中文名称 | 英文        | 参数        | 介绍                                                         |
| -------- | ----------- | ----------- | ------------------------------------------------------------ |
| 镜像     | image       |             | 可以理解为别人制作好的安装包                                 |
| 容器     | container   |             | 通过 `docker run` 或 `docker create` 命令即可部署镜像，随之成为容器并运行在你的电脑上 |
| 卷       | volume      | `-v`        | 在部署镜像的时候通过 `-v` 来设置，即可把宿主机上的路径映射到容器内。也可以通过 `docker volume create` 来创建匿名卷存储相关的数据。 |
| 环境变量 | environment | `-e`        | 即容器内的环境变量，在部署的时候通过 `-e` 来设置，容器内可以通过 `env` 查看。 |
| 端口     | port        | `-p`        | 可以将容器内的某个端口映射到宿主机上                         |
| 网络     | network     | `--network` | 可以将多个容器设置在同一个网络下，方便各个容器之间的网络通信，这样也不必要将每个端口都开放到宿主机上。如果要给容器绑定ip则必须要先创建网络才行。 |


------

## 创建虚拟网络

在实际应用中我们会遇到不同容器之间需要进行网络通信。比如搭建nextcloud，可能需要mariadb、nextcloud、nginx三个容器一起运行，这时就需要将这三个容器分别绑定一个固定ip，不然重启电脑或者重启docker的时候因容器的启动顺序不同，会造成他们在容器内的ip改变，造成服务报错。

如果要给容器绑定固定ip，需要先创建网络（[官方文档](https://docs.docker.com/engine/reference/commandline/network_create/)）：

```shell
docker network create --driver=bridge --subnet=10.0.0.0/24 myNetwork
```

其中 `--subnet` 是`ip/掩码位` 格式，可以指定创建的网络的IP段，具体可以通过 [IP地址在线计算器](https://tool.520101.com/wangluo/ipjisuan/) 了解。当然，也可以不指定这个参数，那么docker将默认分配一个IP段。

创建好虚拟网络之后，我们在部署镜像的时候就可以通过 `--network` 和 `--ip` 来给容器指定网络和IP。

------

当然，如果你已经安装了portainer的话。也可以直接通过web ui来创建网络

![image-20201016141608627](https://pic.watercalmx.com/pic/image-20201016141608627.png)