# 使用 Docker 和 Traefik v2 搭建 RSS 服务（Miniflux）

之前提过，接下来要针对去年的老文章，聊聊如何升级老应用。本篇将以 RSS 服务为例，简单聊聊如何操作。

<!-- more -->

## 写在前面

去年写过三篇使用 Docker 搭建 RSS 服务的文章，适逢升级 Traefik ，暂以下面搭建 RSS 服务文章的第一篇为例，聊聊在 2020 年初，如何升级这类应用/服务：

- 使用 Docker 搭建你自己的 RSS 服务（Miniflux）：[https://soulteary.com/2019/01/22/build-your-own-rss-service-with-docker-miniflux.html](https://soulteary.com/2019/01/22/build-your-own-rss-service-with-docker-miniflux.html)
- 使用 Docker 搭建你自己的 RSS 服务（stringer）：[https://soulteary.com/2019/01/06/build-your-own-rss-service-with-docker-stringer.html](https://soulteary.com/2019/01/06/build-your-own-rss-service-with-docker-stringer.html)
- 使用 Docker 搭建你自己的 RSS 服务（FreshRSS）：[https://soulteary.com/2019/01/05/build-your-own-rss-service-with-docker-freshrss.html](https://soulteary.com/2019/01/05/build-your-own-rss-service-with-docker-freshrss.html)

如果你还不太了解 Traefik，可以参考 [《Traefik 2 使用指南，愉悦的开发体验 》](https://soulteary.com/2020/01/28/traefik-2-user-guide-pleasant-development-experience.html)、[配置基于Traefik v2的 Web 服务器 ](https://soulteary.com/2020/02/01/configure-traefik-v2-based-web-server.html) 先行了解掌握 Traefik v2 相关的知识。

如果你之前已经使用了这些服务，请做好数据备份操作，再跟随文章进行升级，如果你是新用户，那么就可以忽略这些问题，大胆放心的搞起啦。

## Miniflux 升级

自去年文章写好后，miniflux 从 2.0.14 升级到了 2.0.19，应用命令有了比较大的变化：[文档地址](https://miniflux.app/docs/configuration.html)。

### 数据库升级

当时示例中的数据库postgres，也从 10.1 升级到了 12.1，因为跨了大版本，数据文件不兼容。如果要升级，需要先将数据导出，再重新导入。所以如果你已经在使用 Miniflux，并且没有使用云服务商的数据库，而是使用文章示例中的数据库方案，请不要直接修改配置，升级数据库版本，单独升级应用就好了。

### 应用配置

单机使用的完整配置依旧很简单，如果你使用云服务商的数据库，可以删除掉：

```yaml
version: '3'

services:

  miniflux:
    image: miniflux/miniflux:2.0.19
    restart: always
    depends_on:
      - db
    expose:
      - 8080
    networks:
      - traefik
    environment:
      - DEBUG=0
      - LOG_DATE_TIME=1
      # 15 mins
      - POLLING_FREQUENCY=15
      - LISTEN_ADDR=0.0.0.0:8080
      - BASE_URL=https://miniflux.lab.com/
      - CLEANUP_FREQUENCY_HOURS=876000
      - CLEANUP_ARCHIVE_READ_DAYS=36500
      - CLEANUP_REMOVE_SESSIONS_DAYS=36500
      - RUN_MIGRATIONS=1
      - CREATE_ADMIN=1
      - ADMIN_USERNAME=soulteary
      - ADMIN_PASSWORD=soulteary
      - PROXY_IMAGES=all
      - DATABASE_URL=postgres://miniflux:secret@db/miniflux?sslmode=disable
    labels:
      - "traefik.enable=true"
      - "traefik.docker.network=traefik"
      - "traefik.http.routers.whoami0.middlewares=https-redirect@file"
      - "traefik.http.routers.whoami0.entrypoints=http"
      - "traefik.http.routers.whoami0.rule=Host(`miniflux.lab.com`,`miniflux.lab.io`)"
      - "traefik.http.routers.whoami1.middlewares=content-compress@file"
      - "traefik.http.routers.whoami1.entrypoints=https"
      - "traefik.http.routers.whoami1.tls=true"
      - "traefik.http.routers.whoami1.rule=Host(`miniflux.lab.com`,`miniflux.lab.io`)"
      - "traefik.http.services.whoamibackend.loadbalancer.server.scheme=http"
      - "traefik.http.services.whoamibackend.loadbalancer.server.port=8080"

  db:
    image: postgres:12.1-alpine
    restart: always
    expose:
      - 5432
    networks:
      - traefik
    volumes:
      - ./data:/var/lib/postgresql/data
    environment:
      - POSTGRES_USER=miniflux
      - POSTGRES_PASSWORD=secret

networks:
  traefik:
    external: true
```

将内容保存为 `docker-compose.yml` 后，使用 `docker-compose up -d` 启动应用，稍等片刻看到下面的内容的时候，就说明应用启动完毕了。

启动过程中可能遇到 `miniflux_miniflux_1 exited with code 1` 的报错，这是因为 `depends_on` 仅检查依赖数据库是否启动，而不检查数据库是否 `ready for connection`，更好的解决方案是，搭配 “wait-for” 脚本使用，不过因为这里声明了应用出错重启，所以耐心等待应用重启就好了。

```bash
db_1        | 2020-02-02 14:43:24.329 UTC [45] LOG:  database system was shut down at 2020-02-02 14:43:24 UTC
db_1        | 2020-02-02 14:43:24.353 UTC [1] LOG:  database system is ready to accept connections
miniflux_1  | Current schema version: 0
miniflux_1  | Latest schema version: 25
miniflux_1  | Migrating to version: 1
db_1        | 2020-02-02 14:43:26.795 UTC [52] ERROR:  relation "schema_version" does not exist at character 21
db_1        | 2020-02-02 14:43:26.795 UTC [52] STATEMENT:  select version from schema_version
miniflux_1  | Migrating to version: 2
miniflux_1  | Migrating to version: 3
miniflux_1  | Migrating to version: 4
miniflux_1  | Migrating to version: 5
miniflux_1  | Migrating to version: 6
miniflux_1  | Migrating to version: 7
miniflux_1  | Migrating to version: 8
miniflux_1  | Migrating to version: 9
miniflux_1  | Migrating to version: 10
miniflux_1  | Migrating to version: 11
miniflux_1  | Migrating to version: 12
miniflux_1  | Migrating to version: 13
miniflux_1  | Migrating to version: 14
miniflux_1  | Migrating to version: 15
miniflux_1  | Migrating to version: 16
miniflux_1  | Migrating to version: 17
miniflux_1  | Migrating to version: 18
miniflux_1  | Migrating to version: 19
miniflux_1  | Migrating to version: 20
miniflux_1  | Migrating to version: 21
miniflux_1  | Migrating to version: 22
miniflux_1  | Migrating to version: 23
miniflux_1  | Migrating to version: 24
miniflux_1  | Migrating to version: 25
miniflux_1  | [2020-02-02T14:43:27] [INFO] Starting Miniflux...
miniflux_1  | [2020-02-02T14:43:27] [INFO] Starting scheduler...
miniflux_1  | [2020-02-02T14:43:27] [INFO] Listening on "0.0.0.0:8080" without TLS
```

浏览器中访问配置好的域名，比如本例中的 **miniflux.lab.com**或**miniflux.lab.io**，即可开始使用。

### 应用界面

![Miniflux 一如既往的简洁界面](https://attachment.soulteary.com/2020/02/02/miniflux-ui.png)

界面配置等，可参考[去年的文章进行配置](https://soulteary.com/2019/01/22/build-your-own-rss-service-with-docker-miniflux.html)。

## 最后

其他 RSS 应用的更新，可以参考上面的配置进行操作，目前看来 Miniflux / FreshRSS 还是值得继续使用的，stringer 的更新维护相比之下稍显逊色。

--EOF
