# 使用 Docker 和 Traefik 搭建 WordPress（Nginx）

[前一篇](https://soulteary.com/2019/04/07/use-docker-and-traefik-to-build-wordpress.html) 内容介绍了如何使用官方镜像快速搭建 **WordPress**，但是官方默认是“胖容器”应用，接下来将聊聊同样基于容器搭建的其他选择：Nginx。演示如何改造应用为“瘦”容器应用。

本文将花费十分钟左右，介绍如何在 Docker 容器中搭配 Traefik 使用 WordPress 和 Nginx 。

<!-- more -->

## 为什么选择 Nginx

> NGINX is a free, open-source, high-performance HTTP server and reverse proxy, as well as an IMAP/POP3 proxy server. NGINX is known for its high performance, stability, rich feature set, simple configuration, and low resource consumption.

除了官方对于 Nginx 免费、开源、轻量、高性能的定位之外，当前不论在企业中，还是个人学习使用，Nginx 的资源的丰富程度远胜 Apache （前文 WordPress 容器镜像默认工具）。

另外，Nginx 的两个著名的派生应用，也在企业中广泛开花：Tengine、OpenResty，本文中的内容同样适用于这两个版本的“Nginx”。

## 容器镜像清单

本文将使用下面几个官方镜像作为演示，上面有提过，你可以使用 Nginx 的“同类”们将它进行替换。

- [Nginx](https://hub.docker.com/_/nginx): `1.15.10-alpine` 
	- 作为替换 Apache 的服务前端
- [WordPress](https://hub.docker.com/_/wordpress): `5.1.1-php7.1-fpm-alpine`
	- 使用仅包含 WordPress 代码和 PHP 运行时的容器
- [mariadb](https://hub.docker.com/_/mariadb): `10.3.14`
	- 我们的数据库，如果有云数据库，可以不需要配置

## Traefik 的使用

Traefik 的具体使用，可以参考以往的文章，比如：[使用服务发现改善开发体验](https://soulteary.com/2018/06/11/use-server-side-discovery-improve-development.html)、[更完善的 Docker + Traefik 使用方案](https://soulteary.com/2018/08/28/better-use-of-docker-and-traefik.html) 等，更多内容，可以翻看历史内容的标签，这里不过多赘述。

本文只需要关注编排文件中的 `labels` 和 `networks` 字段配置就足够啦。

对不同容器服务的 `networks` 字段，声明包含相同的内容，则可以让不同应用所处于的网络一致。

```yaml
networks:
  - traefik
```

比如上面的声明，会让容器服务都处于名为 `traefik` 的网络环境中。

## 改写容器编排配置

下面的配置在上一篇文章中提到过，为了避免篇幅过长，我做了适当精简。

```yaml
version: '3'

services:

  wp:
    image: ${WP_IMAGE}
    restart: always
    networks:
      - traefik
    environment:
        WORDPRESS_DB_HOST: ${DB_HOST}
        WORDPRESS_TABLE_PREFIX: ${WP_DB_PREFIX}
        WORDPRESS_DB_NAME: ${DB_NAME}
        WORDPRESS_DB_USER: ${DB_USER}
        WORDPRESS_DB_PASSWORD: ${DB_PASS}
    volumes:
    # 如果你有定制上传文件尺寸的需求
    # - ./config/php.conf.uploads.ini:/usr/local/etc/php/conf.d/uploads.ini
      - ./wp-app:/var/www/html
    labels:
      - "traefik.enable=true"
      - "traefik.frontend.rule=Host:${WP_DOMAINS}"
      - "traefik.frontend.entryPoints=https,http"

  mariadb:
    image: ${DB_IMAGE}
    restart: always
    container_name: ${DB_HOST}
    networks:
      - traefik
    environment:
      MYSQL_DATABASE: ${DB_NAME}
      MYSQL_USER: ${DB_USER}
      MYSQL_PASSWORD: ${DB_PASS}
      MYSQL_ROOT_PASSWORD: ${DB_ROOT_PASS}
    volumes:
      - ./data:/var/lib/mysql

networks:
  traefik:
    external: true
```

如果我们使用 `Nginx` 作为 “Web 前端”，那么这里需要进行适当的调整。

### 改写 WordPress 容器编排配置

因为使用 Nginx 取代了 WordPress 作为流量入口，所以 WordPress 服务可以不再绑定 Traefik ，注册请求域名，`labels` 字段可以悉数删除。

另外，因为 WordPress 需要被 Nginx 远程调用，所以需要给 WordPress 这个服务添加一个固定的 `container_name` ，以便于调用。如同上面配置中，WordPress 调用 Mariadb 一样。

```yaml
 services:
  wp:
    image: ${WP_IMAGE}
    restart: always
    container_name: ${WP_HOST}
    networks:
      - traefik
    environment:
        WORDPRESS_DB_HOST: ${DB_HOST}
        WORDPRESS_TABLE_PREFIX: ${WP_DB_PREFIX}
        WORDPRESS_DB_NAME: ${DB_NAME}
        WORDPRESS_DB_USER: ${DB_USER}
        WORDPRESS_DB_PASSWORD: ${DB_PASS}
    volumes:
      - ./wordpress:/var/www/html
```

上面就是改造好的配置啦，数据库没有任何变化，我们略过它，开始配置 Nginx 。

### 编写 Nginx 容器编排配置

```yaml
services:
  nginx:
    image: ${NGX_IMAGE}
    restart: always
    networks:
      - traefik
    expose:
      - 80
    volumes:
      - ./conf:/etc/nginx/conf.d
      - ./logs:/var/log/nginx
      - ./wordpress:/var/www/html
    depends_on:
      - wp
    labels:
      - "traefik.enable=true"
      - "traefik.frontend.rule=Host:${NGX_DOMAINS}"
      - "traefik.frontend.entryPoints=https,http"
```

因为 Nginx 接管了入口流量，所以在 Traefik 上注册服务发现的任务就非它莫属了，这里使用 `labels` 字段，添加一些 Traefik 支持的指令，进行服务注册。

另外，这里需要将 WordPress 挂载到文件系统中的文件也挂载到 Nginx 中，就像这样。

```yaml
- ./wordpress:/var/www/html
```

最后，简单声明一个 Nginx 配置，用来描述如何调用 WordPress 即可。

### 创建简单的 Nginx 服务配置

将下面的内容保存为 `wp.conf` 即可。

```yaml
server {
    listen 80;

    root /var/www/html;
    index index.php;

    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass wp:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }
}
```

### 编写环境文件

和前文一样，为了可维护性，我们将环境配置信息和应用编排文件进行了分离。

```TeXT
WP_IMAGE=wordpress:5.1.1-php7.1-fpm-alpine
WP_DB_PREFIX=wp
WP_HOST=wp

NGX_IMAGE=nginx:1.15.10-alpine
NGX_DOMAINS=wp.lab.com,wp.lab.io

DB_IMAGE=mariadb:10.3.14
DB_HOST=wp-db
DB_NAME=wordpress
DB_USER=wordpress
DB_PASS=wordpress
DB_ROOT_PASS=soulteary
```

将上面的文件保存为 `.env` 后，我们可以开始启动应用了。

## 启动完整的应用

在启动应用之前，我们将刚刚修改的编排文件进行汇总。

```yaml
version: '3'

services:

  wp:
    image: ${WP_IMAGE}
    restart: always
    container_name: ${WP_HOST}
    networks:
      - traefik
    environment:
        WORDPRESS_DB_HOST: ${DB_HOST}
        WORDPRESS_TABLE_PREFIX: ${WP_DB_PREFIX}
        WORDPRESS_DB_NAME: ${DB_NAME}
        WORDPRESS_DB_USER: ${DB_USER}
        WORDPRESS_DB_PASSWORD: ${DB_PASS}
    volumes:
      - ./wordpress:/var/www/html

  mariadb:
    image: ${DB_IMAGE}
    restart: always
    container_name: ${DB_HOST}
    networks:
      - traefik
    environment:
      MYSQL_DATABASE: ${DB_NAME}
      MYSQL_USER: ${DB_USER}
      MYSQL_PASSWORD: ${DB_PASS}
      MYSQL_ROOT_PASSWORD: ${DB_ROOT_PASS}
    volumes:
      - ./data:/var/lib/mysql

  nginx:
    image: ${NGX_IMAGE}
    restart: always
    networks:
      - traefik
    expose:
      - 80
    volumes:
      - ./conf:/etc/nginx/conf.d
      - ./logs:/var/log/nginx
      - ./wordpress:/var/www/html
    depends_on:
      - wp
    labels:
      - "traefik.enable=true"
      - "traefik.frontend.rule=Host:${NGX_DOMAINS}"
      - "traefik.frontend.entryPoints=https,http"

networks:
  traefik:
    external: true
```

将文件保存为 `docker-compose.yml` 后，我们使用 `docker-compose up` 启动应用，验证应用是否正常。

当你看到下面的日志时，你的应用就可以进行访问啦。

```TeXT
wp-db      | 2019-04-07  3:42:58 0 [Note] Server socket created on IP: '::'.
wp-db      | 2019-04-07  3:42:58 0 [Warning] 'proxies_priv' entry '@% root@2ed5be12f731' ignored in --skip-name-resolve mode.
wp-db      | 2019-04-07  3:42:58 0 [Note] Reading of all Master_info entries succeded
wp-db      | 2019-04-07  3:42:58 0 [Note] Added new Master_info '' to hash table
wp-db      | 2019-04-07  3:42:58 0 [Note] mysqld: ready for connections.
wp-db      | Version: '10.3.14-MariaDB-1:10.3.14+maria~bionic'  socket: '/var/run/mysqld/mysqld.sock'  port: 3306  mariadb.org binary distribution
wp         | [07-Apr-2019 03:42:59] NOTICE: fpm is running, pid 1
wp         | [07-Apr-2019 03:42:59] NOTICE: ready to handle connections
```

依旧是使用浏览器访问刚刚 `.env` 中配置的域名，依旧是熟悉的操作，填写信息，进行著名的“三分钟”安装，之后，便是一个新的站点诞生啦。

![久违的 WordPress Dashboard](https://attachment.soulteary.com/2019/04/07/wp-ngx-dashboard.png)

### 一些额外的小技巧

我们使用 `Compose` 进行应用启动的时候，如果是第一次调试，建议执行：

```bash
docker-compose up
```

因为可以在终端中直接看到应用的实际运行日志，如果出错，可以按下 `CTRL+C` 组合键，中断执行，返回调试。

当你的应用**完全就绪**之后，我们需要长期稳定的运行这个服务的时候，再使用 `Compose` 的时候，则可以添加一个 `-d` 参数，让应用以 daemon 模式执行。

```bash
docker-compose up -d
```

这时，应用会乖乖的静默在后台执行，不会向终端输出任何有价值的信息，如果应用异常，我们需要调试，想看到应用日志该怎么处理呢？执行下面的命令就可以了。

```bash
docker-compose logs -f
```

如果发现应用执行出错，使用 `docker-compose down` 结束应用运行后，调整编排配置文件，重新使用不带参数的的 `docker-compose up` 启动应用，待应用完全就绪后，再添加 `daemon` 参数就可以了。

## 最后

感谢各位持续关注、鼓励我写作的同学。是你们的关注让我可以在写作过程中不必重复赘述一堆内容，成文变的高效起来。

常见的 WordPress 多见于部署于线上，需要依赖 `MySQL`、`Mariadb`运行，不适合“随身携带”、或者低配置机器运行。

下一篇内容将聊聊 “如何打造随身携带的 WordPress”。
