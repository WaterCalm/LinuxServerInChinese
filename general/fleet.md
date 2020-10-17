# Fleet

Fleet是一个基于web的镜像管理工具，适用于需要发布、管理大量Docker镜像的组织（个人）。

------

### Fleet如何工作

Fleet将Docker镜像的快照存储在数据库中，该快照由仓库所有者和用户镜像的元数据组成。它将会在设定的时间间隔内与Docker Hub同步，以更新其存储的数据。

然后，它将快照数据以列表的形式显示在状态页面上，其中包含仓库和镜像的链接。管理者还可以为每个镜像设置`稳定`和`不稳定`两个状态。这将帮助用户快速了解每个镜像的当前的状态。

### 为什么是快照？

简单说，Docker Hub提供的API非常慢，所以代理来自Docker Hub的数据并不是好的解决方案。通过将数据缓存在自己的数据库中，Fleet能更有效地展示每个镜像的状态。除此之外，它还可以在一条响应中提供如镜像版本等信息，而不用用户进行多次请求。

以下是通过Docker Hub和Fleet获取所有LinuxServer镜像的名称、拉取、版本信息的比较：

| API                    | 时间（毫秒） |
| ---------------------- | ------------ |
| Docker Hub（多次请求） | 52000毫秒    |
| Fleet                  | 50毫秒       |

------

## 属性

Fleet可以在应用的主页上显示镜像的状态（下列）等相关信息。

### 隐藏

如果镜像被隐藏，它将不会显示在列表中，也不会在API接口中返回任何信息，当然也不会包括统计该镜像的拉取次数。

### 不稳定

如果维护者发现镜像的某些问题，可以将其标记为不稳定状态。如果最新的版本（或其他版本）引起了下游损坏，那么也将标记给镜像。如果上游依赖或应用程序导致惊险个损坏，也将被标记出来。

### 弃用

如果镜像的维护者或者上游应用程序不在提供支持，或者镜像已经到达使用寿命（或已被其他镜像取代），则将惊险个标记为已弃用，以用来让用户知道这一消息，并应该停止使用它。

------

## API

Fleet提供了一个API接口，可用于获取镜像列表以及相关镜像的信息。

### 获取所有仓库和镜像

```
https://fleet.linuxserver.io/api/v1/images
```

返回所有同步的镜像。

响应：

```json
{
    "status": "OK",
    "data" {
        "totalPullCount": 1862494227,
        "repositories": {
            "lsiobase": [
                {
                    "name": "alpine",
                    "pullCount": 4275970,
                    "version": "3.6",
                    "stable": true
                },
                {
                    "name": "alpine.arm64",
                    "pullCount": 66234,
                    "version": "edge",
                    "stable": true
                },
                ...
            ],
            "linuxserver": [
                {
                    "name": "airsonic",
                    "pullCount": 4608329,
                    "version": "v10.2.1",
                    "stable": true
                },
                {
                    "name": "apache",
                    "pullCount": 3011699,
                    "version": "latest",
                    "stable": true
                },
                ...
            ]
            ...
        }
    }
}
```

> [!NOTE]
>
> 任何未与Docker Hub同步的仓库都不会作为API的一部分返回，这也包括被隐藏的镜像。

------

## 部署Fleet

> [!NOTE]
>
> Fleet是一个Java应用，至少需要JRE 11。

[从GitHub获取Fleet的最新版本。](https://github.com/linuxserver/fleet/releases)

### SQL

Fleet将数据存储在MariaDB上。你可以使用下面的命令创建给Fleet使用的数据库：

```sql
CREATE SCHEMA `fleet`;
CREATE USER 'fleet_user' IDENTIFIED BY 'supersecretpassword';
GRANT ALL ON `fleet`.* TO 'fleet_user';
```

数据库的账号密码也是Fleet配置信息的一部分。

### 配置文件

Fleet在运行时将通过加载 fleet.properties 文件来获取所有配置。它可以位于任何位置，并通过Runtime参数加载：

```
# Runtime
fleet.app.port=8080

# Database Connectivity
fleet.database.driver=org.mariadb.jdbc.Driver
fleet.database.url=jdbc:mariadb://<IP_OR_URL>:3306/fleet
fleet.database.username=<fleet_sql_user>
fleet.database.password=<fleet_sql_password>

# Password security
fleet.admin.secret=<a_random_string>
```

可以通过文件、JVM参数或系统环境变量来加载所有配置。Fleet将首先查找配置文件，然后是JVM参数最后才是系统环境变量。在加载时，它只加载第一个值，这在方需要覆盖特定属性时很有用。

> [!NOTE]
>
> 由于BASH环境的限制，如果将配置放在系统环境变量中，请将配置的属性名中的 `.` 替换为 `_` ，并且变量中不能包含该字符。如，`fleet.app.port=8080` 应该改为 `export fleet_app_port=8080` 。
>
> | 属性名                    | 说明                                                         |
> | ------------------------- | ------------------------------------------------------------ |
> | `fleet.app.port`          | 程序将在这个端口下运行。                                     |
> | `fleet.admin.secret`      | 一个字符串，作为密码加密的一部分。这个密钥将进一步增强哈希密码的随机性。**一旦创建，请勿修改！**因为它也同步在密码验证时使用，如果Fleet重启后此属性被删除或修改，所有的密码验证将失败，因为密码都是用之前的密钥进行的加密。 |
> | `fleet.database.driver`   | 连接Fleet的数据库的驱动，应该是 `org.mariadb.jdbc.Driver`    |
> | `fleet.database.url`      | 连接数据库的 JDBC 链接                                       |
> | `fleet.database.username` | 可以管理Fleet数据库的用户名，这个用户需要有Fleet数据库的所有权限，当然也可以有其他数据库的权限。 |
> | `fleet.database.password` | 数据库的用户的密码                                           |

### Runtime参数

除基本配置文件外，Fleet还通过 `-D` 来支持一些Runtime参数。这些可以用于Fleet以特定的方式运行。

> [!NOTE]
>
> 这些参数与上面的不同，只能通过JVM参数（`-D`）使用。

| Runtime参数                  | 说明                                                         |
| ---------------------------- | ------------------------------------------------------------ |
| `fleet.config.base`          | 配置文件的绝对路径                                           |
| `fleet.show.passwords`       | 在日志中显示明文密码。**不推荐**                             |
| `fleet.nuke.database`        | **【注意】**将完全擦出并重建Fleet数据库。如果Fleet数据库与Docker Hub相差太多或者镜像显示错误，可以使用这个参数来解决。 |
| `fleet.skip.sync.on.startup` | 默认情况下，Fleet首次运行时将执行同步。如果设置了次参数，第一次运行时将跳过同步，在设置的时间间隔后进行同步。 |

### 默认用户

首次启动Fleet时，它将创建一个默认用户，可以用这个用户登录进行管理并同步仓库和镜像。

默认用户和密码为：

**用户名：**admin

**密码：**admin

> [!WARNING]
>
> 你应该尽快修改该用户的默认密码！可以通过 `Admin -> User` 菜单去修改。