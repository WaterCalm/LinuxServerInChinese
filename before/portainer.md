# Portainer - 可视化管理Docker

![](https://pic.watercalmx.com/pic/01_homescreen-ce.png)

Portainer是一个优秀的Docker、Swarm、Kubernetes等容器服务的管理工具，拥有简单明了的UI界面。这里我们只要知道这东西可以可视化管理Docker就好，功能很强大，但对于我们小白来说最直观的感受就是告别了黑白的命令行。

## 拉取镜像

拉取镜像前，我们可以先去 Docker Hub 搜索一下，一般来说在这里都能找到对应镜像的文档。https://hub.docker.com/

> [!WARNING]
>
> portainer/portainer 镜像目前已不被推荐使用，应该使用 portainer/portainer-ce 镜像

在命令行下拉取镜像，如果非root用户或者提示权限问题，记得在命令前加上 sudo

```shell
docker pull portainer/portainer-ce
```

因为portainer需要一个卷（volume）来保存配置和相关数据，我们可以用docker创建一个匿名卷，也可以选择挂载一个本地的目录。

> [!NOTE]
>
> 官方的提供的部署命令中除了开放9000端口外，还开放了8000端口。但我们只通过浏览器访问portainer，所以只开放9000端口即可。

```shell
#创建匿名卷
docker volume create portainer_data
#部署容器
docker run -d -p 9000:9000 --name=portainer -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce  
```

如果想挂载到本地目录只需要把 portainer_data 修改为本地的绝对路径即可。

docker的部署命令中，涉及到端口和卷的映射时，都是这种格式  宿主机:容器内 。

> [!TIP]
>
> 建议大家把docker容器的部署命令写在一个 shell 文件中，并赋予文件执行权限。这样如果日后需要更新容器或者重新部署的时候，只要运行这个shell文件即可。

```shell
#创建文件
nano portainer.sh
#把命令写入文件，可以用这种格式方便阅读，记得每行结尾要有“ \”，不然会报错
docker run -d \
  -p 9000:9000 \
  --name=portainer \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /docker/portainer:/data \
  portainer/portainer-ce

#写入完成后保存，nano编辑器就是 ctrl+o写入，ctrl+x退出。vi编辑器则是 按esc后输入 :wq 
#赋予脚本执行权限
chmod +x portainer.sh
#执行脚本部署容器
./portainer.sh
```

部署成功后，会返回容器的id。

然后我们在浏览器中输入 http://ipaddress:9000 即可访问portainer的WEB UI界面

之后，我们按照界面的提示创建管理员账号、选择本地容器就可以进入主界面了。

![](https://pic.watercalmx.com/pic/da42642a58ba578e328375288bfa6e03ca4ec828.png)

![](https://pic.watercalmx.com/pic/33acc03c5a71f9d31476351500484ce4b6a2a849.png)

![](https://pic.watercalmx.com/pic/0bc9f44e4f629bc42c74629b9360ab88358d0ccb.png)