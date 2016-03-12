# 使用 Docker 和 Traefik v2 搭建轻量代码仓库（Gitea）

[使用 Docker 和 Traefik v1 搭建轻量代码仓库（Gogs）](https://soulteary.com/2020/02/04/gogs-git-server-with-docker-and-traefik-v1.html) 一文中，提到了 Gogs。本文将介绍它的增强版本：Gitea 以及如何搭配 Traefik v2 一起使用。

如果你有了解过之前到文章，大概三分钟左右可以搭建完毕。

## 写在前面

官方提供了一份表格，展示了[Gitea 与其他“代码仓库”的差异](https://docs.gitea.io/en-us/comparison/)，有兴趣可以看看。

本文将使用到 Traefik 和 Docker，如果不太熟悉，可以阅读以往的文章以做了解：[Docker](https://soulteary.com/tags/docker.html)、[Traefik](https://soulteary.com/tags/traefik.html)。

## Traefik v2 配置调整

我们使用 SSH 和 HTTP 协议进行数据上传下载（`git clone` / `git push`），所以需要让 Traefik 提供 TCP 协议服务，这里建议单独新建一个入口点。

因为在 Traefik v2 中，每一个用户能够访问到的服务都需要一个入口点（entrypoint），如果我们不单独指定入口点背后的服务类型，那么入口点会先尝试看看它背后对接的服务是否是 TCP，如果不是的话，再尝试使用 HTTP 协议提供服务，所以如果不分离两种协议的话，对整体服务的性能会有一定影响。

参考 [ Traefik 2 使用指南，愉悦的开发体验 ](https://soulteary.com/2020/01/28/traefik-2-user-guide-pleasant-development-experience.html) 一文中的配置，在 `traefik.toml` 中添加一个新的入口点：

```Toml
[entryPoints]
  [entryPoints.http]
    address = ":80"
  [entryPoints.https]
    address = ":443"
  [entryPoints.git]
    address = ":22"
```

如果是使用 `docker-compose` 启动的 Traefik 服务，那么需要将对应的端口也进行开放：

```Toml
  traefik:
    container_name: traefik
    image: traefik:v2.1.3
    restart: always
    ports:
      - 80:80
      - 443:443
      - 22:22
```

## 最简单的配置

在 Gogs 那篇文章中提到过，不推荐使用 SQLite 作为数据库，不过如果你备份比较勤快，觉得问题不大，那么可以试试下面的配置：

```yaml
version: '3.6'

services:

  gitea:
    image: gitea/gitea:1.10.3
    environment:
      - USER_UID=1000
      - USER_GID=1000
      - APP_NAME=Private
      - RUN_MODE=prod
      - RUN_USER=git
      - SSH_DOMAIN=gitea.lab.com
      - SSH_PORT=22
      - SSH_LISTEN_PORT=22
      #  如果不希望使用 SSH 协议
      #- DISABLE_SSH=true
      - HTTP_PORT=3000
      - ROOT_URL=https://gitea.lab.com
      - LFS_START_SERVER=false
      - DB_TYPE=sqlite3
      - INSTALL_LOCK=false
      - DISABLE_GRAVATAR=true
    networks:
      - traefik
    restart: unless-stopped
    labels:
      - "traefik.enable=true"
      - "traefik.docker.network=traefik"
      - "traefik.http.routers.giteaweb.middlewares=https-redirect@file"
      - "traefik.http.routers.giteaweb.entrypoints=http"
      - "traefik.http.routers.giteaweb.rule=Host(`gitea.lab.com`)"
      - "traefik.http.routers.giteassl.middlewares=content-compress@file"
      - "traefik.http.routers.giteassl.entrypoints=https"
      - "traefik.http.routers.giteassl.tls=true"
      - "traefik.http.routers.giteassl.rule=Host(`gitea.lab.com`)"
      - "traefik.http.services.giteabackend.loadbalancer.server.scheme=http"
      - "traefik.http.services.giteabackend.loadbalancer.server.port=3000"
      - "traefik.tcp.routers.giteassh.rule=HostSNI(`*`)"
      - "traefik.tcp.routers.giteassh.tls=true"
      - "traefik.tcp.routers.giteassh.entrypoints=git"
      - "traefik.tcp.routers.giteassh.service=gitea"
      - "traefik.tcp.services.gitea.loadbalancer.server.port=22"
    volumes:
      # 标准 Linux 系统下使用
      # - /etc/localtime:/etc/localtime:ro
      # - /etc/timezone:/etc/timezone:ro
      - ./repositories:/data/git/repositories
      - ./data:/data/gitea/
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
    extra_hosts:
      - "git.lab.com:127.0.0.1"
    healthcheck:
      test: ["CMD-SHELL", "wget -q --spider --proxy off localhost:3000 || exit 1"]
      interval: 5s

networks:
  traefik:
    external: true
```

将内容保存为 `docker-compose.yml` ，使用 `docker-compose up -d` 启动服务，访问上面配置的域名，会看到 Gitea 的欢迎界面。

![Gitea 首次运行界面](https://attachment.soulteary.com/2020/02/04/gitea-first-look.jpg)

打开页面看到服务已经正确启动起来了，点击注册/登陆按钮，首次使用会被重定向到 `/install` 目录。

![Gitea 配置界面](https://attachment.soulteary.com/2020/02/04/gitea-install.jpg)

如果你确定要使用 SQLite ，可以填写下管理员账号，然后点击立即安装即可。

![Gitea 安装完毕](https://attachment.soulteary.com/2020/02/04/gitea-installed.jpg)

接下来就是配置仓库，正常推送数据啦。

## 使用数据库

这里推荐云服务低配置数据库实例，不过如果低频率使用，使用 `docker-compose` 启动一个实例也问题不大，以 MySQL 为例。

```yaml
version: '3.6'

services:
  
  gitea:
    image: gitea/gitea:1.10.3
    environment:
      - USER_UID=1000
      - USER_GID=1000
      - APP_NAME=Private
      - RUN_MODE=prod
      - RUN_USER=git
      - SSH_DOMAIN=gitea.lab.com
      - SSH_PORT=22
      - SSH_LISTEN_PORT=22
      #  如果不希望使用 SSH 协议
      #- DISABLE_SSH=true
      - HTTP_PORT=3000
      - ROOT_URL=https://gitea.lab.com
      - LFS_START_SERVER=false
      - DB_TYPE=mysql
      - DB_HOST=db
      - DB_NAME=gitea
      - DB_USER=gitea
      - DB_PASSWD=gitea
      - DB_CHARSET=utf8
      - INSTALL_LOCK=false
      - DISABLE_GRAVATAR=true
    networks:
      - traefik
      - gitea
    restart: unless-stopped
    labels:
      - "traefik.enable=true"
      - "traefik.docker.network=traefik"
      - "traefik.http.routers.giteaweb.middlewares=https-redirect@file"
      - "traefik.http.routers.giteaweb.entrypoints=http"
      - "traefik.http.routers.giteaweb.rule=Host(`gitea.lab.com`)"
      - "traefik.http.routers.giteassl.middlewares=content-compress@file"
      - "traefik.http.routers.giteassl.entrypoints=https"
      - "traefik.http.routers.giteassl.tls=true"
      - "traefik.http.routers.giteassl.rule=Host(`gitea.lab.com`)"
      - "traefik.http.services.giteabackend.loadbalancer.server.scheme=http"
      - "traefik.http.services.giteabackend.loadbalancer.server.port=3000"
      - "traefik.tcp.routers.giteassh.rule=HostSNI(`*`)"
      - "traefik.tcp.routers.giteassh.tls=true"
      - "traefik.tcp.routers.giteassh.entrypoints=git"
      - "traefik.tcp.routers.giteassh.service=gitea"
      - "traefik.tcp.services.gitea.loadbalancer.server.port=22"
    volumes:
      # 标准 Linux 系统下使用
      # - /etc/localtime:/etc/localtime:ro
      # - /etc/timezone:/etc/timezone:ro
      - ./repositories:/data/git/repositories
      - ./data:/data/gitea/
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
    extra_hosts:
      - "git.lab.com:127.0.0.1"
    healthcheck:
      test: ["CMD-SHELL", "wget -q --spider --proxy off localhost:3000 || exit 1"]
      interval: 5s

  db:
    image: mysql:5.7.16
    restart: always
    networks:
      - gitea
    expose:
      - 3306
    command: --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
    environment:
      MYSQL_ROOT_PASSWORD: gitea
      MYSQL_DATABASE: gitea
      MYSQL_USER: gitea
      MYSQL_PASSWORD: gitea
      TZ: Asia/Shanghai
    volumes:
      - ./mysql:/var/lib/mysql
      # 标准 Linux 系统下使用
      # - /etc/localtime:/etc/localtime:ro
      # - /etc/timezone:/etc/timezone:ro
    # healthcheck:
    #   test: ["CMD-SHELL", "/etc/init.d/mysql status"]
    #   interval: 30s

  cache:
    image: redis:5.0-alpine
    restart: always
    networks:
      - gitea
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
  gitea:
    internal: true
  traefik:
    external: true
```

同样的，将内容保存为 `docker-compose.yml` ，使用 `docker-compose up -d` 启动服务，访问上面配置的域名，然后参考上文进行配置安装即可。

## 其他

更多的配置变量可以从这里找到：[config-cheat-sheet](https://docs.gitea.io/en-us/config-cheat-sheet/)，不过并不是每个变量都可以直接使用，具体是否使用需要翻看代码或者进行尝试。

当服务完整启动之后，在 Traefik 的控制面板中将看到 TCP 路由和服务的完整状态，类似下面这样：

![Traefik 面板 TCP 状态](https://attachment.soulteary.com/2020/02/04/traefik-v2-tcp.jpg)

## 最后

先写到这里，其实已经聊过不少软件，后续有机会聊聊这类软件的用户集成，以及权限管理。

--EOF