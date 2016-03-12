# 如何配置 GitLab 使用 HTTPS

本文将聊聊如何在三种场景下，如何正确配置 GitLab ，为用户提供 HTTPS 服务。

为了行文的简单，这里一律使用容器进行搭建配置，如果你是源码、软件包部署，修改对应的文件配置即可。

## 直接使用 GitLab 处理 HTTPS

如果你既不需要统一管理 SSL 证书，又不需要强制流量只从一个网关入口进来，那么直接使用 GitLab 来处理 HTTPS 请求，或许是最好的方案。

这个方案只需要将证书部署到 GitLab 服务器上，然后稍加修改配置即可。

如果使用 `compose` 配置来描述的话，删除掉所有不相关的配置后，涉及到处理 HTTPS 的配置如下（完整配置见[历史文章](https://soulteary.com/2019/04/10/gitlab-was-built-with-docker-and-traefik-part-1.html)、更多相关内容可以浏览 [GitLab 标签](https://soulteary.com/tags/gitlab.html)）：

```yaml
version: '3'

services:

  gitlab:
    image: 'gitlab/gitlab-ce:12.0.2-ce.0'
    hostname: 'gitlab.lab.com'
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - './cert/lab.com.crt:/etc/gitlab/ssl/lab.com.crt:ro'
      - './cert/lab.com.key:/etc/gitlab/ssl/lab.com.key:ro'
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'https://gitlab.lab.com'
        nginx['enable'] = true
        nginx['client_max_body_size'] = '250m'
        nginx['redirect_http_to_https'] = true
        nginx['redirect_http_to_https_port'] = 80
        nginx['ssl_certificate'] = "/etc/gitlab/ssl/lab.com.crt"
        nginx['ssl_certificate_key'] = "/etc/gitlab/ssl/lab.com.key"
        nginx['ssl_ciphers'] = "ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-GCM-SHA256"
        nginx['ssl_prefer_server_ciphers'] = "on"
        nginx['ssl_protocols'] = "TLSv1.2"
        nginx['http2_enabled'] = true
        nginx['proxy_set_headers'] = {
          "X-Forwarded-Proto" => "http"
        }
```

因为使用 GitLab 处理 HTTP/HTTPS 流量，所以需要开放 `80` 和 `443` 端口。

正确配置端口之后，最关键的配置是 `external_url`。

配置内容中需要包含 `https` 协议头，另外在 `nginx['ssl_certificate']` 和 `nginx['ssl_certificate_key']` 配置项中，需要填写正确的证书路径。

## 使用其他软件来处理 HTTPS

这里主要有两种场景，第一种是使用 Traefik 之类的代理软件，另一种则是使用 云主机的 SLB 服务。

不论是出于想统一管理证书，还是减少暴露在外的公开端口，流量经过统一入口转发到具体应用之上，都可以使用下面的方案来进行操作。

先聊聊使用 Traefik 作为网关的场景。

### 使用 Traefik 作为网关

`traefik.toml` 配置文件中涉及 HTTP 流量处理的配置主要是这部分内容：

```Toml
defaultEntryPoints = ["http", "https"]

[entryPoints]
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
            certFile = "/data/ssl/lab.com.pem"
            keyFile = "/data/ssl/lab.com.key"
```

和上一小节一样，删除掉所有不相关的配置后，核心配置如下：

```yaml
version: '3'

services:

  gitlab:
    image: gitlab/gitlab-ce:12.1.6-ce.0
    hostname: 'gitlab.lab.com'
    expose:
      - 80
    labels:
      - "traefik.enable=true"
      - "traefik.gitlab.port=80"
      - "traefik.gitlab.frontend.rule=Host:gitlab.lab.com"
      - "traefik.gitlab.frontend.entryPoints=http,https"
      - "traefik.gitlab.frontend.headers.SSLProxyHeaders=X-Forwarded-For:https"
      - "traefik.gitlab.frontend.headers.STSSeconds=315360000"
      - "traefik.gitlab.frontend.headers.browserXSSFilter=true"
      - "traefik.gitlab.frontend.headers.contentTypeNosniff=true"
      - "traefik.gitlab.frontend.headers.customrequestheaders=X-Forwarded-Ssl:on"
      - "traefik.gitlab.frontend.passHostHeader=true"
      - "traefik.gitlab.frontend.passTLSCert=false"
    networks:
      - traefik
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'https://gitlab.lab.com'
        nginx['enable'] = true
        nginx['listen_port'] = 80
        nginx['listen_https'] = false
        nginx['http2_enabled'] = false
        nginx['redirect_http_to_https'] = true

networks:
  traefik:
    external: true
```

因为使用 Traefik 处理 HTTP/HTTPS 流量，所以 GitLab 只需要开放 80 端口即可，但是需要在 `label` 中定义服务发现的各种规则。

同样的，这里的核心配置是 `external_url` 和 `nginx['listen_https']`，前者依旧要保持有 `https` 协议，但是后续则需要配置为 `false`。

### 使用 SLB 作为网关

如果要使用云服务商的 SLB 来管理 HTTPS 流量和证书，那么上面的配置可以再简化一些：

```yaml
version: '3'

services:

  gitlab:
    image: gitlab/gitlab-ce:12.1.6-ce.0
    hostname: 'gitlab.lab.com'
    ports:
      - 80:80
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'https://gitlab.lab.com'
        nginx['enable'] = true
        nginx['listen_port'] = 80
        nginx['listen_https'] = false
        nginx['proxy_set_headers'] = {
          "Host" => "$$http_host",
          "X-Real-IP" => "$$remote_addr",
          "X-Forwarded-For" => "$$proxy_add_x_forwarded_for",
          "X-Forwarded-Proto" => "http"
        }

networks:
  traefik:
    external: true
```

在删除所有 `labels` 内容后，GitLab 还不能够正常运行，我们必须再设置 `nginx['proxy_set_headers']` ，配置 `"X-Forwarded-Proto" => "http"` ，让 GitLab 接受流量的时候，返回给代理软件正确的响应。

## 最后

这次就先折腾到这里，等项目上线后，再聊聊如何更高效的使用 `GitLab`。

—EOF

