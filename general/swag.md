# 部署SWAG

这篇指南将说明LinuxServer SWAG镜像的功能并带你入门。我们将解释一些基本概念和它的局限性，然后我们也将提提供高一些通用的案例。如果你有问题，可以在我们的论坛或Discord一起讨论。

> [!TIP]
>
> 此篇文章较长，可以点击右上角的悬浮按钮查看目录以跳转到自己感兴趣的部分。

------

## 介绍

### 什么是 SSL证书？

SSL证书可以让用户使用加密数据进行网络通信。第三方信任证书也会告诉用户，他所连接的服务器是真实的服务提供商而不是其他冒名顶替的服务商。当因托管网站或反向代理而运行Web服务器时，我们也需要使用第三方受信任的SSL证书并对其设置，以便能和用户的浏览器进行安全通信。当你是用受信任的证书连接网站时，大多数浏览器会在地址栏旁边显示一个小锁头来表示。如果是没有受信任的证书（如自签名证书），则大多数浏览器将显示警告页面，或者由于无法通过受信任证书确认网站身份而阻止你访问该网站。

### 什么是Let's Encrypt？

以前，想要获取受信任的SSL证书通常需要联系供应商，向他们提供相关信息以及域名所有权证明文件并付费。现在，通过[Let's Encrypt](https://letsencrypt.org/)，大家可以通过自动的方式获取免费的证书。

LinuxServer.io发布和维护了[SWAG docker镜像](https://hub.docker.com/r/linuxserver/swag)，通过这个镜像可以非常容易地获取以及更新SSL证书并运行Web服务器。其实，它是一个内置了php7、fail2ban（防御入侵）和Let's Encrypt的Nginx web服务器。他就是一个缺少了MySQL的LEMP(Linux/Nginx/MySQL/PHP)环境，所以也可以搭配我们的 [MariaDB docker镜像](https://hub.docker.com/r/linuxserver/mariadb)一起使用。

------

## 创建 SWAG 容器

在这个镜像中，大多数设置都可以通过 docker run/create 或 compose yaml 设置，之后便可以架设起一个具有ssl证书的web服务器。以下是所有可用的设置（包括可选）。针对于不同的情况，可以删除不必要的参数。

### docker cli

```shell
docker create \
  --name=swag \
  --cap-add=NET_ADMIN \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Europe/London \
  -e URL=yourdomain.url \
  -e SUBDOMAINS=www, \
  -e VALIDATION=http \
  -e DNSPLUGIN=cloudflare `#可选` \
  -e DUCKDNSTOKEN=<token> `#可选` \
  -e EMAIL=<e-mail> `#可选` \
  -e DHLEVEL=2048 `#可选` \
  -e ONLY_SUBDOMAINS=false `#可选` \
  -e EXTRA_DOMAINS=<extradomains> `#可选` \
  -e STAGING=false `#可选` \
  -p 443:443 \
  -p 80:80 `#可选` \
  -v </path/to/appdata/config>:/config \
  --restart unless-stopped \
  linuxserver/swag
```

### docker-compose

与docker-compose v2兼容

```yaml
---
version: "2"
services:
  swag:
    image: linuxserver/swag
    container_name: swag
    cap_add:
      - NET_ADMIN
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
      - URL=yourdomain.url
      - SUBDOMAINS=www,
      - VALIDATION=http
      - DNSPLUGIN=cloudflare #optional
      - DUCKDNSTOKEN=<token> #optional
      - EMAIL=<e-mail> #optional
      - DHLEVEL=2048 #optional
      - ONLY_SUBDOMAINS=false #optional
      - EXTRA_DOMAINS=<extradomains> #optional
      - STAGING=false #optional
    volumes:
      - </path/to/appdata/config>:/config
    ports:
      - 443:443
      - 80:80 #optional
    restart: unless-stopped
```

### 验证方式

通过Let's Encrypt获取SSL证书，需要先证明这个域名属于你。这个镜像提供了三种不同的方式，来验证域名：

- **http**
  - Let's Encrypt（acme）将访问你的80端口来验证
  - 需要拥有域名或动态dns地址
- **dns**
  - Let's Encrypt（acme）将通过dns解析来验证
  - 在 `/config/dns-conf/` 文件夹下的 `ini` 文件设置dns解析服务商的API参数
  - 支持通配符证书
  - 需要拥有自己的域名（需付费购买）
- **duckdns**
  - Let's Encrypt（acme）通过DuckDNS来验证
  - 支持通配符证书（仅支持子域名的子域名）
  - 不需要有自己的域名（免费）

关于域名的验证将在第一次启动容器的时候进行，在成功生成SSL证书之前，Nginx不会运行。

证书的有效期为90天。容器每天都会检查证书的有效期，如果有效期不足30天，将执行自动续订。如果有效期依旧不足30天，请查看 `/config/log/letsencrypt` 下的日志，以查找自动续订失败的原因。

> [!NOTE]
>
> 国内的实际情况大家也清楚，普通家庭的宽带的80和443端口已经被运营商关闭了，所以下面http的验证方式可能并不适用于大部分家庭。建议选择其他两种方式，个人比较推荐dns验证方式，而且镜像中的也提供了阿里云dns的API接口。

### 端口

如果要通过https访问，需要映射443端口。但是，您不一定需要在宿主机上开放443端口，最主要的是需要在路由器上映射443端口，即以某种方式将路由器上的443端口映射到容器内的443端口。

例如，可以将路由器上的443端口转发到宿主机上的444端口，然后在创建容器时将容器内的443端口映射到宿主机上的444端口。

仅http验证需要使用80端口，与上面一样，可以将路由器上的80端口转发到宿主机上的81端口，然后映射到容器里的80端口。

> [!NOTE]
>
> 和上面一样，80和443端口并不适用于国内的普通家庭，可以修改为其他任意端口。

### Docker网络

SWAG容器可以很好地运行于桥接网络上，但docker中的默认桥接网络并不允许容器之间通过主机名进行相互连接。所以建议首先创建一个自定义的桥接网络，并时该容器连接到该网络中。

如果你是用的是docker-compose，并且多个服务都配置在同一个yaml文件上，则无需创建自定义网络。因为docker-compose在默认情况下会自动创建自定义桥接网络，并将配置文件中的每个容器都加入进去（如果没有特殊配置的话）。

下面的示例，我们将使用名为 `lsio` 的网络，我们可以通过 `docker network create lsio` 创建他。然后只要使用 `--net=lsio` 创建的容器，它们互相之间就是能ping通的。

> 注意：主机名不区分大小写，但是容器名称区分大小写。为了使容器名称在nginx用作主机名，应该都统一为小写，不然nginx在尝试解析之前也会将它们转换为全部小写的形式。

------

## 样例

### 通过 http 验证的的方式创建容器

架设我们的域名是 `linuxserver-test.com` ，我们希望证书包括 `www.linuxserver-test.com` 和 `ombi.linuxserver-test.com` 这两个域名。首先，转发路由器上的80和443端口到服务器上；然后，设置dns解析，为`www`和`ombi`添加A记录或CNAMES，使他们都指向你的服务器或路由器。

如果使用 docker cli ，先使用 `docker network create lsio` 创建自定义网络，然后创建容器：

```shell
docker create \
  --name=swag \
  --cap-add=NET_ADMIN \
  --net=lsio \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Europe/London \
  -e URL=linuxserver-test.com \
  -e SUBDOMAINS=www,ombi \
  -e VALIDATION=http \
  -p 443:443 \
  -p 80:80 \
  -v /home/aptalca/appdata/swag:/config \
  --restart unless-stopped \
  linuxserver/swag
```

创建成功后，使用 `docker start swag` 来运行容器。

如果使用docker-compose，可以使用下面的yml：

```yaml
---
version: "2"
services:
  swag:
    image: linuxserver/swag
    container_name: swag
    cap_add:
      - NET_ADMIN
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
      - URL=linuxserver-test.com
      - SUBDOMAINS=www,ombi
      - VALIDATION=http
    volumes:
      - /home/aptalca/appdata/swag:/config
    ports:
      - 443:443
      - 80:80
    restart: unless-stopped
```

我们可以使用 `docker-compose up -d` 

容器启动之后，使用 `docker logs swag -f` 来查看日志。当初始化结束并完成验证步骤之后，日志会输出 `Server ready`。

现在，我们可以打开 https://www.linuxserver-test.com 便能看到默认的网页。

### 通过dns验证获取通配符证书的方式创建容器

我们假设域名是 linuxserver-test.com ，并且希望证书可以涵盖 www.linuxserver-test.com 、ombi.linuxserver-test.com以及任何其他的子域名。我们依旧在路由器上将443端口转发到服务器上（80端口则不是必须的）

> [!NOTE]
>
> 其实443端口也并不是必须的，样例中之所以设置了443端口是因为https的默认端口就是443端口。

要使用dns验证，我们首先要确认我们使用的是镜像所支持的dns解析商。目前支持以下供应商的dns解析服务：aliyun、cloudflare、cloudxns、cpanel、digitalocean、dnsimple、dnsmadeeasy、domeneshop、gandi、google、inwx、linode、luadns、nsone、ovh、rfc2136、route53、transip。

一般来说，您的dns解析供应商就是您的域名供应商，并且也很容易更换域名的dns解析供应商。推荐使用Cloudflare，因为它既免费又可靠。

> [!NOTE]
>
> 针对于国内，cloudflare可能并不是最好的选择，个人还是更推荐阿里云，所以关于cloudflare的设置暂且略过。

现在，我们创建容器。

如果用docker cli，我们首先先使用 `docker network create lsio` 创建自定义桥接网络，然后再创建容器：

```shell
docker create \
  --name=swag \
  --cap-add=NET_ADMIN \
  --net=lsio \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Europe/London \
  -e URL=linuxserver-test.com \
  -e SUBDOMAINS=wildcard \
  -e VALIDATION=dns \
  -e DNSPLUGIN=cloudflare \
  -p 443:443 \
  -p 80:80 \
  -v /home/aptalca/appdata/swag:/config \
  --restart unless-stopped \
  linuxserver/swag
```

然后，我们通过 `docker start swag` 启动容器。

如果使用 docker-compose ，用如下的yml：

```yaml
---
version: "2"
services:
  swag:
    image: linuxserver/swag
    container_name: swag
    cap_add:
      - NET_ADMIN
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
      - URL=linuxserver-test.com
      - SUBDOMAINS=wildcard
      - VALIDATION=dns
      - DNSPLUGIN=cloudflare
    volumes:
      - /home/aptalca/appdata/swag:/config
    ports:
      - 443:443
      - 80:80
    restart: unless-stopped
```

之后我们使用 `docker-compose up -d` 启动容器。

启动容器之后，可以使用`docker logs swag -f` 来查看日志。在初始化完成后，我们会发现在验证过程中发生了一些错误，这是因为我们尚未配置cloudflare api的凭据。我们可以在 `/config/dns-conf` 里找到对应的文件，填写相关的凭据即可。

> [!NOTE]
>
> 官方的样例中以cloudflare为例，可能并不符合国内的使用情况。但其他的和这个操作类似，都是在上面的文件夹中找到对应的 `ini` 文件，阿里云的只要填写 `AccessKey ID` 和 `AccessKey Secret` 即可。如果感觉并不安全，可以使用阿里云提供的子账户功能，子账户中可以详细地配置其拥有的权限。

修改成功后，使用 `docker restart swag` 来重启容器，并再次查看日志。如果设置没错的话，看到 `Server ready` 后，web服务器就准备就绪了。可以通过 https://www.linuxserver-test.com 访问。

### 通过duckdns认证获取通配符证书的方式创建容器

首先，需要先申请一个 [DuckDNS](https://duckdns.org/) 的子域名，假设我们的子域名是 `linuxserver-test` ，因此我们的网址是 `linuxserver-test.duckdns.org` 。然后，我们需要确保子域名指向的是我们路由器的IP地址。也可以使用我们的 [DuckDNS docker 镜像](https://hub.docker.com/r/linuxserver/duckdns)来自动更新IP地址。别忘了，也要获取 DuckDNS 的token和在路由器的443端口转发（80端口可选）。

现在，我们来创建容器。

如果使用的是 docker cli ，使用 `docker network create lsio` 来创建自定义桥接网络，然后我们启动容器：

```shell
docker create \
  --name=swag \
  --cap-add=NET_ADMIN \
  --net=lsio \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Europe/London \
  -e URL=linuxserver-test.duckdns.org \
  -e SUBDOMAINS=wildcard \
  -e VALIDATION=duckdns \
  -e DUCKDNSTOKEN=97654867496t0877648659765854 \
  -p 443:443 \
  -p 80:80 \
  -v /home/aptalca/appdata/swag:/config \
  --restart unless-stopped \
  linuxserver/swag
```

然后使用 `docker start swag` 来启动容器。

使用 docker-compose ，可以使用以下的yml：

```yaml
---
version: "2"
services:
  swag:
    image: linuxserver/swag
    container_name: swag
    cap_add:
      - NET_ADMIN
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
      - URL=linuxserver-test.duckdns.org
      - SUBDOMAINS=wildcard
      - VALIDATION=duckdns
      - DUCKDNSTOKEN=97654867496t0877648659765854
    volumes:
      - /home/aptalca/appdata/swag:/config
    ports:
      - 443:443
      - 80:80
    restart: unless-stopped
```

然后我们使用 `docker-compose up -d` 来创建容器。

容器启动之后，通过 `docker logs swag -f` 来查看日志。在经过一些初始化和验证的步骤后，便会看到 `Server ready` 。

现在我们可以访问 https://www.linuxserver-test.duckdns.org 。

> **注意：**由于DuckDNS的限制，证书仅适用于子域名，但并不适用于主URL。因此，当我们访问https://linuxserver-test.duckdns.org，则会看到有关ssl证书无效的警告。但是通过www（或ombi或任何其他）等子域名访问，它便会正常工作。

------

## Web服务样例

### 简单的html网页web服务

当容器正常工作后，我们放入web文档并修改nginx配置文件，以设置我们的web服务器。

所有必须的文件都在容器内的 `/config` 文件夹下，按照当前的样例则是在 `/home/aptalca/appdata/swag` 文件夹下。

把所有的 html 文件放入 `/config/www` 文件夹下。

Nginx主站点的配置文件在 /config/nginx/default 。不要删除这个文件，容器重启时也将重新创建该文件，但可以根据自己的需要修改该文件。默认情况下，它正在侦听443端口，并且网站根目录位于 /config/www 下，如果你把 page1.html 放到这里，则可以用 https://linuxserver-test.com/page1.html 来访问。

想要启用侦听80端口，并使其自动重定向到443端口，请取消默认站点配置顶部的行，使其显示为：

```
# redirect all traffic to https
server {
    listen 80;
    listen [::]:80;
    server_name _;
    return 301 https://$host$request_uri;
}
```

对配置文件进行了任何修改后，只需使用 `docker restart swag` 重启容器即可冲洗加载nginx配置。

### Wordpress站点

Wordpress需要mysql数据库，为此我们可以使用 [linuxserver MariaDB docker镜像](https://hub.docker.com/r/linuxserver/mariadb)。

以下是docker compose的配置文件，用于设置两个容器。在例子中，我们使用 cloudflare dns 的方式进行 Let's Encrypt 的验证，你也可以使用其他的方法来设置：

```yaml
---
version: "2"
services:
  mariadb:
    image: linuxserver/mariadb
    container_name: mariadb
    environment:
      - PUID=1000
      - PGID=1000
      - MYSQL_ROOT_PASSWORD=mariadbpassword
      - TZ=Europe/London
      - MYSQL_DATABASE=WP_database
      - MYSQL_USER=WP_dbuser
      - MYSQL_PASSWORD=WP_dbpassword
    volumes:
      - /home/aptalca/appdata/mariadb:/config
    restart: unless-stopped
  swag:
    image: linuxserver/swag
    container_name: swag
    cap_add:
      - NET_ADMIN
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
      - URL=linuxserver-test.com
      - SUBDOMAINS=wildcard
      - VALIDATION=dns
      - DNSPLUGIN=cloudflare
    volumes:
      - /home/aptalca/appdata/swag:/config
    ports:
      - 443:443
      - 80:80
    depends_on:
      - mariadb
    restart: unless-stopped
```

使用docker cli的话，可以参考下面的命令（前提是已经创建了名为lsio的自定义网络）：

MariaDB：

```shell
docker create \
  --name=mariadb \
  --net=lsio \
  -e PUID=1000 \
  -e PGID=1000 \
  -e MYSQL_ROOT_PASSWORD=mariadbpassword \
  -e TZ=Europe/London \
  -e MYSQL_DATABASE=WP_database \
  -e MYSQL_USER=WP_dbuser \
  -e MYSQL_PASSWORD=WP_dbpassword \
  -v /home/aptalca/appdata/mariadb:/config \
  --restart unless-stopped \
  linuxserver/mariadb
```

SWAG：

```shell
docker create \
  --name=swag \
  --cap-add=NET_ADMIN \
  --net=lsio \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Europe/London \
  -e URL=linuxserver-test.com \
  -e SUBDOMAINS=wildcard \
  -e VALIDATION=dns \
  -e DNSPLUGIN=cloudflare \
  -p 443:443 \
  -p 80:80 \
  -v /home/aptalca/appdata/swag:/config \
  --restart unless-stopped \
  linuxserver/swag
```

SWAG容器成功启动后，并获取到ssl证书且web服务器成功启动后，我们就可以下载最新版本的Wordpress并解压到 www 文件夹了：

```shell
wget https://wordpress.org/latest.tar.gz
tar xvf latest.tar.gz -C /home/aptalca/appdata/swag/www/
rm latest.tar.gz
```

我们已经将所有wordpress文件保存到了容器内的 `/config/www/wordpress` 文件夹下，我们也同步修改SWAG默认站点的配置文件，将网站的根目录改为此路径。在 `/config/nginx/site-confs/default` 找到 `root /config/www;` 并修改为 `root /config/www/wordpress;` 并重新启动SWAG。

现在，我们可以访问 `https://linuxserver-test.com/wp-admin/install.php` 来配置 wordpress。在 Database Host 中填入 mariadb （因为两个容器都在同一个自定义桥接网络中，所以可以直接使用主机名来彼此访问），然后再输入mariadb的数据库名、用户名和密码即可（`WP_database`、`WP_dbuser` 、`WP_dbpassword`）。

完成所有的安装步骤之后，应该就可以通过 `https://linuxserver-test.com` 访问wordpress了。

如果你希望启用80端口的http请求并自动重定向到443端口上的https，请取消下面这些内容的注释：

```
# redirect all traffic to https
server {
    listen 80;
    listen [::]:80;
    server_name _;
    return 301 https://$host$request_uri;
}
```

------

## 反向代理

反向代理是一种代理服务器，他代理客户端从一个或多个服务器中检索资源，然后将这些资源再返回给客户端。看起来就像它自己本身就是web服务器一样。

![反向代理](https://pic.watercalmx.com/pic/reverseproxy.png)

在这种情况下，客户端浏览器可以通过443端口的https连接到我们的SWAG容器，然后SWAG容器去连接ombi容器，检索数据后将其通过https再返回给客户端浏览器。SWAG容器和ombi容器之间是本地连接，不需要加密，但SWAG容器与客户端浏览器之间的所有通信都将被加密。

### 预设的代理设置

我们的SWAG镜像已经预制了用于流行应用的服务的预设反向代理配置文件。[它们托管在GitHub上](https://github.com/linuxserver/reverse-proxy-confs)，把他们放入 `/config/nginx/proxy-confs` 文件夹下座位样例文件。但想要使用其中的某些文件，则需要重新命名配置文件，去掉 `.sample` 后缀并重新启动SWAG容器。该文件夹中任何名称与 `*.subdomain.conf` 或 `*.subfolder.conf` 匹配的配置文件都将在容器启动的时候加载到nginx中。

大多数的代理配置文件无需更改即可使用，但有些可能需要进行一些其他的更改。所有需要更改的内容都在文件顶部的注释中。配置文件中，都是使用容器名称来进行互相访问，所以请使容器的名称与每个容器的文档中的名称相同。

当然，以上的前提都是SWAG容器与其他容器都位于同一个自定义的桥接网络中，这样他们才能通过主机名互相访问。确保遵循前面关于Docker 网络部分的说明。

### 了解代理配置文件的结构

#### 子域名代理配置

下面这个是Heimdall作为子域名的代理服务器配置（即`https://heimdall.linuxserver-test.com`）：

```
# make sure that your dns has a cname set for heimdall

server {
    listen 443 ssl;
    listen [::]:443 ssl;

    server_name heimdall.*;

    include /config/nginx/ssl.conf;

    client_max_body_size 0;

    # enable for ldap auth, fill in ldap details in ldap.conf
    #include /config/nginx/ldap.conf;

    # enable for Authelia
    #include /config/nginx/authelia-server.conf;

    location / {
        # enable the next two lines for http auth
        #auth_basic "Restricted";
        #auth_basic_user_file /config/nginx/.htpasswd;

        # enable the next two lines for ldap auth
        #auth_request /auth;
        #error_page 401 =200 /ldaplogin;

        # enable for Authelia
        #include /config/nginx/authelia-location.conf;

        include /config/nginx/proxy.conf;
        resolver 127.0.0.11 valid=30s;
        set $upstream_app heimdall;
        set $upstream_port 443;
        set $upstream_proto https;
        proxy_pass $upstream_proto://$upstream_app:$upstream_port;

    }
}
```

让你剖析这个配置文件，看看每个指令或块的作用。

```
server {
}
```

这是服务器块，每当nginx从客户端收到请求时，它都会根据目标服务器的名称、端口和其他相关信息来确定应该处理哪个服务器块，而匹配的服务器块将确定nginx如何处理和响应这些请求。

        listen 443 ssl;
        listen [::]:443 ssl;
这意味着只有通过443端口的访问才会与该服务器块匹配。

```
    server_name heimdall.*;
```

只有通过与 `heimdall.*` 匹配的域名的访问才会与该服务器块匹配。

```
    include /config/nginx/ssl.conf;
```

该指定在此处包含了ssl.conf文件里的内容，该文件包含所有与ssl相关的设置（证书位置，使用的加密算法等）。

```
    client_max_body_size 0;
```

不限制客户端上传数据的大小，默认为1M

```
    # enable for ldap auth, fill in ldap details in ldap.conf
    #include /config/nginx/ldap.conf;
```

默认情况下已经注释掉，启用后将包含ldap.conf文件里的内容，这是LDAP身份认证的必要设置。

```
    # enable for Authelia
    #include /config/nginx/authelia-server.conf;
```

默认情况下已经注释掉，启用后将包含authelia-server.conf文件里的内容，这是Authelia集成的必要设置。

```
    location / {
    }
```

位置块用于配置子文件夹或路径。匹配服务器块后，nginx继续查找与之匹配的位置块。在我们的示例中，将匹配任何子文件夹或路径。

```
        # enable the next two lines for http auth
        #auth_basic "Restricted";
        #auth_basic_user_file /config/nginx/.htpasswd;
```

默认情况下已经被注释掉，启用后它将先使用.htpasswd认证用户身份后才允许访问。

```
        # enable the next two lines for ldap auth
        #auth_request /auth;
        #error_page 401 =200 /login;
```

默认情况下已经被注释掉，启用后将通过LDAP身份认证后才允许访问。

```
        # enable for Authelia
        #include /config/nginx/authelia-location.conf;
```

默认情况下已经被注释掉，启用后将通过Authelia身份验证后才允许访问。

```
        include /config/nginx/proxy.conf;
```

包含proxy.conf的内容，其中包含代理连接常见的各种指令和标头。

```
        resolver 127.0.0.11 valid=30s;
```

当容器名称用作后面的代理地址时，告诉nginx使用docker的dns解析ip地址。

```
        set $upstream_app heimdall;
        set $upstream_port 443;
        set $upstream_proto https;
        proxy_pass $upstream_proto://$upstream_app:$upstream_port;
```

这是棘手的一部分，通常我们只输入 `proxy_pass https://heimdall:443;` 指令就可以通过主机名连接到Heimdall容器。尽管它在大部分情况下都有效，但nginx有一个令人讨厌的习惯，就是在启动期间nginx会检查proxy_pass中的所有主机名，如果其中任何一个不能访问，它将拒绝启动。我们确实不希望使用已经停止的代理容器来阻止我们的Web服务器启动，因此我们使用了这个小技巧。

如果proxy_pass语句中包含变量而不是主机名，nginix将不会在启动的时候检查其是否可被访问。因此我们在这里设置了3个变量，一个值为 `heimdall` 的变量 `upstream_app` ；一个值为其端口`443`的变量 `upstream_port` ；一个值为 `https` 的变量 `upstream_proto` 。我们将这些变量用作 proxy_pass 指令中的地址。这样，如果heimdall容器由于任何原因关闭，nginx仍然可以启动。当使用的是变量而不是主机名时，还必须将上一行的解析器设置为docker dns。

如果被代理的容器与SWAG不在同一个自定义桥接网络中时（或者位于网络上的其他位置），可以把 `upstream_app` 变量修改为IP地址。如，`set $upstream_app 192.168.1.10;`

#### 子文件夹代理配置

下面这个是通过名为 mytinytodo 子文件夹的预设代理配置：

```
# works with https://github.com/breakall/mytinytodo-docker
# set the mtt_url to 'https://your.domain.com/todo/' in db/config.php

location /todo {
    return 301 $scheme://$host/todo/;
}
location ^~ /todo/ {

    # enable the next two lines for http auth
    #auth_basic "Restricted";
    #auth_basic_user_file /config/nginx/.htpasswd;

    # enable the next two lines for ldap auth, also customize and enable ldap.conf in the default conf
    #auth_request /auth;
    #error_page 401 =200 /ldaplogin;

    # enable for Authelia, also enable authelia-server.conf in the default site config
    #include /config/nginx/authelia-location.conf;

    include /config/nginx/proxy.conf;
    resolver 127.0.0.11 valid=30s;
    set $upstream_app mytinytodo;
    set $upstream_port 80;
    set $upstream_proto http;
    proxy_pass $upstream_proto://$upstream_app:$upstream_port/;
}
```

与子域名代理配置不同，这里没有服务器块。因为所有的子文件夹代理配置文件都会被包含到默认站点的配置中定义的根域名的主服务器块中。因此，在这我们仅为特定的子文件夹定义位置块。

许多元素与子域名元素相同，因此对于那些元素可以参考上一节，这里只看一些不同的指令。

```
# works with https://github.com/breakall/mytinytodo-docker
# set the mtt_url to 'https://your.domain.com/todo/' in db/config.php
```

若想使用反向代理，需要修改tinytodo容器的相关配置。

```
location ^~ /todo {
    return 301 $scheme://$host/todo/;
}
```

将 `https://linuxserver-test.com/todo` 的请求重定向到 `https://linuxserver-test.com/todo/`

```
location ^~ /todo/ {
}
```

任何使用 `https://linuxserver-test.com/todo/` 的请求都将匹配到这里

```
    set $upstream_app mytinytodo;
    set $upstream_port 80;
    set $upstream_proto http;
    proxy_pass $upstream_proto://$upstream_app:$upstream_port/;
```

与前面的示例相同，我们将 `upstream_app` 设置为 `mytinytudo` ，并告诉nginx使用该变量作为地址。要注意的是，此处的端口指的是容器的端口，因为nginx是通过docker的网络访问的。即使 `mytinytudo`容器使用 `-p 8080:80` 将端口映射了出去，但此处我们仍要将 upstream_port 设置为 `80`。

> Nginx在这里有一点要注意，即使我们设置代理地址为 `http://$upstream_mytinytodo:80/` ，但nginx实际上仍会去连接 `http://$upstream_mytinytodo:80/todo` 。每当我们在 proxy_pass URL 使用参数时，nginx都会在代理地址后面自动在后面加上 `location` 的位置（本例中为 /todo）。如果我们包含了子文件夹，那么nginx将尝试连接 `http://$upstream_mytinytodo:80/todo/todo` 并且导致失败。

### 例 - Ombi子域名反向代理

在此示例中，我们在 `https://ombi.linuxserver-test.com` 上反向代理Ombi

首先，确保我们的dns设置正确，并且它的A记录指向我们的服务器IP。如果我们使用docker cli，还需要像上面说的一样去创建自定义桥接网络（如，lsio）。再就是确保路由器的配置正确，443端口已经转发到我们的服务器上对应的端口上。

这是可以同时启动两个容器的docker compose配置文件：

```yaml
---
version: "2"
services:
  ombi:
    image: linuxserver/ombi
    container_name: ombi
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
    volumes:
      - /home/aptalca/appdata/ombi:/config
    ports:
      - 3579:3579
    restart: unless-stopped
  swag:
    image: linuxserver/swag
    container_name: swag
    cap_add:
      - NET_ADMIN
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
      - URL=linuxserver-test.com
      - SUBDOMAINS=wildcard
      - VALIDATION=dns
      - DNSPLUGIN=cloudflare
    volumes:
      - /home/aptalca/appdata/swag:/config
    ports:
      - 443:443
      - 80:80
    restart: unless-stopped
```

这是docker cli的命令：

Ombi：

```shell
docker create \
  --name=ombi \
  --net=lsio \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Europe/London \
  -p 3579:3579 \
  -v /home/aptalca/appdata/ombi:/config \
  --restart unless-stopped \
  linuxserver/ombi
```

SWAG：

```shell
docker create \
  --name=swag \
  --cap-add=NET_ADMIN \
  --net=lsio \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Europe/London \
  -e URL=linuxserver-test.com \
  -e SUBDOMAINS=wildcard \
  -e VALIDATION=dns \
  -e DNSPLUGIN=cloudflare \
  -p 443:443 \
  -p 80:80 \
  -v /home/aptalca/appdata/swag:/config \
  --restart unless-stopped \
  linuxserver/swag
```

当容器启动并运行后（即可以通过`https://linuxserver-test.com`访问），我们将 `/config/nginx/proxy-confs/` 文件夹下的 `ombi.subdomain.conf.sample` 重命名为 `ombi.subdomain.conf` 。然后重启SWAG容器使其生效。现在我们浏览 https://ombi.linuxserver-test.com 即可看到Ombi的界面。

### 例 - NextCloud子域名反向代理配置

NextCloud稍微有点麻烦，因为该程序内置了一些安全措施，我们需要手动配置一些内容。

与其他示例一样，确保dns、docker网络和路由器设置正确。对于DuckDNS，我们不需要创建CNAME，因为所有子域名的子域名都会自动指向与子域名相同的IP，只要确保它指向了正确的IP地址即可。

下面的示例中，我们使用duckdns通配符证书，当然你可以根据自己的实际情况进行更改。

下面的docker compose包含了SWAG、NextCloud、MariaDB三个容器：

```yaml
---
version: "2"
services:
  nextcloud:
    image: linuxserver/nextcloud
    container_name: nextcloud
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
    volumes:
      - /home/aptalca/appdata/nextcloud/config:/config
      - /home/aptalca/appdata/nextcloud/data:/data
    depends_on:
      - mariadb
    restart: unless-stopped
  mariadb:
    image: linuxserver/mariadb
    container_name: mariadb
    environment:
      - PUID=1000
      - PGID=1000
      - MYSQL_ROOT_PASSWORD=mariadbpassword
      - TZ=Europe/London
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=ncuser
      - MYSQL_PASSWORD=ncpassword
    volumes:
      - /home/aptalca/appdata/mariadb:/config
    restart: unless-stopped
  swag:
    image: linuxserver/swag
    container_name: swag
    cap_add:
      - NET_ADMIN
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
      - URL=linuxserver-test.duckdns.org
      - SUBDOMAINS=wildcard
      - VALIDATION=duckdns
      - DUCKDNSTOKEN=97654867496t0877648659765854
    volumes:
      - /home/aptalca/appdata/swag:/config
    ports:
      - 443:443
      - 80:80
    restart: unless-stopped
```

docker cli的版本：

NextCloud：

```shell
docker create \
  --name=nextcloud \
  --net=lsio
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Europe/London \
  -v /home/aptalca/appdata/nextcloud/config:/config \
  -v /home/aptalca/appdata/nextcloud/data:/data \
  --restart unless-stopped \
  linuxserver/nextcloud
```

MariaDB：

```shell
docker create \
  --name=mariadb \
  --net=lsio \
  -e PUID=1000 \
  -e PGID=1000 \
  -e MYSQL_ROOT_PASSWORD=mariadbpassword \
  -e TZ=Europe/London \
  -e MYSQL_DATABASE=nextcloud \
  -e MYSQL_USER=ncuser \
  -e MYSQL_PASSWORD=ncpassword \
  -v /home/aptalca/appdata/mariadb:/config \
  --restart unless-stopped \
  linuxserver/mariadb
```

SWAG：

```shell
docker create \
  --name=swag \
  --cap-add=NET_ADMIN \
  --net=lsio \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Europe/London \
  -e URL=linuxserver-test.duckdns.org \
  -e SUBDOMAINS=wildcard \
  -e VALIDATION=duckdns \
  -e DUCKDNSTOKEN=97654867496t0877648659765854 \
  -p 443:443 \
  -p 80:80 \
  -v /home/aptalca/appdata/swag:/config \
  --restart unless-stopped \
  linuxserver/swag
```

将SWAG容器的`/config/nginx/proxy-confs`文件夹下的`nextcloud.subdomain.conf.sample`重命名为`nextcloud.subdomain.conf`并重新启动SWAG容器。

如果这是第一次访问NextCloud（包括以前从未在本地访问过），我们可以直接通过访问 `https://nextcloud.linuxserver-test.duckdns.org` 进行配置即可。

![](https://pic.watercalmx.com/pic/Screenshot-2019-04-14-at-18.35.02---Edited.png)

完成安装后，我们就可以看到NextCloud的介绍幻灯片并进入到仪表盘中，在地址栏上也能看到小锁头。

![](https://pic.watercalmx.com/pic/Screenshot-2019-04-14-at-18.40.45---Edited.png)

如果我们之前访问过NextCloud或者这是一个已有的实例，我们直接设置反向代理会被拒绝访问。这种情况下，我们需要按照`nextcloud.subdomain.conf`文件上方被注释的内容进行配置：

```
# assuming this container is called "swag", edit your nextcloud container's config
# located at /config/www/nextcloud/config/config.php and add the following lines before the ");":
#  'trusted_proxies' => ['swag'],
#  'overwrite.cli.url' => 'https://nextcloud.your-domain.com/',
#  'overwritehost' => 'nextcloud.your-domain.com',
#  'overwriteprotocol' => 'https',
#
# Also don't forget to add your domain name to the trusted domains array. It should look somewhat like this:
#  array (
#    0 => '192.168.0.1:444', # This line may look different on your setup, don't modify it.
#    1 => 'nextcloud.your-domain.com',
#  ),
```

这些配置将告诉NextCloud，这些代理是应该被信任的，并且应该将域名如何重定向。

> 如果你是按照上述的说明进行第一次的设置，那么只需要添加`'trusted_proxies'=> ['swag']`即可，不然NextCloud 16 以上的版本仍会提示有关反向代理设置错误的警告。
>
> 默认情况下，SWAG的配置中禁用了HSTS。你可以在SWAG的ssl.conf中启用它。

### 例 - Plex子文件夹反向代理

在这个示例中，我们将设置Plex的访问地址为 `https://linuxserver-test.com/plex`。起初，我们将通过本地网络进行配置Plex，并在同一个网络中访问它。如果需要通过其他网络访问或使用桥接网络，可以使用 `PLEX_CLAIM` 变量通过plex账户来声明服务器。

Plex服务器设置成功后，便可以将其从host网络切换到桥接网络上。

下面是docker compose启动所有容器的配置：

```yaml
---
version: "2"
services:
  plex:
    image: linuxserver/plex
    container_name: plex
    network_mode: host
    environment:
      - PUID=1000
      - PGID=1000
      - VERSION=docker
    volumes:
      - /home/aptalca/appdata/plex:/config
      - /home/aptalca/tvshows:/data/tvshows
      - /home/aptalca/movies:/data/movies
    restart: unless-stopped
  swag:
    image: linuxserver/swag
    container_name: swag
    cap_add:
      - NET_ADMIN
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
      - URL=linuxserver-test.com
      - SUBDOMAINS=wildcard
      - VALIDATION=dns
      - DNSPLUGIN=cloudflare
    volumes:
      - /home/aptalca/appdata/swag:/config
    ports:
      - 443:443
      - 80:80
    restart: unless-stopped
```

下面是docker cli的版本：

Plex：

```shell
docker create \
  --name=plex \
  --net=host \
  -e PUID=1000 \
  -e PGID=1000 \
  -e VERSION=docker \
  -v /home/aptalca/appdata/plex:/config \
  -v /home/aptalca/tvshows:/data/tvshows \
  -v /home/aptalca/movies:/data/movies \
  --restart unless-stopped \
  linuxserver/plex
```

SWAG：

```shell
docker create \
  --name=swag \
  --cap-add=NET_ADMIN \
  --net=lsio \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Europe/London \
  -e URL=linuxserver-test.com \
  -e SUBDOMAINS=wildcard \
  -e VALIDATION=dns \
  -e DNSPLUGIN=cloudflare \
  -p 443:443 \
  -p 80:80 \
  -v /home/aptalca/appdata/swag:/config \
  --restart unless-stopped \
  linuxserver/swag
```

设置容器后，我们访问 http://LOCALSERVERIP:32400/web 并使用Plex账户设置Plex服务器。然后我们将SWAG容器里 `/config/nginx/proxy-confs` 下的 `plex.subfolder.conf.sample` 重命名为 `plex.subfolder.conf` 

如果我们为plex容器使用桥接网络，则可以重新启动SWAG容器，并应该能够通过 `https://linuxserver-test.com/plex` 访问Plex。

如果Plex容器使用host网络，那么还必须修改 `plex.subfolder.conf` 。找到 `proxy_pass http:// $ upstream_plex:32400;` 修改为Plex的本地IP地址，即 `proxy_pass http://192.168.1.10:32400;` 。然后重启SWAG后，就可以通过 `https://linuxserver-test.com/plex` 访问Plex。

如果我们希望保持Plex通过自己域名进行连接（包括移动端的App），则可以将 `https://linuxserver-test.com/plex` 添加到Plex服务器设置中的`“Custom server access URLs”` 。之后便可以再Plex服务器中关闭远程访问并删除32400端口。所有Plex服务器的连接都将通过443端口的SWAG进行反向代理。

### 例 - 使用Heimdall作为网站根目录的主页

在此示例中，我们将把Heimdall设置为网站根目录的主页，即直接访问 `https://linuxserver-test.com` 将到达Heimdall。

跟之前一样，我们需要确保端口转发等设置已经设置完毕。

下面是docker compose的配置：

```yaml
---
version: "2"
services:
  heimdall:
    image: linuxserver/heimdall
    container_name: heimdall
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
    volumes:
      - /home/aptalca/appdata/heimdall:/config
    restart: unless-stopped
  swag:
    image: linuxserver/swag
    container_name: swag
    cap_add:
      - NET_ADMIN
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
      - URL=linuxserver-test.com
      - SUBDOMAINS=wildcard
      - VALIDATION=dns
      - DNSPLUGIN=cloudflare
    volumes:
      - /home/aptalca/appdata/swag:/config
    ports:
      - 443:443
      - 80:80
    restart: unless-stopped
```

下面是docker cli的版本：

Heimdall：

```shell
docker create \
  --name=heimdall \
  --net=lsio \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Europe/London \
  -v /home/aptalca/appdata/heimdall:/config \
  --restart unless-stopped \
  linuxserver/heimdall
```

SWAG：

```shell
docker create \
  --name=swag \
  --cap-add=NET_ADMIN \
  --net=lsio \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Europe/London \
  -e URL=linuxserver-test.com \
  -e SUBDOMAINS=wildcard \
  -e VALIDATION=dns \
  -e DNSPLUGIN=cloudflare \
  -p 443:443 \
  -p 80:80 \
  -v /home/aptalca/appdata/swag:/config \
  --restart unless-stopped \
  linuxserver/swag
```

设置好之后，我们重命名SWAG容器里 `/config/nginx/proxy-confs` 文件夹下的 `heimdall.subfolder.conf.sample` 为 `heimdall.subfolder.conf` 。当查看配置文件时，我们发现位置块配置为了根目录 `location / {` ，这会引起冲突。因为在SWAG的默认站点配置中已经使用了 location / {  。因此，我们需要注释掉 `/config/nginx/site-confs/default` 文件中的位置块：

```
    #location / {
    #    try_files $uri $uri/ /index.html /index.php?$args =404;
    #}
```

这样，nginx将使用heimdall配置文件中的位置块。

再次访问 `https://linuxserver-test.com` 我们将看到Heimdall的界面。

如果我们希望用密码来保护这个主页，可以在宿主机上运行以下命令创建新的 `.htpasswd` 文件：`docker exec -it swag htpasswd -c /config/nginx/.htpasswd anyusername`。然后在 `heimdall.subfolder.conf` 文件中添加如下几行来启用它：

```
    # enable the next two lines for http auth
    auth_basic "Restricted";
    auth_basic_user_file /config/nginx/.htpasswd;
```

------

## 总结

该镜像的作用远不止于此，因为它就是一个功能完善的Web服务器，以上的示例足以使你入门。更多的信息请参阅[GitHub](https://github.com/linuxserver/docker-swag/blob/master/README.md)或[Docker Hub](https://hub.docker.com/r/linuxserver/swag)上的官方文档。如果有任何疑问和问题或想要分享的想法，可以来我们的discord一起探讨。 → https://discord.gg/YWrKVTn

### 如何寻求帮助

如你所见，这里存在着很多不同的配置，因此在需要帮助之前，请提供你确切的设置方式。如果遇到bug并确定是bug，请在GitHub上进行报告。如果需要配置方面的帮助，请加入我们的论坛，然后将以下信息发到类似于PasteBin的在线工具上并附上链接发到论坛上寻求帮助：

- 你使用的创建docker的命令或compose的yaml
- 完整的docker日志（如，docker logs swag）
- 任何相关的配置文件（一般来说是 nginx.conf 或 特定的代理配置文件）