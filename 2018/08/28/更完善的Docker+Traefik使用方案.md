# 更完善的 Docker + Traefik 使用方案

在踩坑无数之后，多次修改后，这篇草稿箱中的文字终于得以成型，撒花。

六月更新架构的时候，去掉了 `openresty` 作为服务器前端，取而代之的是裸跑 `Traefik`，因为只暴露网关的 `80` / `443`，后面所有子容器都是以 `expose` 方案对内暴露端口到一块虚拟网卡上，安全问题也不大，网关挂载着通配符证书，可以方便的添加删除后面的应用，虽说用起来挺舒服的，但是有两点始终让我不是很爽。

1. 因为 docker 直接使用 iptable 操作端口转发，在不修改 iptable 的前提下，之前积累了大量的 `fail2ban` 规则无法使用了，也就是说得忍受大量扫描器污染日志。
2. 因为应用跑完全隔离的方案，子容器能拿得到的客户端 IP 都是虚拟网卡 IP ，想统计个数据，得用对账的方案，比较麻烦。

问了一下之前出去创业的师傅，他们直接在给容器分配公网，壕无人道，而且直接分配公网 IP 配置项目也不少...

如何能在最少配置的情况下，最少资源消耗的情况下，达到现有使用的便捷程度，便更为了我这篇文字的主要目的。

赶在休假结束之前，重新梳理了服务器运行环境和Traefik的使用，记录下来，或许对折腾“免”运维的服务的你也会有帮助。

在折腾具体配置之前，需要先提供标准的系统环境。

## 基于 Ubuntu 18.04 系统配置 Docker 运行环境

2018年第三个季度，Ubuntu 18.04 更新了快一个季度了，基本上该更新的软件也都更新了，该兼容的软件也都进行了兼容处理；Docker 在经历改名风波后，版本迭代高歌猛进，一大波编排软件蜂拥而至，现在版本也升级到了 v18 。

考虑到后面软件会做越来越多向后兼容的事情，16.04 的维护周期也近半，那么这次就使用最新的系统和软件来进行基础环境的配置吧，这里没有选择 CoreOS 是因为我这里还有一些其他软件的使用需求，想在一个相对中立的环境中使用。

> 截止这篇文章写成：
> 我使用的国外主流云厂商皆支持 Ubuntu 18.04，
> 国内的云厂商中，阿里云支持，但是腾讯云暂时还不支持。

如果你的云主机厂商支持最新版本的系统，那么可以直接参考我下面给出的命令进行基础环境配置。

如果你的云主机厂商不支持，那么请使用 `do-release-upgrade` 命令先进行手动升级，升级过程中可以一路 `yes`，以及使用当前软件维护的最新版本: `install the package maintainer's version`。

下面开始介绍如何配置最基础的系统环境。

先更新软件包列表，升级软件版本到最新的稳定版，然后为避免系统中残留一些老古董影响软件运行，我们要进行尝试性的卸载老版本软件操作，以及安装一些常用软件。

```bash
apt update && apt upgrade -y
apt remove docker docker-engine docker.io
apt install -y apt-transport-https  ca-certificates curl software-properties-common
```

向系统中添加 Docker 官方 GPG Key，然后验证该 Key 有效性，并更新仓库源到系统，Ubuntu 18.04 会直接触发拉取软件包列表的操作，比较人性化，最后直接敲入 `install` 命令，进行社区版的 docker 安装即可。

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
apt-key fingerprint 0EBFCD88
add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
apt install -y docker-ce
```

在 `https://github.com/docker/compose/releases` 找到最新稳定版本安装。

```bash
curl -L https://github.com/docker/compose/releases/download/1.22.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

至此，基础的 docker 环境就安装就绪了。

```bash
Docker version 18.06.1-ce, build e68fc7a
docker-compose version 1.22.0, build f46880fe
Storage Driver: overlay2
```

## 简单的系统加固

基础的修改 ssh 端口，规避扫描器，应该人人都会，就略过不提，如果不会可以百度或者翻阅之前的博客文章，我们来说说 ufw 防火墙的配置。

服务默认会是未激活状态，在激活之前，务必先豁免 ssh 端口，避免再使用 vnc 登录上去补救。

```bash
> ufw status
Status: inactive

ufw all SSH_PORT
Rules updated
Rules updated (v6)
```

当然，如果你不喜欢修改 SSH 端口的话，可以直接使用下面的命令。

```bash
ufw disable

