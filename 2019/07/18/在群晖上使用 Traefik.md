# 在群晖上使用 Traefik

这篇文章聊聊如何在群晖系统上使用 Traefik，让 NAS 设备摇身一变为好用的 Web 服务器。

<!-- more -->

迄今为止，我已经写了接近三十篇搭配 Traefik 使用的各种开源软件记录，所以如果你想了解更多，不妨看看那些[历史文章](https://soulteary.com/tags/traefik.html)。

## 写在前面

![使用一个崭新的系统进行演示](https://attachment.soulteary.com/2019/07/18/begin.png)

因为我家里的设备已经有安装 Traefik ，为了能够使用干净纯粹的环境，本次基于虚拟机进行演示：虚拟机群晖系统版本 6.1+，可以用于 6.2+ 的系统使用（新版本只有界面有差异，功能、配置方面是一致的）。

![配置 SSL 证书](https://attachment.soulteary.com/2019/07/18/configure-ssl.png)

为了方便文章描述，我自签了证书，并将域名配置给了这台“群晖”虚拟机。

在群晖上使用 Traefik 有两种玩法：

- 单独使用 Traefik ，指定一个非 80 / 443 端口提供服务。
- 使用 Traefik 配合系统自带的 Nginx 使用，支持通过 80 /443 端口访问服务。

在继续聊 Traefik 前，必须先了解群晖系统的一些**默认逻辑**：

- 群晖**默认提供** Web 界面 ，支持用户在浏览器上使用 5000 或者 5001 端口进行 HTTP/HTTPS 的方式使用系统，可以配置用户自己的 SSL 证书。
- 群晖**默认逻辑**是用户直接访问 IP 或者主机域名后（不带端口号），直接跳转上面的 5000/5001 端口。
- 群晖各种应用/共享协议会使用低位端口号，如果不想冲突，用户定义端口**需要规避**这类端口，如发生端口冲突的事情，冲突软件就只能进行“二选一”了，那个软件先启动，那么另外一个就只能报错退出了。

所以这里我们需要记住两条**基础规则**：

- 80/443 端口没有那么好用，使用 Traefik 等三方软件得带着端口号，忍受“不完美”。
- 需要规避一堆低位端口，避免让系统/应用功能不可用。

先聊聊如何单独使用 Traefik。

（下文中使用的域名需要自己进行 hosts 绑定或者 DNS 解析指向）

## 单独使用 Traefik

单独使用 Traefik 非常简单，就像上面两条规则描述的那样。

不过为了方便后续维护，Traefik 推荐运行在容器当中，所以如果之前没有安装它的话，需要要在套件中找到 Docker 并进行安装，安装完毕之后，可以看到 FileStation 中多了一个名为 docker 的目录。

![安装 docker 套件](https://attachment.soulteary.com/2019/07/18/install-docker.png)

### 将 Traefik 作为服务运行

Traefik 默认的端口是 80 ，上文提到过，这个端口默认被系统占用，所以我们这里将端口映射到一个相对冷僻的高位数字： `52080` 。

```yaml
version: '3'

services:

  traefik:
    image: traefik:v1.7-alpine
    restart: always
    container_name: traefik
    ports:
      - 52080:80
    networks:
      - traefik
    command: traefik -c /etc/traefik.toml
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /var/run/docker.sock:/var/run/docker.sock
      - ./traefik.toml:/etc/traefik.toml
    healthcheck:
      test: ["CMD-SHELL", "wget -q --spider --proxy off localhost:4399/ || exit 1"]

networks:
  traefik:
    external: true
```

将配置保存为 `docker-compose.yml` ，我们继续编写 traefik 的配置文件： `traefik.toml`。

```Toml
debug = false
sendAnonymousUsage = false
defaultEntryPoints = ["http"]

[entryPoints]
    [entryPoints.http]
        address = ":80"
        compress = true
    [entryPoints.traefik-api]
        address = ":4399"

[file]
    [backends]
        [backends.dashboard]
            [backends.dashboard.servers.server1]
                url = "http://127.0.0.1:4399"

[frontends]
    [frontends.dashboard]
        entrypoints = ["http"]
        backend = "dashboard"
        [frontends.dashboard.routes.route01]
            rule = "Host:dashboard.orange.lab.com"

[traefikLog]
filePath = "/tmp/traefik.log"

[accessLog]
filePath = "/tmp/access.log"

[api]
entryPoint = "traefik-api"
dashboard = true
defaultEntryPoints = ["http"]

[docker]
endpoint = "unix:///var/run/docker.sock"
domain = "traefik.orange.lab.com"
watch = true
exposedbydefault = false
usebindportip = false
swarmmode = false
```

将两个文件单独保存之后，把文件上传到群晖上，启动 Traefik：
（这里以刚刚系统自动创建的 docker 目录为例）

```yaml
# 创建目录
mkdir -p /volume1/docker/traefik
cd /volume1/docker/traefik

docker network create traefik
docker-compose up -d
```

命令执行完毕后，访问 `dashboard.orange.lab.com:52080` 就能看到 Traefik 的 Dashboard 了。

![暂时空空如也的 Dashboard](https://attachment.soulteary.com/2019/07/18/traefik-dashboard.png)

因为暂时没有运行其他的应用，所以 Dashboard 看起来空空如也。

那么，让我们来添加两个应用，测试下 Traefik 的功能吧。

### 安装第一个应用（WordPress）

和 Nginx 作为反向代理不同的是，使用 Traefik 添加应用只需要注明一条规则，就能够让你的应用使用某个域名进行访问了，简化了非常多操作。

这里使用[之前的文章](https://soulteary.com/2019/04/07/use-docker-and-traefik-to-build-wordpress.html) 里的 WordPress 配置，并进行简化，启动第一个测试应用。

```yaml
version: '3'

services:

  wp:
    image: wordpress:5.2.2-php7.1-apache
    restart: always
    networks:
      - traefik
    environment:
        WORDPRESS_DB_HOST: wp-db
        WORDPRESS_TABLE_PREFIX: wp
        WORDPRESS_DB_NAME: wordpress
        WORDPRESS_DB_USER: wordpress
        WORDPRESS_DB_PASSWORD: wordpress
    labels:
      - "traefik.enable=true"
      - "traefik.frontend.rule=Host:wp.orange.lab.com"
      - "traefik.frontend.entryPoints=http"

  mariadb:
    image: mariadb:10.3.8
    restart: always
    container_name: wp-db
    networks:
      - traefik
    environment:
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress
      MYSQL_ROOT_PASSWORD: soulteary

networks:
  traefik:
    external: true
```

将上面的配置保存为 `docker-compose.yml` ，和处理 traefik 类似，我们将它也上传到群晖的目录中，并将容器启动起来。

```bash
mkdir -p /volume1/docker/wordpress
cd /volume1/docker/wordpress

docker-compose up -d
```

稍等片刻，打开 `http://wp.orange.lab.com:52080` 就能看到熟悉的安装界面了。

![熟悉的 WordPress 安装界面](https://attachment.soulteary.com/2019/07/18/install-wordpress.png)

### 安装第二个应用（Nginx）

Nginx 除了作为服务端常常使用的服务网关外，还经常作为动静态站点的 Web 前端软件。

这里同样使用一篇[之前文章](https://soulteary.com/2019/04/07/use-docker-and-traefik-to-build-wordpress-with-nginx.html) 里的 Nginx 配置，并进行简化，启动第二个测试应用。

```yaml
version: '3'

services:

  nginx:
    image: nginx:1.15.10-alpine
    restart: always
    networks:
      - traefik
    expose:
      - 80
    labels:
      - "traefik.enable=true"
      - "traefik.frontend.rule=Host:ngx.orange.lab.com"
      - "traefik.frontend.entryPoints=http"

networks:
  traefik:
    external: true
```

同样将上面的配置保存为 `docker-compose.yml` ，和处理前面的应用类似，我们还是将它上传到群晖的目录中，并将容器启动起来。

```bash
mkdir -p /volume1/docker/nginx
cd /volume1/docker/nginx

docker-compose up -d
```

稍等片刻，打开 `http://nginx.orange.lab.com:52080` 你就能看到 “Welcome to nginx!” 的默认运行界面啦。

![Nginx 默认运行界面](https://attachment.soulteary.com/2019/07/18/install-nginx.png)

如果你经常搭建网站、尤其是在同一台机器进行搭建，你会发现使用 Traefik 来做“服务域名管理”确实是非常高效的。

我们再来看看之前的 Traefik Dashboard 吧。

![再看 Traefik Dashboard](https://attachment.soulteary.com/2019/07/18/traefik-dashboard-with-apps.png)

这里会展示所有正确注册服务发现的应用，所以如果你在浏览器里打不开你的应用，可以在这里检查下应用是否存在、以及应用配置是否正确。

## 搭配 Web Station 使用

聊完 Traefik 独立使用，我们来讲讲怎么去掉地址栏里多余的端口号，比如上文中出现的 “52080”。

因为群晖更新频繁，每次更新都会覆盖用户对系统软件的修改。所以我们既要保证修改不会影响群晖各种功能正常，又要让我们的修改不受到群晖系统或者软件升级所影响。

### 改变群晖默认行为

我们知道，如果浏览器中想隐藏端口，需要使用两个默认端口：80 和 443 。这两个端口分别对应 HTTP 和 HTTPS 两种协议状况。

前文提到过群晖默认访问 80/443 端口会跳转到 5000/5001 端口的后台页面，但是如果我们安装官方提供的 Web Station 套件后，这个默认行为**就可以被打破了**。

![Web Station 安装之后](https://attachment.soulteary.com/2019/07/18/change-default-ui.png)

当安装完毕 Web Station 之后，我们再次访问群晖的域名或者IP，将看到上面这个蓝色的默认页面。

![安装 Web Station](https://attachment.soulteary.com/2019/07/18/install-webstation.png)

并且在 File Station 中，我们能看到有一个叫做 `web` 的目录被自动创建出来了，里面保存的文件就是我们看到的“蓝色界面”。


### 使用 Web Station 代理 Traefik 请求

既然群晖设备的地址可以去掉端口号，那么刚刚两个使用 Traefik 通过域名暴露服务的软件也没有什么问题。

打开 Web Station 套件，我们使用域名添加一个网站。

![添加虚拟主机](https://attachment.soulteary.com/2019/07/18/add-vhost.png)

此刻，如果你使用这个域名打开网站，会发现网站的界面居然还是那个蓝色的初始界面。这是因为 Web Station 默认生成的配置只支持比较简单的场景，不过修改也很容易。

 使用终端切换到 `/etc/nginx/conf.d` 目录，我们会看到一些配置和一些目录，这些目录默认是空的。

```bash
/etc/nginx/conf.d# ls
0a977aa1-e8b6-4f98-9f0a-b595268aaa5b  dsm.docker.conf  dsm.ssdp.conf  events.conf  main.conf
```

在上面的 `0a977aa1-e8b6-4f98-9f0a-b595268aaa5b` 目录添加一个名为 `user.conf`的配置。

```bash
location / {
    proxy_set_header Host                $http_host;
    proxy_set_header X-Real-IP           $remote_addr;
    proxy_set_header X-Forwarded-For     $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto   $scheme;
    proxy_intercept_errors on;
    proxy_http_version 1.1;
    proxy_pass http://127.0.0.1:52080/;
}
```

 然后使用下面的命令重启群晖的 Web Station：

```bash
/usr/syno/bin/synopkg restart WebStation
```

当然，也可以使用标准的 nginx 命令

```bash
nginx -t && nginx -s reload
```

![重启 Web Station 后，WP 就正常了](https://attachment.soulteary.com/2019/07/18/add-vhost-success.png)

然后前文中我们启动的 WordPress 就能够正常使用了。按照上面的方法，再重复操作几次，其他的站点也都能去掉端口运行啦。

## 其他

在安装配置群晖系统的时候，其实我们除了可以打开 `http://find.synology.com/` 或者下载使用 Synology Assistant 外。

只需要使用系统自带的 `arp` 命令就能发现等待操作的群晖设备，比如下面这样。

```bash
# arp -a

pear.pear (10.11.12.13) at 20:76:93:xx:yy:zz on en0 ifscope [ethernet]
notebook.pear (10.11.12.110) at 8c:85:90:xx:yy:zz on en0 ifscope permanent [ethernet]
diskstation.pear (10.11.12.179) at 0:11:32:xx:yy:zz on en0 ifscope [ethernet]
? (172.16.24.1) at 0:50:5xx:yy:zz on vmnet1 ifscope permanent [ethernet]
? (192.168.247.1) at 0:50:5xx:yy:zz on vmnet8 ifscope permanent [ethernet]
? (224.0.0.251) at 1:0:5xx:yy:zz on en0 ifscope permanent [ethernet]
? (239.255.255.250) at 1:0:5e:xx:yy:zz on en0 ifscope permanent [ethernet]
```


## 最后

近几年出现的群晖设备性能越来越强，甚至远胜前几年使用 ATOM CPU 的Web 服务器。

如果只是让他们简单的做一个存储型设备，未免太过浪费，物尽其用，或许会更好一些。

—EOF
