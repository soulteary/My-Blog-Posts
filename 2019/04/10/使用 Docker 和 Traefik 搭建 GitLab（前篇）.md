# 使用 Docker 和 Traefik 搭建 GitLab （前篇）

[之前](https://soulteary.com/tags/gitlab.html)曾不止一次的介绍过 GitLab 在容器中的安装使用。考虑到多数使用场景都是在内网环境下，所以也未曾过多的进行过安全配置。最近在帮研究院进行系统搭建，其中一个述求是“公网环境下使用”。

本篇将介绍如何更好的使用容器中的 GitLab ，并搭配 Traefik 实现自动挂载 HTTPS 。

## 编写 Traefik 配置规则

Traefik 的详细使用，可以参考以往的文章，比如：[使用服务发现改善开发体验](https://soulteary.com/2018/06/11/use-server-side-discovery-improve-development.html)、[更完善的 Docker + Traefik 使用方案](https://soulteary.com/2018/08/28/better-use-of-docker-and-traefik.html) 等，更多内容可以翻看历史[内容标签](https://soulteary.com/tags/traefik.html)，这里不过多赘述。

本文依旧只需要关注编排文件中的 `labels` 和 `networks` 字段配置就足够啦。

对 GitLab 容器服务的 `networks` 字段设置全局使用的网卡 `traefik`（本例），就可以**让 Traefik 自动接管 GitLab 对外的 Web 服务请求**。

```yaml
networks:
  - traefik
```

编排文件中的 `labels` 字段，声明了 Traefik 如何对流量进行转发。假设我们要对外提供三种访问能力：

- `https://gitlab.${DOMAIN}`
	- 访问 GitLab Web 页面和 Web API
- `https://registry.${DOMAIN}`
	- 使用终端访问容器仓库
- `https://page.${DOMAIN}`
	- 使用浏览器访问一些仓库的预览页面（类似 GitHub Page）

那么我们可以这样配置：

```yaml
labels:
  - "traefik.enable=true"
  # GitLab Web 服务
  - "traefik.gitlab.port=80"
  - "traefik.gitlab.frontend.rule=Host:gitlab.${BASEHOST}"
  - "traefik.gitlab.frontend.entryPoints=http,https"
  - "traefik.gitlab.frontend.headers.SSLProxyHeaders=X-Forwarded-For:https"
  - "traefik.gitlab.frontend.headers.STSSeconds=315360000"
  - "traefik.gitlab.frontend.headers.browserXSSFilter=true"
  - "traefik.gitlab.frontend.headers.contentTypeNosniff=true"
  - "traefik.gitlab.frontend.headers.customrequestheaders=X-Forwarded-Ssl:on"
  - "traefik.gitlab.frontend.passHostHeader=true"
  - "traefik.gitlab.frontend.passTLSCert=false"
  # Registry 服务
  - "traefik.registry.port=5100"
  - "traefik.registry.frontend.rule=Host:registry.${BASEHOST}"
  - "traefik.registry.frontend.entryPoints=http,https"
  # Pages 服务
  - "traefik.pages.port=5201"
  - "traefik.pages.frontend.rule=Host:page.${BASEHOST}"
  - "traefik.pages.frontend.entryPoints=http,https"
```

`Registry`、`Pages` 这里，为了节约篇幅，我就不重复粘贴相同的内容了，你可以参考 `GitLab Web 服务`补全响应头处理。

当然，如果你觉得容器编排文件写的内容太多了，想放到 GitLab 中进行处理也是可以的，稍后我会讲。

## 编写 GitLab 配置

配置 GitLab 还是需要一些额外的耐心，不过好在坑我都替你趟完了。

### 配置 GitLab Nginx 服务

在给出参考代码之前，我们需要先知道 GitLab 的一个“Tricks”：

如果你设置的 `external_url` 内容包含 `https`，那么服务默认会使用 SSL 方式对外提供服务。如果你的 `external_url` 声明为 `http://gitlab.${DOMAIN}` ，当系统运行起来后（默认端口为 `80`），当我们使用 `https` 进行访问，又会出现各种问题，[官方文档](https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/doc/settings/nginx.md#supporting-proxied-ssl)写的也是不清不楚。

搜索 `reddit` 上的讨论历史，发现有个老外都自暴自弃放弃使用  `https`，改用 `http`了，令人哭笑不得。

**遇到问题解决问题就好了鸭，逃避是什么鬼。**

使用容器方式搭建 GitLab ，所有的配置都需要声明在编排文件的 `environment`字段内，下面是 GitLab Web 服务的使用配置。

```yaml
environment:
  GITLAB_OMNIBUS_CONFIG: |
    external_url 'https://gitlab.${BASEHOST}'
    nginx['enable'] = true
    nginx['listen_port'] = 80
    nginx['listen_https'] = false
    nginx['http2_enabled'] = false
    nginx['client_max_body_size'] = '250m'
    nginx['redirect_http_to_https'] = true
```

需要注意的是，声明的外部链接地址 `external_url` 需要使用 `HTTPS` 协议。而监听端口需要设置为 `80`，另外也要配置Nginx不进行 `https` 监听，不使用 `HTTP2`，至于 `HTTP` 自动转向 `HTTPS` 可配可不配，因为 Traefik 侧我默认开启了 `HTTP` 转向 `HTTPS` 功能。

前文提到，如果我们不想使用 Traefik 进行响应头的修改，那么该如何在 GitLab 中进行配置呢，也很简单，多添加一个 `proxy_set_headers` 的配置即可：

```yaml
nginx['proxy_set_headers'] = {
  "Host" => "$$http_host",
  "X-Real-IP" => "$$remote_addr",
  "X-Forwarded-For" => "$$proxy_add_x_forwarded_for",
  "X-Forwarded-Proto" => "https",
  "X-Forwarded-Ssl" => "on"
}
```

这里有一点需要额外注意，所有出现在 `environment` 字段内的变量，都需要使用**双$**符号声明，而非 **单个$**，否则结果不会如愿以偿。

前端提到了，我们要同时提供 **Web 访问**、**容器仓库**、**页面预览**三个功能，所以配置还需要加上其他两项。

```yaml
registry['enable'] = true
registry_external_url 'https://registry.${BASEHOST}/'
registry_nginx['listen_port'] = 5100
registry_nginx['listen_https'] = false
registry_nginx['redirect_http_to_https'] = true

pages_external_url "https://page.${BASEHOST}/"
pages_nginx['listen_port'] = 5200
pages_nginx['listen_https'] = false
pages_nginx['redirect_http_to_https'] = true

gitlab_pages['enable'] = true
gitlab_pages['inplace_chroot'] = true
gitlab_pages['external_http'] = ['gitlab:5201']
gitlab_pages['dir'] = "/var/opt/gitlab/gitlab-pages"
gitlab_pages['log_directory'] = "/var/log/gitlab/gitlab-pages"
gitlab_pages['artifacts_server'] = true
gitlab_pages['artifacts_server_timeout'] = 10
```

可以看到，除了 `gitlab_pages` 字段比较特殊外，配置和刚刚大同小异。

另外提一点，我原本的习惯是将所有的流量都配置到 `80` 端口，再让 Traefik 进行转发可读性会更好一些，但是看到了另外一位[国外同学的配置](https://www.davd.io/byecloud-gitlab-with-docker-and-traefik/)后，我觉得让端口保持在默认端口也是不错的选择，比如 `5100`、`5200`。

配置文件最上面的监控需要额外配置，我们回头细聊，为了减少影响，这里将这部分功能进行关闭。

### 配置 GitLab SSH 端口

这里我选择让 GitLab 的 `SSH` 端口保持默认，而修改宿主机的 `SSH` 端口到其他位置，这样做的好处是：

- 可以减少对 GitLab 的配置。
- 仓库访问地址显得更美观了，避免了用户使用软件过程中需要解决的额外问题。

使用编排文件，将 GitLab 端口映射到宿主机中。

```yaml
version: '3'

services:

  gitlab:
    expose:
      - 80
      - 443
    ports:
      - '0.0.0.0:22:22'
```

这里有一个小细节，如果你不在 `labels` 中对你的服务端口进行声明，Traefik 会使用你暴露的第一个端口作为服务发现的端口。所以将你所有依赖的内容都显式声明，是一个好的习惯。

## 完整的配置文件

比较重要的细节都讲完了，这里给出完整的配置参考（容器仓库和页面预览服务的响应头有删减，有需求可以自行添加）：

```yaml
version: '3'

services:

  gitlab:
    restart: always
    image: ${GITLAB_IMAGE}
    hostname: ${HOSTNAME}
    healthcheck:
      disable: true
    expose:
      - 80
      - 443
    ports:
      - '0.0.0.0:22:22'
    # 解决搜索引擎搜索不出小于3字符问题
    # https://gitbaai.ac.cn/gitlab-org/gitlab-ce/issues/40379
    entrypoint: |
      bash -c 'sed -i "s/MIN_CHARS_FOR_PARTIAL_MATCHING = 3/MIN_CHARS_FOR_PARTIAL_MATCHING = 1/g" /opt/gitlab/embedded/service/gitlab-rails/lib/gitlab/sql/pattern.rb && /assets/wrapper'
    volumes:
      - ./config:/etc/gitlab
      - ./data:/var/opt/gitlab
      - ./logs:/var/log/gitlab
      - ./embedded-logs:/opt/gitlab/embedded/logs/
    labels:
      - "traefik.enable=true"
      # GitLab Web 服务
      - "traefik.gitlab.port=80"
      - "traefik.gitlab.frontend.rule=Host:gitlab.${BASEHOST}"
      - "traefik.gitlab.frontend.entryPoints=http,https"
      - "traefik.gitlab.frontend.headers.SSLProxyHeaders=X-Forwarded-For:https"
      - "traefik.gitlab.frontend.headers.STSSeconds=315360000"
      - "traefik.gitlab.frontend.headers.browserXSSFilter=true"
      - "traefik.gitlab.frontend.headers.contentTypeNosniff=true"
      - "traefik.gitlab.frontend.headers.customrequestheaders=X-Forwarded-Ssl:on"
      - "traefik.gitlab.frontend.passHostHeader=true"
      - "traefik.gitlab.frontend.passTLSCert=false"
      # Registry 服务
      - "traefik.registry.port=5100"
      - "traefik.registry.frontend.rule=Host:registry.${BASEHOST}"
      - "traefik.registry.frontend.entryPoints=http,https"
      # Pages 服务
      - "traefik.pages.port=5201"
      - "traefik.pages.frontend.rule=Host:page.${BASEHOST}"
      - "traefik.pages.frontend.entryPoints=http,https"
    networks:
      - traefik
networks:
  traefik:
    external: true
```

光有编排配置，不能够愉快使用，这里还需要创建一个 `.env` 环境配置文件：

```TeXT
GITLAB_IMAGE=gitlab/gitlab-ce:11.8.6-ce.0
BASEHOST=lab.com
HOSTNAME=gitlab.lab.com
```

两个配置文件都准备好之后，使用 `docker-compose up` 启动你的应用，然后就可以开始使用了。

如果你还不熟悉 `docker-compose` 的使用，可以翻阅[之前的文章](https://soulteary.com/2019/04/07/use-docker-and-traefik-to-build-wordpress-with-nginx.html)，查阅 “一些额外的小技巧”一节。

## 最后

下一篇，我将着重介绍一些安全配置上的问题。