ufw reset
ufw default deny incoming
ufw default allow outgoing

ufw allow 22/tcp
ufw allow 80/tcp

ufw enable
```

然后激活防火墙状态。

```bash
ufw enable
Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
Firewall is active and enabled on system startup
```

防火墙启动完毕，可以顺便折腾一下 fail2ban ，大幅减少常规的扫描器对于日志的骚扰，和一些初级的猜解、渗透，后面再写一篇详述。

## Docker 端口绑定和 UFW 的冲突

在配置完毕防火墙后，接下来可以试验配置是否生效，启动一个映射到 80 端口的 `nginx:alpine` 镜像，然后浏览器或者命令行访问服务器公网IP，可以看到熟悉的 nginx 默认欢迎页，ufw 并没有什么作用。

```bash
root@VM-0-10-ubuntu:~# docker run --rm -p 80:80 nginx:alpine
Unable to find image 'nginx:alpine' locally
alpine: Pulling from library/nginx
911c6d0c7995: Pull complete 
131e13eca73f: Pull complete 
95376bf29516: Pull complete 
6717402ec973: Pull complete 
Digest: sha256:23e4dacbc60479fa7f23b3b8e18aad41bd8445706d0538b25ba1d575a6e2410b
Status: Downloaded newer image for nginx:alpine
114.xxx.xxx.xxx - - [27/Aug/2018:16:44:17 +0000] "GET / HTTP/1.1" 200 612 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/68.0.3440.106 Safari/537.36" "-"
2018/08/27 16:44:18 [error] 6#6: *1 open() "/usr/share/nginx/html/favicon.ico" failed (2: No such file or directory), client: 114.xxx.xxx.xxx, server: localhost, request: "GET /favicon.ico HTTP/1.1", host: "119.28.182.48", referrer: "http://119.28.182.48/"
114.xxx.xxx.xxx - - [27/Aug/2018:16:44:18 +0000] "GET /favicon.ico HTTP/1.1" 404 571 "http://119.28.182.48/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/68.0.3440.106 Safari/537.36" "-"
```

这里 ufw 没有生效的原因在于 docker 默认使用了 iptable 添加了一些转发规则，压根没有走到 ufw 的规则中。

一般网上推荐的方案是，关闭这个特性，比如使用类似下面的操作。

```bash
echo '{"iptables": false}' | sudo tee /etc/docker/daemon.json > /dev/null
```

但是如果你真的这样做了，接下来你将无法获得客户端访问时，使用的IP信息，各个容器直接访问互联网、以及容器互通方面也会遇到一些小问题，然后你又不得不添加一些 iptable 去修正这个问题。

如果你还是选择关闭 iptable 特性，执行容器，那么如果你想获取客户端IP，便只能使用以下几个方案达成:

1. 前端启动一个L7的Haproxy / nginx反向代理后面的服务。
2. 运行模式改为 host ，放弃容器完整虚拟化，将端口直接暴露到 `host` 上，此时将不再能够通过 `docker ps` 查看到你的容器端口状态，只能通过 `docker network inspect` 网卡看到对应的端口开启状态，十分不利于维护。
	- 如果你是使用经典的 `docker run` 命令，那么需要配合 `--net=host` 参数。
	- 如果你是使用非 `swarm` 模式的 compose， 则需要声明 `network_mode: "host"`， compose 版本需要声明 3.2 及以上， `port` 导出实际并不需要用繁琐的模式定义。
	- 不用尝试创建自定义网络为 host ，截止本文完成时的编排工具版本以及 docker 版本，这个功能不支持。
3. 修改 ufw 、docker 的 iptable 转发规则，完成你想要的转发方式。

可以看到，不管是哪种方案，搞起来都十分繁琐，而且不利于重复部署，未来调试维护成本太高了。

## 更好的方案

让 Docker 保持默认配置和行为，但是留出端口控制权给 UFW 以及外层的网关，子容器依旧全部使用 `expose` 使用私有化的方法导出端口给网关。

这个方案是不是看起来和上面小节中的方案1很相似，但是其实差别还不小，使用 Traefik 可以在不不配 `consul` / `zk` 的情况下，自动监听 `docker daemon` 的状况，做到服务发现、负载均衡、可用性自动切换、甚至自动绑定域名证书。

之前不得不说是过分追求全容器方案，导致我使用 Traefik 都是在容器中。虽说升级相对轻松，只需要修改 compose 配置中的版本字段即可，程序可用性也不需要太过关注，直接交付给 Linux Daemon 去维护，但是这样就面临一个问题，网关拿到数据的时候，已经经过了至少两块虚拟网卡的转发，一来浪费性能，二来丢失客户端IP，三来如果要保障更高级别的安全，还得关闭 docker iptable 转发的特性，这个面临的问题，上面的小节里说的够多了。

在决定“裸”运行 Traefik 后，我们需要对它的配置进行一定的改动，我这里提供一份最简单的配置，相信已经可以满足许多常见场景。

```Toml
################################################################
# 全局设置
################################################################

