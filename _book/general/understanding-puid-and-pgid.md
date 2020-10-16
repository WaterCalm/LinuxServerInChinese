# 什么是PUID和PGID

> [!NOTE]
>
> 虽然Docker目前已经支持了 `--user` 参数。但我们的镜像目前还不兼容，所以我们还是建议继续使用PUID和PGID。

## 为什么要用这个？

因为Docker需要系统中管理网络、文件系统、进程的权限，所以通常是以 root 的身份在运行。这意味着，容器内的用户默认也是以 root 用户运行。这种高权限的访问，并不是日常使用中所推荐的，除非你对linux运维有着深入的理解。

另一个问题就是对容器映射出来的文件的管理。如果进程是以 root 身份运行，那么它所创建的文件的所有者都是 root 用户，这有可能会导致你没有权限去修改这些文件（如果你是以非root身份登陆服务器。）

而是用 PUID 和 PGID 允许我们将容器内的用户权限映射给宿主机上的用户。我们所有的容器都可以使用这种方法来映射用户权限。

------

## 使用变量

当使用我们的镜像创建容器时，确保你是用了 `-e PUID` 和 `-e PGID`：

```shell
docker create --name=beets -e PUID=1000 -e PGID=1000 linuxserver/beets
```

如果使用 `docker-compose` ，把他们添加在 `environment:` 下面：

```yaml
environment:
  - PUID=1000
  - PGID=1000
```

你很可能希望使用自己的 `id`，可以通过下面的命令来查询你当前用户的id。其中 `uid` 和 `gid` 分别对应 `PUID` 和 `PGID`：

```shell
id $user
```

