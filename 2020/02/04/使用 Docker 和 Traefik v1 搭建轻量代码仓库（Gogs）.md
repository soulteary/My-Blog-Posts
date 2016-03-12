# 使用 Docker 和 Traefik v1 搭建轻量代码仓库（Gogs）

本文成文于 2019年9月，将介绍如何使用 Traefik v1 搭建易于维护管理的 Gogs 。

原计划是替换家中 HomeLab 的代码仓库，但由于 GitLab CI 的良好体验，家里的 HomeLab 最终还是选择继续使用 GitLab。

这篇文章也就沉入了草稿箱，最近在折腾 Traefik 升级和测试服务器，遇到了一些相关的小需求，故将内容更新了一些后发布出来，希望能帮到有需要的同学。

## 写在前面

一直以来，都在使用 GitLab 作为团队/个人的仓库工具，随着版本的不断升级，GitLab 的界面功能越来越强大，消耗的服务器资源也越来越多。

最近将 GitLab Community Edition 升级到了 12.3+ ，服务器上依旧是丝般顺滑，但是家里 UPS 显示服务器待机功率默默上去了 10w。

这 10w 消耗的电费是小，但是原本静音的服务器，开始了轻微的风扇转动，这就有些不能忍了，于是有了使用更轻量应用替换 GitLab 的想法。

## 前置准备

Gogs 自身支持 HTTPS、支持挂在 SSL 证书，但是考虑到可维护性，这个事情交托给 Traefik 来处理。

