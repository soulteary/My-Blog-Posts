# 使用 Apache 搭建 VPC 服务器代理

有的小伙伴或许没有使用过 VPC 网络下的服务器，在该网络环境下，服务器默认没有公网 IP ，所以用户无法访问到服务器。一般策略是使用 SLB 进行入网流量代理，这样用户就能从公网访问服务器上的应用了。

但是这样只能解决流量进入的问题，并解决不了 VPC 环境下的内网机器访问公网资源的问题，给每一台机器单独分配 IP 显然不是最优解，这时我们一般会选择使用某一台服务器作为出口，搭建代理服务器。

## 使用容器配置 Apache 代理服务器

为内网环境服务器搭建代理服务器，我们一般会优先选择 [Apache Traffic Server](https://trafficserver.apache.org/) ，但是其实使用 `Apache` 也可以简单的解决问题。

相比较 Traffic Server，使用 Apache 作为代理服务器非常简单。容器编排文件 `docker-compose.yml` 只需要 22 行：

```yaml
version: "3.6"

services:

  proxy:
    image: httpd:2.4.39-alpine
    restart: always
    container_name: network-proxy
    ports:
      - 1080:80
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /etc/timezone:/etc/timezone:ro
      - ./httpd.conf:/usr/local/apache2/conf/httpd.conf
    healthcheck:
      test: ["CMD-SHELL", "httpd -T"]
      interval: 5s
      retries: 12
    logging:
        driver: "json-file"
        options:
            max-size: "10m"
```

Apache 配置文件 `httpd.conf`也无需像网上配置那么复杂，只需要下面这30来行就行：

```TeXT
ServerName localhost
Listen 80

LoadModule mpm_event_module modules/mod_mpm_event.so
LoadModule authz_core_module modules/mod_authz_core.so
LoadModule access_compat_module modules/mod_access_compat.so
LoadModule log_config_module modules/mod_log_config.so
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_connect_module modules/mod_proxy_connect.so
LoadModule proxy_http_module modules/mod_proxy_http.so
LoadModule proxy_http2_module modules/mod_proxy_http2.so
LoadModule unixd_module modules/mod_unixd.so

User daemon
Group daemon

ErrorLog /proc/self/fd/2
LogLevel warn

LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined
LogFormat "%h %l %u %t \"%r\" %>s %b" common
CustomLog /proc/self/fd/1 common

ProxyRequests On
ProxyVia On

<Proxy *>
    Order deny,allow
    Deny from all
    Allow from 192.168.0.0/24
</Proxy>
```

如果你和我一样，明确代理服务器的服务目标，可以在 `<Proxy>` 配置中将其声明，避免服务被盗用，当然，推荐搭配防火墙安全策略一起使用，万无一失。

使用 `docker-compose up` 启动应用，会看到类似下面的日志：

```TeXT
network-proxy | [Sat Aug 10 15:32:06.652264 2019] [mpm_event:notice] [pid 1:tid 140135351733576] AH00489: Apache/2.4.39 (Unix) configured -- resuming normal operations
network-proxy | [Sat Aug 10 15:32:06.652318 2019] [core:notice] [pid 1:tid 140135351733576] AH00094: Command line: 'httpd -D FOREGROUND'
```

看日志服务确实是启动起来了，但是是否有效不得而知，所以我们要进行测试。

### 测试服务

在另外一台服务器上使用 curl 测试代理服务器是否正常工作，如果能否正常使用，结果会类似下面这样：

```bash
# http_proxy=http://192.168.0.50:1080 curl http://cip.cc/
IP	: 39.xxx.xxx.xxx
地址	: 中国  北京
运营商	: 阿里云/电信/联通/移动/铁通/教育网
数据二	: 香港 | 特别行政区
数据三	: 中国北京北京市 | 阿里云

URL	: http://www.cip.cc/39.xxx.xxx.xxx
```

## 配置服务器

让服务器默认出公网的流量走代理服务器很简单，只需要在 `/etc/profile` 配置文件中添加两行即可：

```bash
export http_proxy=http://192.168.0.50:1080
export https_proxy=http://192.168.0.50:1080
```

对 profile 文件进行修改后，需要手动重载文件：

```bash
source /etc/profile
```

或者断开当前的终端连接，重新连接服务器，也可以让配置生效。再次使用 curl 对代理服务器进行验证，会看到默认出公网的流量会先经过代理服务器。

```bash
# curl -v https://www.baidu.com
* Rebuilt URL to: https://www.baidu.com/
*   Trying 192.168.0.50...
* TCP_NODELAY set
* Connected to 192.168.0.50 (192.168.0.50) port 1080 (#0)
* allocate connect buffer!
* Establish HTTP proxy tunnel to www.baidu.com:443
> CONNECT www.baidu.com:443 HTTP/1.1
> Host: www.baidu.com:443
> User-Agent: curl/7.58.0
> Proxy-Connection: Keep-Alive
>
< HTTP/1.0 200 Connection Established
< Proxy-agent: Apache/2.4.39 (Unix)
<
* Proxy replied 200 to CONNECT request
* CONNECT phase completed!
* ALPN, offering h2
* ALPN, offering http/1.1
```

## 配置容器服务

Docker 官方文档中[有提过](https://docs.docker.com/config/daemon/systemd/#httphttps-proxy) ，如果想要 Docker Daemon 使用系统代理配置，需要在其启动之前进行配置，所以配置 `daemon.json` 大法在此处就不适用啦。

解决方法是覆盖默认的 `docker.service` 配置文件，先创建一个服务配置目录：

```bash
sudo mkdir -p /etc/systemd/system/docker.service.d
```

然后创建一个文件并编辑文件内容 `/etc/systemd/system/docker.service.d/http-proxy.conf` ，添加环境变量：

```bash
[Service]

Environment="HTTP_PROXY=http://192.168.0.50:1080"
Environment="HTTPS_PROXY=http://192.168.0.50:1080"
Environment="NO_PROXY=localhost,127.0.0.1,192.168.0.0/24,*.domain.ltd"
```

接着重启服务：

```bash
sudo systemctl daemon-reload && sudo systemctl restart docker
```

最后，使用 `docker pull` 命令验证配置是否正常：

```bash
# docker pull alpine
Using default tag: latest
latest: Pulling from library/alpine
050382585609: Already exists
Digest: sha256:6a92cd1fcdc8d8cdec60f33dda4db2cb1fcdcacf3410a8e05b3741f44a9b5998
Status: Downloaded newer image for alpine:latest
docker.io/library/alpine:latest
```

## 配置容器内部环境

如果不进行容器内部网络配置，使用容器访问公网服务，基本会遇到网络超时：

```bash
docker run --rm -it alpine

/ # apk update
fetch http://dl-cdn.alpinelinux.org/alpine/v3.10/main/x86_64/APKINDEX.tar.gz
ERROR: http://dl-cdn.alpinelinux.org/alpine/v3.10/main: network error (check Internet connection and firewall)
WARNING: Ignoring APKINDEX.00740ba1.tar.gz: No such file or directory
```

Docker 官方文档其实也[有提过 ](https://docs.docker.com/network/proxy/#use-environment-variables)，解决方案的原理是：通过编辑 `~/.docker/config.json`  Docker 客户端配置文件，来为容器自动注入 PROXY 环境变量。

```TeXT
{
    "proxies": {
        "default": {
            "httpProxy": "http://192.168.0.50:1080",
            "httpsProxy": "http://192.168.0.50:1080",
            "noProxy": "127.0.0.1,localhost,192.168.0.0/24,*.domain.ltd"
        }
    }
}
```

将上述配置添加好之后，无须重启容器服务，直接再次执行命令即可：

```bash
# docker run --rm -it alpine

/ # apk update
fetch http://dl-cdn.alpinelinux.org/alpine/v3.10/main/x86_64/APKINDEX.tar.gz
fetch http://dl-cdn.alpinelinux.org/alpine/v3.10/community/x86_64/APKINDEX.tar.gz
v3.10.1-62-g89778c626e [http://dl-cdn.alpinelinux.org/alpine/v3.10/main]
v3.10.1-60-gb0081284ea [http://dl-cdn.alpinelinux.org/alpine/v3.10/community]
OK: 10337 distinct packages available
```

至此，VPC 环境下的服务器和容器访问外网就设置完毕啦。

## 最后

别忘记设置防火墙规则，服务器访问公网的 IP 不允许入网流量，减少服务器对外安全隐患。

—EOF