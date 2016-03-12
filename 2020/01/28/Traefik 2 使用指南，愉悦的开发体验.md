# Traefik 2 使用指南，愉悦的开发体验

2018年 写过 [使用服务发现改善开发体验](https://soulteary.com/2018/06/11/use-server-side-discovery-improve-development.html)，里面提到了一些开发过程的痛点，其中使用了 Traefik 作为服务网关 / 服务发现工具。

在耐心等待 Traefik 升级到 2.1 之后，开始正式着手升级应用。

下面就来聊聊，怎么更好的使用 Traefik 2 吧。

## 写在前面

相比较 Traefik 1 来说，2.x 从设计到功能都有了比较大的改变，原始的配置和规则基本都会遇到不兼容的问题。

打一个比方，如果说 1.x 版本是大单体应用，那么 2.x 版本，各个模块都被拆的很细，允许用户像乐高一样使用它，而且开始支持 TCP 协议，自由度大大提升，不过因为自由度的提升，使用的成本也有了一定的增加。

当然，官方商业版本还是基于 v1.x ，所以暂时不升级，问题也还没有那么大，但是如果你想使用 Traefik 像 Hadoop 一样处理 TCP 流量，那么升级无疑是最好的选择。

## 新版界面预览

在实际动手前，可以先看看新版的界面。

相比较老版本看起来更加直观了。根据资源类型划分了不同的区域“接入点”、“HTTP”、“TCP”、“其他”，对于调试或排查问题方便了不少。

![新版 Dashboard](https://attachment.soulteary.com/2020/01/28/traefik-overview.jpg)

新版本终于将路由独立了出来，并且能够直观的看到某条路由的全链路。

![路由列表](https://attachment.soulteary.com/2020/01/28/traefik-route-list.jpg) 

在应用详情页能够清晰的了解到所有该了解到东西，从入口点到服务路由，再到中间件、以及最终的后端应用清晰可见。

![应用详情页](https://attachment.soulteary.com/2020/01/28/traefik-app-detail.jpg)

## 准备环境

推荐使用以下版本或比该版本更高的软件，本文成稿时，我使用的软件版本是：

- Docker version 19.03.5
- docker-compose version 1.25.2
- Traefik version 2.1.3

## Traefik 的 Compose配置文件升级

这里依旧选择使用 Compose 来进行 Traefik 的服务启动和管理，简单够用。先来看看 Traefik 1.7 的 docker-compose.yml ：

```yaml
version: '3.6'

services:

  traefik:
    container_name: traefik
    image: traefik:v1.7-alpine
    restart: always
    ports:
      - 80:80
      - 443:443
      - 4399:4399
      - 4398:4398
    networks:
      - traefik
    command: traefik -c /etc/traefik.toml
    labels:
      - "traefik.enable=false"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./traefik.toml:/etc/traefik.toml
      - ./ssl/:/data/ssl/
    healthcheck:
      test: ["CMD-SHELL", "wget -q --spider --proxy off localhost:4398/ping || exit 1"]

# 先创建外部网卡
# docker network create traefik
networks:
  traefik:
    external: true
```

升级后的配置可以看到基本没有变化，甚至还简短了一些，2.x 中，官方特别声明可以使用动态配置，所以这里多了一条目录映射规则 `./config/:/etc/traefik/config/:ro`。

```yaml
version: '3.7'

services:

  traefik:
    container_name: traefik
    image: traefik:v2.1.3
    restart: always
    ports:
      - 80:80
      - 443:443
    networks:
      - traefik
    command: traefik --configFile /etc/traefik.toml
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - ./ssl/:/data/ssl/:ro
      - ./traefik.toml:/etc/traefik.toml:ro
      - ./config/:/etc/traefik/config/:ro
    healthcheck:
      test: ["CMD-SHELL", "wget -q --spider --proxy off localhost:8080/ping || exit 1"]

# 先创建外部网卡
# docker network create traefik
networks:
  traefik:
    external: true
```

当然，作为服务网关，得有服务健康自检，默认的时间太长，建议每 3～5 秒检查一次。

```yaml
healthcheck:
  test: ["CMD-SHELL", "wget -q --spider --proxy off localhost:8080/ping || exit 1"]
  interval: 3s
  timeout: 5s
```

## Traefik 的应用配置升级

还是先来看 1.7 版本的 Traefik 配置文件 traefik.toml ：

```Toml
debug = false
logLevel = "WARN"
defaultEntryPoints = ["http", "https"]
sendAnonymousUsage = false

[entryPoints]

    [entryPoints.http]
        address = ":80"
        compress = true
    [entryPoints.https]
        address = ":443"
    [entryPoints.https.tls]
        [[entryPoints.https.tls.certificates]]
            certFile = "/data/ssl/lab.io.crt"
            keyFile = "/data/ssl/lab.io.key"
        [[entryPoints.https.tls.certificates]]
            certFile = "/data/ssl/lab.com.crt"
            keyFile = "/data/ssl/lab.com.key"
    [entryPoints.traefik-api]
        address = ":4399"
    [entryPoints.traefik-ping]
        address = ":4398"

[file]
    [backends]
        [backends.dashboard]
            [backends.dashboard.servers.server1]
                url = "http://127.0.0.1:4399"
        [backends.ping]
            [backends.ping.servers.server1]
                url = "http://127.0.0.1:4398"

[frontends]
    [frontends.dashboard]
        entrypoints = ["http", "https"]
        backend = "dashboard"
        [frontends.dashboard.routes.route01]
            rule = "Host:dashboard.lab.io"
    [frontends.ping]
        entrypoints = ["http", "https"]
        backend = "ping"
        [frontends.ping.routes.route01]
            rule = "Host:ping.lab.com"
        [frontends.ping.routes.route02]
            rule = "ReplacePathRegex: ^/ /ping"

[api]
    entryPoint = "traefik-api"
    dashboard = true
    defaultEntryPoints = ["http"]

[ping]
    entryPoint = "traefik-ping"

[docker]
endpoint = "unix:///var/run/docker.sock"
domain = "lab.io"
watch = true
exposedbydefault = false
usebindportip = false
swarmmode = false
```

上面这个大概六十多行的配置文件，轻松实现了一个支持 80 / 443 （SSL）网关、内部管理服务根据域名进行负载发现，但是每次想更新证书、想添加静态的服务就很麻烦了，因为不得不在更新内容后重启 Traefik 服务。

而 Traefik 2.0 支持从目录读取配置、支持动态加载，所以类似上面的问题就不存在了，只要对配置做好静态、动态配置拆分就好了，先来看静态配置 traefik.toml ：

```Toml
[global]
  checkNewVersion = false
  sendAnonymousUsage = false

[log]
  level = "WARN"
  format = "common"

[api]
  dashboard = true
  insecure = true

[ping]

[accessLog]

[providers]
  [providers.docker]
    watch = true
    exposedByDefault = false
    endpoint = "unix:///var/run/docker.sock"
    swarmMode = false
    useBindPortIP = false
    network = "traefik"
  [providers.file]
    watch = true
    directory = "/etc/traefik/config"
    debugLogGeneratedTemplate = true

[entryPoints]
  [entryPoints.http]
    address = ":80"
  [entryPoints.https]
    address = ":443"
```

不到四十行的主文件，实现了上面老配置的大多数功能，接下来来分别处理SSL证书管理和动态服务发现的问题，先聊聊证书管理。

```Toml
[tls]
  [tls.options]
    [tls.options.default]
      minVersion = "VersionTLS12"
      maxVersion = "VersionTLS12"
    [tls.options.test-tls13]
      minVersion = "VersionTLS13"
      cipherSuites = [
        "TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256",
        "TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384",
        "TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305",
        "TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305",
        "TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256",
        "TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256",
      ]

  [[tls.certificates]]
    certFile = "/data/ssl/lab.io.crt"
    keyFile = "/data/ssl/lab.io.key"

  [[tls.certificates]]
    certFile = "/data/ssl/lab.com.crt"
    keyFile = "/data/ssl/lab.com.key"
```

将上面的文件保存为 **tls.toml** 并保存在 config 目录下，就好了。相比老版本的 Traefik， 新版的 Traefik 不光是可以定制每个请求响应使用的 TLS 版本，还可以定制加密算法、以及独立为某个/某些域名单独进行配置（就像上面这样）！

动态服务发现，同样的可以被拆分为单独的配置文件，不过相比较老版本，新版本比较麻烦的一点是 HTTP  协议自动跳转 HTTPS 协议需要一点 Hacks ，老版本设置 HTTP 自动跳转 HTTPS 比较简单，只需要 2 行就行。

```Toml
[entryPoints.http.redirect]
  entryPoint = "https"
```

而新版本需要像下面这样配置：

```Toml
[http.middlewares.https-redirect.redirectScheme]
  scheme = "https"
[http.middlewares.content-compress.compress]

# tricks
# https://github.com/containous/traefik/issues/4863#issuecomment-491093096
[http.services]
  [http.services.noop.LoadBalancer]
     [[http.services.noop.LoadBalancer.servers]]
        url = "" # or url = "localhost"

[http.routers]
  [http.routers.https-redirect]
    entryPoints = ["http"]
    rule = "HostRegexp(`{any:.*}`)"
    middlewares = ["https-redirect"]
    service = "noop"
```

这里我们相当于定义了几个公共方法，不过好处是我们可以单独的为后续使用 Traefik 的每一个服务单独配置是否进行 HTTP-\>HTTPS 跳转，将上面的内容保存为 **default.toml** ，继续处理服务发现的配置。

```Toml
[http.middlewares.dash-compress.compress]
[http.middlewares.dash-auth.basicAuth]
  users = [
    "test:$apr1$H6uskkkW$IgXLP6ewTrSuBkTrqE8wj/",
    "test2:$apr1$d9hr9HBB$4HxwgUir3HP4EsggP/QNo0",
  ]

[http.routers.dashboard-redirect-https]
  rule = "Host(`dashboard.lab.io`, `dashboard.lab.com`)"
  entryPoints = ["http"]
  service = "noop"
  middlewares = ["https-redirect"]
  priority = 100

[http.routers.dashboard]
  rule = "Host(`dashboard.lab.io`, `dashboard.lab.com`)"
  entrypoints = ["https"]
  service = "dashboard@internal"
  middlewares = ["dash-auth", "dash-compress"]
  [http.routers.dashboard.tls]

[http.routers.api]
  rule = "Host(`dashboard.lab.io`, `dashboard.lab.com`) && PathPrefix(`/api`)"
  entrypoints = ["https"]
  service = "api@internal"
  middlewares = ["dash-auth", "dash-compress"]
  [http.routers.api.tls]

[http.routers.ping]
  rule = "Host(`dashboard.lab.io`, `dashboard.lab.com`) && PathPrefix(`/ping`)"
  entrypoints = ["https"]
  service = "ping@internal"
  middlewares = ["dash-auth", "dash-compress"]
  [http.routers.ping.tls]
```

将配置保存为**dashboard.lab.com.toml** [^1]  ，至此就基本完成了老配置 Traefik 的所有功能，后续如果有“规则”需要变化，只需要修改刚刚这几个文件即可，而无需重启 Traefik 就能生效了。

## 其他

调试学习 Traefik 的时候，发现 Traefik 容器镜像中的 entrypoint.sh 写的很有意思。

```bash
#!/bin/sh
set -e

# first arg is `-f` or `--some-option`
if [ "${1#-}" != "$1" ]; then
    set -- traefik "$@"
fi

# if our command is a valid Traefik subcommand, let's invoke it through Traefik instead
# (this allows for "docker run traefik version", etc)
if traefik "$1" --help >/dev/null 2>&1
then
    set -- traefik "$@"
else
    echo "= '$1' is not a Traefik command: assuming shell execution." 1>&2
fi

exec "$@"
```

简单几行脚本，实现了如果执行命令并非 Traefik 应用命令，执行系统命令的逻辑，值得容器镜像封装时学习。

## 最后

下一篇将聊聊之前的老应用们该如何升级。

--EOF

[^1]:	原始文件名 "dashboard.lab.com " 存在笔误，感谢网友陈卫弥勘误。