Gogs 默认数据库使用的是 SQLite，轻量有余，但是作为重要数据的数据后端却不是那么安全，从官方网站的“[如何修复数据库](https://www.sqlite.org/howtocorrupt.html)”可以看到挂掉的可能性还是不少的，所以我们要将其替换。

Gogs 默认的缓存方案是应用本身的内存，一般来说足够应付个人/小团队使用，但是为了进一步提高性能和健壮性，我们将缓存功能从应用主体解耦，交托于 Redis 进行处理。

那么开始配置 Gogs 依赖的软件和环境吧。

## 配置 MySQL 数据库

使用 compose 配置数据库非常简单，二十行以内解决战斗：

```yaml
version: '3.6'

services:

  db:
    image: mysql:5.7.16
    restart: always
    expose:
      - 3306
    volumes:
      # 标准 Linux 系统下使用
      #   - /etc/localtime:/etc/localtime:ro
	  #   - /etc/timezone:/etc/timezone:ro
      - ./mysql:/var/lib/mysql
    command: --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
    environment:
      MYSQL_ROOT_PASSWORD: gogs
      MYSQL_DATABASE: gogs
      MYSQL_USER: gogs
      MYSQL_PASSWORD: gogs
      TZ: Asia/Shanghai
```

## 配置 Redis 内存缓存

使用 compose 搞定 Redis 更为简单，因为我们不需要将缓存持久化，所以不到二十行就完事了：

```yaml
  cache:
    image: redis:5.0-alpine
    restart: always
    expose:
      - 6379
    environment:
      TZ: Asia/Shanghai
    # volumes:
      # 标准 Linux 系统下使用
      # - /etc/localtime:/etc/localtime:ro
      # - /etc/timezone:/etc/timezone:ro
```

## 配置 Gogs 应用

不需要 Traefik 、MySQL、Redis 的 Gogs 编排文件显得十分简单：

```yaml
version: '3.6'

services:

  app:
    image: gogs/gogs:0.11.91
    restart: always
    ports:
      - 22:22
      - 80:3000
    volumes:
      # 标准 Linux 系统下使用
      # - /etc/localtime:/etc/localtime:ro
      # - /etc/timezone:/etc/timezone:ro
      - ./data/:/data/
    labels:
      - "traefik.enable=true"
      - "traefik.port=3000"
      - "traefik.frontend.rule=Host:${GOGS_DOMIAN}"
      - "traefik.frontend.entryPoints=http,https"
      - "traefik.frontend.headers.SSLRedirect=true"
      - "traefik.frontend.headers.SSLProxyHeaders=X-Forwarded-Proto:https"
      - "traefik.frontend.headers.STSSeconds=315360000"
      - "traefik.frontend.headers.frameDeny=true"

networks:
  traefik:
    external: true
```

但是这样配置的应用，显然少了“应用健康检查”、“仓库数据安全存储”、“页面高性能响应”的能力。

### 定制网络环境

想让 Gogs 、Redis、MySQL 作为一个整体一起正常工作，又不受到其他应用干扰，需要做几件事：

- 将它们放置相同的网段
- 仅对外暴露 Gogs ，对外隐藏 MySQL、Redis

想要达到这个效果，需要修改 `docker-compose.yml` 文件，定义 `networks` ：

```yaml
version: '3.6'

services:

  app:
    networks:
      - traefik
      - gogs

  db:
    networks:
      - gogs


  cache:
    networks:
      - gogs

networks:
  gogs:
    internal: true
  traefik:
    external: true
```

上面的示例代码中，我们声明了两个网络环境，分别为私有网络 `gogs`，用于 Gogs 和 MySQL、Redis 互通；外部的网络 `traefik`，用于暴露 Web 服务、Git SSH 服务给用户。

### 配置服务域名

应用网路互通后，Gogs 可以通过 Docker 赋予的容器名称访问 MySQL、Redis，或者使用 `gogs` 随机分配的内网地址进行数据交互。

然而这两种方案都不是特别利于维护，一旦容器扩展/重建后，容器名称会发生变化、分配的 IP 地址也会发生变化。

所以这里可以使用 compose 组网声明 MySQL、Redis 的内网域名。

```yaml
version: '3.6'

services:

  app:
    links:
      - db:mysql.gogs.lab.com
      - cache:cache.gogs.lab.com
```

然后在 Gogs 容器中访问上面的域名就能够直接访问到 MySQL、Redis 啦。

### 解决 Gogs 启动时因为依赖服务未就绪报错

如果你直接启动包含三个应用的编排文件，可能会遇到 Gogs 报错，所以可以配合给三个应用都添加健康检查，以及启动依赖关系来解决问题：

```yaml
version: '3.6'

services:

  app:
    depends_on:
      - db
      - cache
    healthcheck:
      test: ["CMD-SHELL", "wget -q --spider --proxy off localhost:3000 || exit 1"]


  db:
    healthcheck:
      test: ["CMD-SHELL", "/etc/init.d/mysql status"]
      interval: 30s

  cache:
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
```

### 解决多网卡情况下 Traefik 概率不工作的问题

现在，你可能会发现一贯很灵敏的 Traefik 出现了偶尔不工作的问题，原因是 Traefik 有时将端口暴露到了 `gogs` 私有网络网卡上，解决方案很简单，声明 Traefik 工作使用的网卡就成：

```yaml
version: '3.6'

services:

  app:
    labels:
      - "traefik.docker.network=traefik"
```

### 避免反代情况下 Gogs 寻找服务域名

根据之前的经验，反代的应用有可能会向公网 DNS 寻求帮助，查找服务域名，比如 gogs.lab.com ，为了避免这种情况发生，应用出现转半天转不开的情况，我们可以选择让 Gogs 容器的服务域名的解析地址映射为本地：

```yaml
version: '3.6'

services:

  app:
    extra_hosts:
      - "${GOGS_DOMIAN}:127.0.0.1"
```

### 持久化数据文件

相比较“默认方案”直接映射 `/data` 整个大目录，如果将子目录单独映射，则可以更好的控制应用数据迁移、配置更新。

```yaml
version: '3.6'

services:

  app:
    volumes:
      - ./app.ini:/data/gogs/conf/app.ini:ro
      - ./logs:/data/gogs/data/log
      - ./data/avatars:/data/gogs/data/avatars
      - ./data/ssh:/data/ssh
      - ./data/git:/data/git
```

### 定制页面模版

官方文档中提到我们可以修改 `custom/templates/inject/` 和 `public/css` 下的文件，来定制页面展示。

但是容器中，这块实际的目录却有一些变化，如果你有定制模版的需求，可以参考下面的配置解决问题。

```yaml
version: '3.6'

services:

  app:
    volumes:
     - ./data/custom/template/head.tmpl:/app/gogs/templates/inject/head.tmpl
      - ./data/custom/template/footer.tmpl:/app/gogs/templates/inject/footer.tmpl
      - ./data/custom/inject-assets/:/app/gogs/public/inject-assets/
```

### 限制 Gogs 的日志文件大小

加上了健康检查的 Gogs ，日志会随着时间慢慢变大，而这里日志对于我们解决问题没有丝毫帮助，因该被丢弃：

```TeXT
app_1    | [Macaron] 2019-09-28 07:07:04: Started GET / for 127.0.0.1
app_1    | [Macaron] 2019-09-28 07:07:04: Completed GET / 302 Found in 7.6542ms
app_1    | [Macaron] 2019-09-28 07:07:04: Started GET /user/login for 127.0.0.1
app_1    | [Macaron] 2019-09-28 07:07:04: Completed GET /user/login 200 OK in 16.5518ms
app_1    | [Macaron] 2019-09-28 07:07:34: Started GET / for 127.0.0.1
app_1    | [Macaron] 2019-09-28 07:07:34: Completed GET / 302 Found in 2.892ms
app_1    | [Macaron] 2019-09-28 07:07:34: Started GET /user/login for 127.0.0.1
app_1    | [Macaron] 2019-09-28 07:07:34: Completed GET /user/login 200 OK in 10.2743ms
```

配置 Gogs 应用日志输出选项，给出一个“最大尺寸”限制即可：

```yaml
version: '3.6'

services:

  app:
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
```

## 完整配置

将上述所有内容合并，完整的 `docker-compose.yml` 配置文件如下：

```yaml
version: '3.6'

services:

  app:
    image: ${DOCKER_GOGS_IMAGE}
    restart: always
    networks:
      - traefik
      - gogs
    expose:
      - 3000
    ports:
      - 22:22
    links:
      - db:${MYSQL_HOST}
      - cache:${REDIS_HOST}
    depends_on:
      - db
      - cache
    labels:
      - "traefik.enable=true"
      - "traefik.port=3000"
      - "traefik.frontend.rule=Host:${GOGS_DOMIAN}"
      - "traefik.frontend.entryPoints=http,https"
      - "traefik.frontend.headers.SSLRedirect=true"
      - "traefik.frontend.headers.SSLProxyHeaders=X-Forwarded-Proto:https"
      - "traefik.frontend.headers.STSSeconds=315360000"
      - "traefik.frontend.headers.frameDeny=true"
      - "traefik.docker.network=traefik"
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
    extra_hosts:
      - "${GOGS_DOMIAN}:127.0.0.1"
    volumes:
      # 标准 Linux 系统下使用
      # - /etc/localtime:/etc/localtime:ro
      # - /etc/timezone:/etc/timezone:ro
      - ./app.ini:/data/gogs/conf/app.ini:ro
      - ./logs:/data/gogs/data/log
      - ./data/avatars:/data/gogs/data/avatars
      - ./data/ssh:/data/ssh
      - ./data/git:/data/git
      # 根据自己需求使用
      # - ./data/custom/template/head.tmpl:/app/gogs/templates/inject/head.tmpl
      # - ./data/custom/template/footer.tmpl:/app/gogs/templates/inject/footer.tmpl
      # - ./data/custom/inject-assets/:/app/gogs/public/inject-assets/
    healthcheck:
      test: ["CMD-SHELL", "wget -q --spider --proxy off localhost:3000 || exit 1"]
      interval: 5s


  db:
    image: ${DOCKE_MYSQL_IMAGE}
    restart: always
    networks:
      - gogs
    expose:
      - 3306
    command: --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
    environment:
      MYSQL_ROOT_PASSWORD: gogs
      MYSQL_DATABASE: gogs
      MYSQL_USER: gogs
      MYSQL_PASSWORD: gogs
      TZ: Asia/Shanghai
    volumes:
      # 标准 Linux 系统下使用
      # - /etc/localtime:/etc/localtime:ro
      # - /etc/timezone:/etc/timezone:ro
      - ./mysql:/var/lib/mysql
    healthcheck:
      test: ["CMD-SHELL", "/etc/init.d/mysql status"]
      interval: 30s


  cache:
    image: ${DOCKER_REDIS_IMAGE}
    restart: always
    networks:
      - gogs
    expose:
      - 6379
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
    environment:
      TZ: Asia/Shanghai
    # volumes:
      # 标准 Linux 系统下使用
      # - /etc/localtime:/etc/localtime:ro
      # - /etc/timezone:/etc/timezone:ro

networks:
  gogs:
    internal: true
  traefik:
    external: true
```

和配置文件搭配使用的 `.env` 环境变量文件内容如下：

```TeXT
DOCKER_GOGS_IMAGE=gogs/gogs:0.11.91
GOGS_DOMIAN=gogs.lab.com

DOCKE_MYSQL_IMAGE=mysql:5.7.16
MYSQL_HOST=mysql.gogs.lab.com

DOCKER_REDIS_IMAGE=redis:5.0-alpine
REDIS_HOST=cache.gogs.lab.com
```

Gogs 使用的 `app.ini` 配置文件内容：

```Toml
APP_NAME = Private Repo
RUN_USER = git
RUN_MODE = prod

[database]
DB_TYPE  = mysql
HOST     = mysql.gogs.lab.com:3306
NAME     = gogs
USER     = gogs
PASSWD   = gogs
SSL_MODE = disable
PATH     = data/gogs.db

[cache]
ADAPTER=redis
INTERVAL=60
HOST=network=tcp,addr=cache.gogs.lab.com:6379,password=,db=0,pool_size=100,idle_timeout=180

[repository]
ROOT = /data/git/gogs-repositories
FORCE_PRIVATE=true
MAX_CREATION_LIMIT=-1
DISABLE_HTTP_GIT=true

[server]
DOMAIN           = gogs.lab.com
HTTP_PORT        = 3000
ROOT_URL         = https://gogs.lab.com/
DISABLE_SSH      = false
SSH_PORT         = 22
SSH_LISTEN_HOST  = 0.0.0.0
SSH_LISTEN_PORT  = 22
START_SSH_SERVER = false
OFFLINE_MODE     = true

[mailer]
ENABLED = false

[service]
REGISTER_EMAIL_CONFIRM = false
ENABLE_NOTIFY_MAIL     = false
DISABLE_REGISTRATION   = false
ENABLE_CAPTCHA         = false
REQUIRE_SIGNIN_VIEW    = true

[picture]
DISABLE_GRAVATAR        = true
ENABLE_FEDERATED_AVATAR = false

[session]
PROVIDER=redis
PROVIDER_CONFIG=network=tcp,addr=cache.gogs.lab.com:6379,password=,db=0,pool_size=100,idle_timeout=180

[log]
MODE      = console, file
LEVEL     = Info
ROOT_PATH = /app/gogs/log

[admin]
DISABLE_REGULAR_ORG_CREATION=true

[security]
INSTALL_LOCK = true
LOGIN_REMEMBER_DAYS=true
SECRET_KEY   = pLdr79uA4YnwDab

[other]
SHOW_FOOTER_BRANDING=false

```

## 启动应用

使用 `docker-compose up` 启动应用，稍等片刻可以看到日志类似下面：

```yaml
Network traefik is external, skipping
Creating network "gogs_gogs" with the default driver
Creating gogs_db_1    ... done
Creating gogs_cache_1 ... done
Creating gogs_app_1   ... done
Attaching to gogs_cache_1, gogs_db_1, gogs_app_1
cache_1  | 1:C 28 Sep 2019 15:38:57.955 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
cache_1  | 1:C 28 Sep 2019 15:38:57.955 # Redis version=5.0.6, bits=64, commit=00000000, modified=0, pid=1, just started
cache_1  | 1:C 28 Sep 2019 15:38:57.955 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
db_1     | Initializing database
db_1     | 2019-09-28T07:38:58.172565Z 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
db_1     | 2019-09-28T07:38:59.117064Z 0 [Warning] InnoDB: New log files created, LSN=45790
db_1     | 2019-09-28T07:38:59.264592Z 0 [Warning] InnoDB: Creating foreign key constraint system tables.
db_1     | 2019-09-28T07:38:59.281117Z 0 [Warning] No existing UUID has been found, so we assume that this is the first time that this server has been started. Generating a new UUID: 08ba4fc7-e1c3-11e9-a436-0242c0a89003.
app_1    | usermod: no changes
cache_1  | 1:M 28 Sep 2019 15:38:57.956 * Running mode=standalone, port=6379.
cache_1  | 1:M 28 Sep 2019 15:38:57.956 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
cache_1  | 1:M 28 Sep 2019 15:38:57.956 # Server initialized
cache_1  | 1:M 28 Sep 2019 15:38:57.956 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
cache_1  | 1:M 28 Sep 2019 15:38:57.956 * Ready to accept connections
db_1     | 2019-09-28T07:38:59.284542Z 0 [Warning] Gtid table is not ready to be used. Table 'mysql.gtid_executed' cannot be opened.
db_1     | 2019-09-28T07:38:59.287236Z 1 [Warning] root@localhost is created with an empty password ! Please consider switching off the --initialize-insecure option.
app_1    | Sep 28 07:38:59 syslogd started: BusyBox v1.30.1
... ...
... ...
db_1     | 2019-09-28T07:39:11.612027Z 0 [Note] Event Scheduler: Loaded 0 events
db_1     | 2019-09-28T07:39:11.612400Z 0 [Note] mysqld: ready for connections.
db_1     | Version: '5.7.16'  socket: '/var/run/mysqld/mysqld.sock'  port: 3306  MySQL Community Server (GPL)
app_1    | 2019/09/28 07:39:11 [TRACE] Custom path: /data/gogs
app_1    | 2019/09/28 07:39:11 [TRACE] Log path: /app/gogs/log
app_1    | 2019/09/28 07:39:11 [TRACE] Build Time: 2019-08-12 02:30:12 UTC
app_1    | 2019/09/28 07:39:11 [TRACE] Build Git Hash: c154721f4a8f3e24d2f6fb61e74b4b64529255c2
app_1    | 2019/09/28 07:39:11 [ INFO] Private Repo 0.11.91.0811
app_1    | 2019/09/28 07:39:11 [ INFO] Cache Service Enabled
app_1    | 2019/09/28 07:39:11 [ INFO] Session Service Enabled
app_1    | 2019/09/28 07:39:14 [ INFO] Git Version: 2.22.0
app_1    | 2019/09/28 07:39:14 [ INFO] Git config user.name set to Gogs
app_1    | 2019/09/28 07:39:14 [ INFO] Git config user.email set to gogs@fake.local
app_1    | 2019/09/28 07:39:14 [ INFO] SQLite3 Supported
app_1    | 2019/09/28 07:39:14 [ INFO] Run Mode: Production
app_1    | 2019/09/28 07:39:14 [ INFO] Listen: http://0.0.0.0:3000
app_1    | [Macaron] 2019-09-28 07:39:14: Started GET / for 127.0.0.1
app_1    | [Macaron] 2019-09-28 07:39:14: Completed GET / 302 Found in 2.4563ms
app_1    | [Macaron] 2019-09-28 07:39:14: Started GET /user/login for 127.0.0.1
app_1    | [Macaron] 2019-09-28 07:39:14: Completed GET /user/login 200 OK in 13.0259ms
app_1    | [Macaron] 2019-09-28 07:39:20: Started GET / for 127.0.0.1
app_1    | [Macaron] 2019-09-28 07:39:20: Completed GET / 302 Found in 1.4996ms
app_1    | [Macaron] 2019-09-28 07:39:20: Started GET /user/login for 127.0.0.1
app_1    | [Macaron] 2019-09-28 07:39:20: Completed GET /user/login 200 OK in 11.8768ms
app_1    | [Macaron] 2019-09-28 07:39:25: Started GET / for 127.0.0.1
app_1    | [Macaron] 2019-09-28 07:39:25: Completed GET / 302 Found in 1.9598ms
app_1    | [Macaron] 2019-09-28 07:39:25: Started GET /user/login for 127.0.0.1
app_1    | [Macaron] 2019-09-28 07:39:25: Completed GET /user/login 200 OK in 10.8364ms
app_1    | [Macaron] 2019-09-28 07:39:30: Started GET / for 127.0.0.1
```

访问 `gogs.lab.com` 打开页面，就可以开始使用啦。

![gogs 默认界面](https://attachment.soulteary.com/2020/02/04/gogs.png)

## 备份数据

备份数据需要使用 `gogs backup` ，不论是在 容器内执行，还是在容器外使用 `docker exec` 都是可以的。

```yaml
chown -R /backup
su git -c "/app/gogs/gogs backup -v --target /backup/"

2019/09/28 17:12:36 [ INFO] Backup succeed! Archive is located at: /app/gogs/backup/gogs-backup-20190928171236.zip
```

## 最后

比较巧合的是，去年九月开始，gogs 的更新开始了休眠模式，随后它的 fork 版本 Gitea 开始了茁壮成长。

下一篇聊聊，怎么使用 Traefik v2 TCP 模式搭建 Gitea 。

--EOF