# 激活调试模式 （默认关闭）
debug = true

# 日志等级 （默认 ERROR）
logLevel = "INFO"

# 全局入口点类型 （默认 http）
defaultEntryPoints = ["https", "http"]

# 不上报统计信息
sendAnonymousUsage = false

################################################################
# 入口点设置
################################################################

[entryPoints]

    # 默认前端
    [entryPoints.http]
        address = ":80"
        compress = true
        [entryPoints.http.redirect]
            entryPoint = "https"
    [entryPoints.https]
        address = ":443"
        compress = true
    [entryPoints.https.tls]
        [[entryPoints.https.tls.certificates]]
            certFile = "/data/ssl/your.com.cer"
            keyFile = "/data/ssl/your.com.key"

    # 控制台端口
    [entryPoints.traefik-api]
        address = ":4399"

    # Ping端口
    [entryPoints.traefik-ping]
        address = ":4398"


################################################################
# Traefik File configuration
################################################################

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
        entrypoints = ["https"]
        backend = "dashboard"
        [frontends.dashboard.routes.route01]
            rule = "Host:dashboard.your.com"
    [frontends.ping]
        entrypoints = ["https"]
        backend = "ping"
        [frontends.ping.routes.route01]
            rule = "Host:ping.your.com"
        [frontends.ping.routes.route02]
            rule = "ReplacePathRegex: ^/ /ping"

################################################################
# Traefik logs configuration
################################################################

# Traefik logs
# Enabled by default and log to stdout
#
# Optional
#
# Default: os.Stdout
[traefikLog]
  filePath = "/data/logs/traefik.log"

[accessLog]
  filePath = "/data/logs/access.log"

# Format is either "json" or "common".
#
# Optional
# Default: "common"
#
# format = "common"

################################################################
# 访问日志 配置
################################################################

# Enable access logs
# By default it will write to stdout and produce logs in the textual
# Common Log Format (CLF), extended with additional fields.
#
# Optional
#
# [accessLog]

# Sets the file path for the access log. If not specified, stdout will be used.
# Intermediate directories are created if necessary.
#
# Optional
# Default: os.Stdout
#
# filePath = "/path/to/log/log.txt"

# Format is either "json" or "common".
#
# Optional
# Default: "common"
#
# format = "common"

################################################################
# API 及 控制台 配置
################################################################

# 启用API以及控制台
[api]
    # 入口点名称
    entryPoint = "traefik-api"

    # 开启控制台（默认开启）
    dashboard = true

    # 默认协议
    defaultEntryPoints = ["https"]

################################################################
# Ping 配置
################################################################

# 启用 ping
[ping]
    # 入口点名称
    entryPoint = "traefik-ping"

################################################################
# Docker 后端配置
################################################################

# 启用Docker后端
[docker]

# Docker服务后端
endpoint = "unix:///var/run/docker.sock"
# 默认域名
domain = "traefix.your.com"
# 监控docker变化
watch = true

# 使用自定义模板（可选）
# filename = "docker.tmpl"

# 对容器默认进行暴露（默认开启）
#   如果关闭选项，则容器不包含 `traefik.enable=true` 标签，就不会被暴露
exposedbydefault = false

# 使用绑定端口的IP地址取代内部私有网络（默认关闭）
usebindportip = false

# 使用 Swarm Mode （默认关闭）
swarmmode = false

