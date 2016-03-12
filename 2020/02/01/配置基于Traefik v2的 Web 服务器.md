# 配置基于Traefik v2的 Web 服务器

以往聊到 Web 服务器，我们通常会使用 Nginx、Apache，或者后起之秀 Caddy，本文将继续介绍一种相对小众但是好用的 Web 服务软件：Traefik。

本文先聊聊传统架构场景下的使用，云服务架构场景晚些时候有空再写。

## 写在前面

如果你使用的是 SLB + VPC 的架构，那么使用[《Traefik 2 使用指南，愉悦的开发体验》](https://soulteary.com/2020/01/28/traefik-2-user-guide-pleasant-development-experience.html)  中的容器方案会更利于维护。

如果你使用的是传统的单体 VPS 架构，服务器前缺少云平台的负载均衡网关，那么就可以使用 Traefik 直接作为服务网关，在保证高性能转发、无感知重载、动态加载SSL证书等能力外，还提供了一定的可视化能力，解决了日常开发调试中“盲人摸象”的问题。

然而即使是使用传统的 VPS 架构，在 Traefik 和 Docker 容器的加持下，也可以发挥出不错的性能和便捷的开发能力。

## 配置系统

作为 Web Server 需要配置的东西不多，根据情况简单提升下文件打开数目：

```bash
echo "* soft nofile 51200">>/etc/security/limits.conf
echo "* hard nofile 51200">>/etc/security/limits.conf
echo "session required pam_limits.so">>/etc/pam.d/common-session
echo "ulimit -SHn 51200">>/etc/profile
```

SSH 免密登录等可参考[《Ubuntu 18.04 基础系统配置》](https://soulteary.com/2019/04/06/configure-ubuntu-18-04.html)、[《配置Ubuntu WebServer基础环境》](https://soulteary.com/2015/01/24/configure-ubuntu-for-web-server.html)，这里不做赘述。

接着配置安装 Docker 容器运行环境：

```bash
apt update && apt upgrade -y
apt install -y apt-transport-https  ca-certificates curl software-properties-common
 
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
apt-key fingerprint 0EBFCD88
add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
apt install -y docker-ce
 
curl -L https://github.com/docker/compose/releases/download/1.25.3/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

如果有挂载云储存（云硬盘、NAS），可以参考 [《迁移 Docker 容器储存位置 》](https://soulteary.com/2019/07/14/migrate-docker-container-storage-location.html)、[《Ubuntu 18.04 基础系统配置》](https://soulteary.com/2019/04/06/configure-ubuntu-18-04.html) 中的内容进行磁盘挂载、内容迁移。

因为引用了两篇内容，这里做一些概括性描述怎么操作。

格式化并挂载磁盘到 `/data` 可以参考使用下面的命令：

```bash
fdisk -u /dev/vdb
# 交互式输入：p->n->p->enter->enter->enter->w
mkfs.ext4 /dev/vdb1
echo /dev/vdb1 /data ext4 defaults 0 0 >> /etc/fstab
```

迁移容器数据内容到新的目录位置可以参考下面的命令：

```bash
service docker stop
mkdir -p /data/docker/lib
rsync -avz /var/lib/docker /data/docker/lib
 
 
vi /etc/docker/daemon.json
{
    "data-root": "/data/docker/lib"
}
service docker start
```

## Traefik 作为服务网关

作为服务网关，更偏重于性能和稳定性，考虑容器化会降低一部分性能，这里推荐直接使用对应 Linux 发行版的二进制应用文件直接启动应用。

### 安装应用

安装 Traefik 很容易，只需要下载二进制文件，就好了，像是下面这样。

```bash
wget https://github.com/containous/traefik/releases/download/v2.1.3/traefik_v2.1.3_linux_amd64.tar.gz
tar zxvf traefik_v2.1.3_linux_amd64.tar.gz
mv traefik /usr/bin/
```

验证是否安装就绪，可以使用 `command` 或者 `which` 来进行验证系统是否找的着软件或者执行路径是什么，再或者使用 `traefik version` 来看看当前应用的版本也行。

```bash
which traefik
command -v traefik
```

如果显示的结果都是 `/usr/bin/traefik` 就说明安装成功。本文使用的 Traefik 版本信息如下：

```bash
traefik version

Version:      2.1.3
Codename:     cantal
Go version:   go1.13.6
Built:        2020-01-21T17:30:29Z
OS/Arch:      linux/amd64
```

### 配置应用

在书写应用配置前，需要先准备应用配置目录。

```bash
mkdir -p /data/basic/traefik/{logs,conf}
```

参考[《Traefik 2 使用指南，愉悦的开发体验》](https://soulteary.com/2020/01/28/traefik-2-user-guide-pleasant-development-experience.html)  一文中的内容，很容易写出一个简单的 traefik.toml 配置：

```bash
[global]
  checkNewVersion = false
  sendAnonymousUsage = false

[log]
  level = "INFO"
  format = "common"
  filePath = "/data/basic/traefik/logs/traefik.log"

[api]
  dashboard = true
  insecure = true
  debug = false

[ping]

[accessLog]
  filePath = "/data/basic/traefik/logs/access.log"
  bufferingSize = 100

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
    directory = "/data/basic/traefik/conf"
    debugLogGeneratedTemplate = true

[entryPoints]
  [entryPoints.http]
    address = ":80"
  [entryPoints.https]
    address = ":443"

```

将上面的内容保存到 `/data/basic/traefik/traefik.toml` 后，启动应用，并访问你的服务器 IP 进行验证。

```bash
traefik --configFile /data/basic/traefik/traefik.toml

INFO[0000] Configuration loaded from file: /data/basic/traefik/traefik.toml 
```

当你在浏览器中看到熟悉的 `404 page not found` 的时候，说明基础配置就完成了，如果你想配置 dashboard ，可以参考[《Traefik 2 使用指南，愉悦的开发体验》](https://soulteary.com/2020/01/28/traefik-2-user-guide-pleasant-development-experience.html) 文中的其他配置。

### 配置进程守护服务

即使软件通过了编译测试、功能测试，实际运行时，还是可能遇到极端情况，导致软件中止运行，所以我们需要安装进程守护服务，对应用进行“保活”。

在容器方案里，只需要很简单的一句“restart: always”配置即可，但是退化到传统服务器方案中，这个事情就变的稍微麻烦了一些。

这里推荐老牌应用 supervisor，在 debian / ubuntu 系统中安装只需要一句话：

```bash
apt-get install -y supervisor
```

等待安装完毕，准备编写 Traefik 的进程守护配置文件：

```bash
[program:traefik]
command=traefik --configFile /data/basic/traefik/traefik.toml
user=root
autostart=true
startsecs=3
startretries=100
autorestart=true
stderr_logfile=/data/basic/traefik/logs/supervisor-traefik-error.log
stderr_logfile_maxbytes=50MB
stderr_logfile_backups=10
stdout_logfile=/data/basic/traefik/logs/supervisor-traefik-access.log
stdout_logfile_maxbytes=50MB
stdout_logfile_backups=10
```

将上面的内容保存到 `/etc/supervisor/conf.d/traefik.conf` 中，重启进程守护服务，查看系统进程，会发现 traefik 已经启动起来了。

```bash
service supervisor restart

ps -ef | grep traefik

root      1318  1285  2 21:38 ?        00:00:01 traefik --configFile /data/basic/traefik/traefik.toml
```

## 自动申请证书

作为服务网关，Traefik 还有一个令人满意的功能，就是“根据所启动的服务”，动态的自动申请并续签泛域名证书。

首先需要准备储存申请证书的目录：

```bash
mkdir -p /data/basic/traefik/ssl/
```

接着在 Traefik 主配置中添加 ACME 证书申请配置：

```bash
[certificatesResolvers.le.acme]
  email = "yourmail@company.com"
  storage = "/data/basic/traefik/ssl/acme.json"
  [certificatesResolvers.le.acme.dnsChallenge]
    resolvers = ["1.1.1.1:53", "8.8.8.8:53"]
    provider = "cloudflare"
    delayBeforeCheck = 30
```

参考官方配置[Traefik 使用DNS 验证方式申请证书](https://docs.traefik.io/https/acme/#dnschallenge)，我这边选择 Cloudflare 作为服务商，根据 [https://go-acme.github.io/lego/dns/cloudflare/](https://go-acme.github.io/lego/dns/cloudflare/) 所需配置的变量有三个（不推荐使用 `CF_API_KEY`）。

```text
# CloudFlare 账户邮箱
CF_API_EMAIL
# API token 需要 DNS:Edit 权限
CLOUDFLARE_DNS_API_TOKEN
# API token 需要 Zone:Read 权限
CLOUDFLARE_ZONE_API_TOKEN	
```

因为使用 supervisor ，所以这里设置变量需要写在  `/etc/supervisor/conf.d/traefik.conf` 中，并使用 `Key="value"` 的格式：

```bash
[program:traefik]
command=traefik --configFile /data/basic/traefik/traefik.toml
user=root
environment=CF_API_EMAIL="yourmail@company.com",CLOUDFLARE_DNS_API_TOKEN="token1",CLOUDFLARE_ZONE_API_TOKEN="token2"
autostart=true
...
```

然后重启守护服务。

```bash
service supervisor stop && service supervisor start
```

接下来，随便启动一个服务，然后稍等片刻，检查证书储存文件（`/data/basic/traefik/ssl/acme.json`），顺利的话，会看到申请好的 SSL 证书。

## 验证 Web 应用

想要验证服务的基础功能是否好用，只需要随便启动一个应用，并声明它所使用的域名即可，下面是 docker-compose.yml 配置文件内容：

```yaml
version: '3'

services:
  whoami:
    image: soulteary/whoami
    networks:
      - traefik
    labels:
      - "traefik.enable=true"
      - "traefik.docker.network=traefik"
      - "traefik.http.routers.whoami0.middlewares=https-redirect@file"
      - "traefik.http.routers.whoami0.entrypoints=http"
      - "traefik.http.routers.whoami0.rule=Host(`whoami.soulteary.com`)"
      - "traefik.http.routers.whoami1.middlewares=content-compress@file"
      - "traefik.http.routers.whoami1.entrypoints=https"
      - "traefik.http.routers.whoami1.tls=true"
      - "traefik.http.routers.whoami1.tls.certresolver=le"
      - "traefik.http.routers.whoami1.tls.domains[0].main=soulteary.com"
      - "traefik.http.routers.whoami1.tls.domains[0].sans=*.soulteary.com"
      - "traefik.http.routers.whoami1.rule=Host(`whoami.soulteary.com`)"
      - "traefik.http.services.whoamibackend.loadbalancer.server.scheme=http"
      - "traefik.http.services.whoamibackend.loadbalancer.server.port=80"
networks:
  traefik:
    external: true
```

将内容保存完毕，使用 `docker-compose up -d`  将应用启动之后，稍等片刻，访问域名，会看到应用已经就绪，现证书也已经自动申请完毕。

## 其他

去年写了不少有关[traefik](https://soulteary.com//tags/traefik.html) 的文章，接下来我会找时间把这些文章做一遍升级。我个人和线上生产环境都已经使用了一年多 Traefik ，恰逢 Traefik 版本升级，记录一些实践笔记，希望能帮助到有需要的同学。

--EOF