# Enable docker TLS connection.
#
# Optional
#
#  [docker.tls]
#  ca = "/etc/ssl/ca.crt"
#  cert = "/etc/ssl/docker.crt"
#  key = "/etc/ssl/docker.key"
#  insecureskipverify = true
```

将配置做适当修改，保存之后，运行即可：

```bash
traefik -c /etc/traefik.toml
```

当然，此时你是无法访问到你的 Traefik 网关提供的服务的，为什么呢，因为这个软件端口绑定会受限制于 UFW 的规则，所以我们要更新 UFW 规则，允许外网访问我们的 80 和 443 端口。

```bash
root@VM-0-10-ubuntu:~# ufw allow 80
Rule added
Rule added (v6)
```

如果你操作顺利，此刻你已经能够顺利访问你的网站了。

当然，这里少了 docker daemon 的协助，进程管理还是要看护一下的，推荐使用 supervisor 进行辅助管理，之前的博客有介绍过不止一次，有兴趣可以翻阅，这里同样给出一份最基础的配置参考：

```Toml
[program:traefik]
command=traefik -c /etc/traefik.toml --sendAnonymousUsage=false
user=root
autostart=true
startsecs=3
startretries=100
autorestart=true
stderr_logfile=/data/traefik/error.log
stderr_logfile_maxbytes=50MB
stderr_logfile_backups=10
stdout_logfile=/dara/traefik/access.log
stdout_logfile_maxbytes=50MB
stdout_logfile_backups=10
```

你的网关就就绪之后，我们随便找一个目录使用一个叫做 `whoami` 的软件镜像帮助我们验证：网关能够如期的使用，除了自动服务发现，负载解析，还能提供包括统计、转发、header重写等功能。


```yaml
version: '3'

services:
  whoami:
    image: emilevauge/whoami
    expose:
      - 80
    labels:
      - "traefik.enable=true"
      - "traefik.port=80"
      - "traefik.frontend.rule=Host:who.your.com"
```

将上面的配置保存为 `docker-compose.yml`，然后后台运行起来。

浏览器或者命令行访问 `who.your.com`，获得下面的信息：

```text
Hostname: b0cd60b18550
IP: 127.0.0.1
IP: 172.20.0.2
GET / HTTP/1.1
Host: who.your.com
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/68.0.3440.106 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8,ja;q=0.7
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
X-Forwarded-For: 221.xxx.xx.xxx
X-Forwarded-Host: who.your.com
X-Forwarded-Port: 443
X-Forwarded-Proto: https
X-Forwarded-Server: VM-0-10-ubuntu
X-Real-Ip: 221.xxx.xx.xxx
```

可以看到服务发现、SSL证书挂载、源站IP转发等功能都能够正确使用，而且不出意料，QPS 也会高不少（毕竟少了至少一层网络转发、至少一层完整的虚拟化）。


另外，由于没有修改 docker 的配置，容器不会出现不允许访问外网的情况，简单验证：

```bash
root@VM-0-10-ubuntu:/data/who# docker run -it --rm alpine ping -c 1 8.8.8.8
Unable to find image 'alpine:latest' locally
latest: Pulling from library/alpine
8e3ba11ec2a2: Pull complete 
Digest: sha256:7043076348bf5040220df6ad703798fd8593a0918d06d3ce30c6c93be117e430
Status: Downloaded newer image for alpine:latest
PING 8.8.8.8 (8.8.8.8): 56 data bytes
64 bytes from 8.8.8.8: seq=0 ttl=46 time=14.075 ms

--- 8.8.8.8 ping statistics ---
1 packets transmitted, 1 packets received, 0% packet loss
round-trip min/avg/max = 14.075/14.075/14.075 ms
```

接下来就是逐步升级每台服务器以及做数据迁移了。

## 资源链接

其实关于容器内获取外部IP，社区有大量讨论，比如：[^15086](https://github.com/moby/moby/issues/15086#issuecomment-125662376) 等等，涉及不同的网卡模式，不同的编排工具，不同的端口映射模式，又有许多延伸话题。

而 Docker 和 UFW 防火墙的恩怨情仇，其实也是老化长谈，但是不知道为何，网上能看到的资料一边倒到修改 iptable ...

- [Uncomplicated Firewall (UFW) is not blocking anything when using Docker](https://askubuntu.com/questions/652556/uncomplicated-firewall-ufw-is-not-blocking-anything-when-using-docker/652572#652572)
- [Running Docker behind the ufw firewall](https://svenv.nl/unixandlinux/dockerufw/)

## 其他

希望本文能够给你一些额外的启示，帮到正在使用 Traefik 和 Docker 来做服务化的你。



