# 使用 Docker 和 Traefik v2 搭建 Flarum 轻论坛应用

距离写完[《使用 Docker 和 Traefik 搭建 Flarum 轻论坛应用》](https://soulteary.com/2019/07/16/building-a-flarum-forum-app-with-docker-and-traefik.html)已经过去了十个月。

在上一篇搭建教程中，我描述过这个应用的优劣势，因为缺乏开发者，所以时隔近一年的时间里，软件除了能够保持缓慢前行外，并没有实质的变化。国内相关社区同样因为缺少活力，依旧还在使用陈旧的迭代方案，短期来看，应该不会有太多惊喜出现，不过作为一款轻量社区来讲，flarum 是合格的。

本文将介绍如何使用 Docker 来对 Flarum 最新版 v0.1.0-beta.12 进行容器封装，以及如何搭配 traefik v2 一起使用。

## 写在前面

在[这篇“搭建RSS工具”文章](https://soulteary.com/2019/01/05/build-your-own-rss-service-with-docker-freshrss.html)的末尾，我提过：

> 之前写文章总是考虑没有阅读基础的同学，而忽略了一直订阅、关注着我的同学，未来重复的内容，我将会和本文一样，给予简短的指引，不赘述基础建设，只聊主题相关的核心部分。

为了保证内容的简洁，相关资料可以自行从网站历史资料找翻阅，学习这件事只有探索折腾才有意思，不是么？

## 使用环境

- Docker: 19.03.8, build afacb8b
- docker-compose: version 1.25.4, build 8d51620a
- Flarum: v0.1.0-beta.12  (官方版本)
- PHP: 7.4 (官方镜像)
- MySQL: 5.7 (官方镜像)
- Redis: 5.0.8 (官方镜像)

## 准备数据库

单机论坛可以考虑使用下面的编排文件，启动项目所需的数据库和缓存服务。

```yaml
version: '3.6'

services:

  mysql:
    container_name: ${DOCKER_MYSQL_HOST}
    image: ${DOCKER_MYSQL_IMAGE}
    restart: always
    expose:
      - 3306
    networks:
      - traefik
    environment:
      MYSQL_USER: ${DOCKER_MYSQL_USER}
      MYSQL_PASSWORD: ${DOCKER_MYSQL_PASS}
      MYSQL_DATABASE: ${DOCKER_MYSQL_NAME}
      MYSQL_ROOT_PASSWORD: ${DOCKER_MYSQL_ROOT}
    command: --character-set-server=utf8mb4 --collation-server=utf8mb4_bin --default-storage-engine=INNODB --max_allowed_packet=256M --transaction-isolation=READ-COMMITTED --binlog_format=row --ngram_token_size=2
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /etc/timezone:/etc/timezone:ro
      - ./db:/var/lib/mysql
    healthcheck:
      test: ["CMD-SHELL", "/etc/init.d/mysql status"]
      interval: 30s

  redis:
    image: ${DOCKER_REDIS_IMAGE}
    restart: always
    container_name: ${DOCKER_REDIS_HOST}
    expose:
      - 6379
    networks:
      - traefik
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
    environment:
      TZ: Asia/Shanghai

networks:
  traefik:
    external: true
```

将上面的内容保存为 `docker-compose.yml` ，然后继续配置 `.env` 环境变量文件。

```Toml
DOCKER_REDIS_IMAGE=redis:5.0.8-alpine
DOCKER_REDIS_HOST=flarum-redis.lab.com

DOCKER_MYSQL_IMAGE=mysql:5.7
DOCKER_MYSQL_HOST=flarum-db.lab.com
DOCKER_MYSQL_USER=flarum
DOCKER_MYSQL_PASS=flarum
DOCKER_MYSQL_NAME=flarum
DOCKER_MYSQL_ROOT=flarum
```

两个文件都准备好后，使用 `docker-compose up -d`，启动数据库和缓存服务即可。

为了让 flarum 支持搜索中文内容，全文检索以配置好，最小搜索长度为 2，如果有特殊需求可以修改 `--ngram_token_size=2` 为适合你的数值。

如果你不需要使用 Redis Session 功能，可以删除掉 Redis 相关内容。

## 获取当前版本最新代码

为了项目的可维护性，我们一般需要将应用和其依赖组件进行版本锁定。所以这里建议使用 composer 将代码下载下来后，作为“代码基”使用代码仓库单独管理保存，而非在容器中进行下载构建，这样对于每次软件变更都能做到心中有数。

和之前一样，我们使用下面的命令可以将 flarum 当前最新的 beta 版本下载到本地。

```bash
composer create-project flarum/flarum ./codebase --stability=beta
```

如果长时间不能完成下载，可以在命令行后添加 `-vvv`，可以辅助判断是因为什么问题导致下载出现异常。

如果是因为网络问题，可以考虑使用下面的方法，将 composer 源修改为阿里云或其他国内 CDN 地址（以阿里云为例）：

```bash
composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/
```

修改完毕之后，可以使用下面的命令验证修改是否成功。

```bash
composer config -g -l repo.packagist | grep repositories.packagist.org.url

# 输出如下则修改成功
[repositories.packagist.org.url] https://mirrors.aliyun.com/composer/
```

修改完毕之后，再次执行 `create-project` 命令即可。

如果是使用 flarum 做线上业务，此处可以考虑使用生产环境的私有 composer 搭配持续集成进行操作，安全性和可靠性会有极大的提升，细节可参考下面两篇文章：[《搭建高性能的私有 Composer 镜像服务》](https://soulteary.com/2019/08/23/build-a-high-performance-private-composer-image-service.html)、[《如何搭配 CI 系统使用 Composer》](https://soulteary.com/2019/08/24/how-to-use-composer-with-ci-system.html)。

准备好的 **codebase** 目录结构如下：

```TeXT
├── CHANGELOG.md
├── LICENSE
├── README.md
├── composer.json
├── composer.lock
├── extend.php
├── flarum
├── public
├── site.php
├── storage
└── vendor
```

确认 **codebase** 目录内已经保存了完整的 flarum 程序后，我们开始编写容器镜像配置文件。

## 封装容器镜像

之前文章中，我使用了当时最新的 `PHP 7.3.2`，如今 `PHP 7.4` 已经到来，所以这里将使用最新版本的 PHP 封装 Flarum 的运行环境，我当前选择的版本是：`php:7.4-fpm-alpine3.11`。

Dockerfile 文件内容可参考下面：

```bash
FROM php:7.4-fpm-alpine3.11
LABEL maintainer="soulteary@gmail.com"

ENV LANG en_US.UTF-8
ENV LANGUAGE en_US.UTF-8
ENV LC_ALL=en_US.UTF-8

ARG USE_CHINA_MIRROR=0
RUN if [ "$USE_CHINA_MIRROR" = 1 ]; then \
        echo 'use china mirror' && \
        echo '' > /etc/apk/repositories && \
        echo "https://mirror.tuna.tsinghua.edu.cn/alpine/v3.11/main"         >> /etc/apk/repositories && \
        echo "https://mirror.tuna.tsinghua.edu.cn/alpine/v3.11/community"    >> /etc/apk/repositories && \
        echo "Asia/Shanghai" > /etc/timezone; \
    fi

RUN apk --no-cache --no-progress update && \
    apk --no-cache --no-progress upgrade

RUN apk add libpng libpng-dev libjpeg-turbo libjpeg-turbo-dev libwebp libwebp-dev zlib-dev libxpm-dev freetype freetype-dev oniguruma-dev

RUN docker-php-ext-install pdo pdo_mysql mbstring

RUN docker-php-ext-configure gd --with-freetype=/usr/include/ --with-jpeg=/usr/include/ && \
    docker-php-ext-install -j$(getconf _NPROCESSORS_ONLN) gd

ENTRYPOINT ["docker-php-entrypoint"]

WORKDIR /var/www/html

RUN set -eux; \
	cd /usr/local/etc; \
	if [ -d php-fpm.d ]; then \
		# for some reason, upstream's php-fpm.conf.default has "include=NONE/etc/php-fpm.d/*.conf"
		sed 's!=NONE/!=!g' php-fpm.conf.default | tee php-fpm.conf > /dev/null; \
		cp php-fpm.d/www.conf.default php-fpm.d/www.conf; \
	else \
		# PHP 5.x doesn't use "include=" by default, so we'll create our own simple config that mimics PHP 7+ for consistency
		mkdir php-fpm.d; \
		cp php-fpm.conf.default php-fpm.d/www.conf; \
		{ \
			echo '[global]'; \
			echo 'include=etc/php-fpm.d/*.conf'; \
		} | tee php-fpm.conf; \
	fi; \
	{ \
		echo '[global]'; \
		echo 'error_log = /proc/self/fd/2'; \
		echo; echo '; https://github.com/docker-library/php/pull/725#issuecomment-443540114'; echo 'log_limit = 8192'; \
		echo; \
		echo '[www]'; \
		echo '; if we send this to /proc/self/fd/1, it never appears'; \
		echo 'access.log = /proc/self/fd/2'; \
		echo; \
		echo 'clear_env = no'; \
		echo; \
		echo '; Ensure worker stdout and stderr are sent to the main error log.'; \
		echo 'catch_workers_output = yes'; \
		echo 'decorate_workers_output = no'; \
	} | tee php-fpm.d/docker.conf; \
	{ \
		echo '[global]'; \
		echo 'daemonize = no'; \
		echo; \
		echo '[www]'; \
		echo 'listen = 9000'; \
	} | tee php-fpm.d/zz-docker.conf


# Override stop signal to stop process gracefully
# https://github.com/php/php-src/blob/17baa87faddc2550def3ae7314236826bc1b1398/sapi/fpm/php-fpm.8.in#L163
STOPSIGNAL SIGQUIT

EXPOSE 9000
CMD ["php-fpm"]
```

上面的配置文件主要做了几件事：

1. 安装必须的环境依赖；
2. 编译 PHP 相关插件；
3. 设置 PHP FPM。

将内容保存为 `Dockerfile` 后，可以使用下面的命令构建我们所需要的镜像：

```bash
docker build -t soulteary/flarum:v0.1.0-beta.12 -f Dockerfile .
```

如果是在国内网络环境编译，可以使用下面的命令，加速编译构建过程。

```bash
docker build --build-arg USE_CHINA_MIRROR=1 -t soulteary/flarum:v0.1.0-beta.12 -f Dockerfile .
```

准备好运行镜像后，我们就可以准备运行目录和容器编排配置文件了。

## 准备目录结构

我们提供给 flarum 使用的目录结构如下：

```TeXT
├── Dockerfile
├── conf
├── docker-compose.yml
├── logs
├── start.sh
└── wwwroot
```

将之前准备好的 **“CodeBase”** 目录中的以下内容复制到 **wwwroot** 目录下：

- extend.php
- site.php
- flarum
- public

考虑到后续会安装/卸载 flarum 插件，这个工作可以交给启动脚本来做，至于启动脚本怎么编写，可以耐心继续往下看。

## 准备编排文件及 Nginx 配置

因为使用 FPM 模式的 PHP 环境，我们需要借助 Apache / Nginx 的帮助来提供 Web 服务，我这里选择 Nginx。

使用 Nginx，需要分别配置两个文件，先配置 `conf/docker-nginx.conf`：

```bash
user nginx;
worker_processes 1;

error_log /var/log/nginx/error.log warn;
pid /var/run/nginx.pid;

events {
    worker_connections 1024;
}

http {
    include /etc/nginx/mime.types;

    # 交给 Traefik 或者 SLB 处理，不开启 Gzip
    #gzip  on;
    include /etc/nginx/conf.d/*.conf;
    default_type application/octet-stream;
    log_format main '$http_x_forwarded_for - $remote_user [$time_local] "$request" $status $body_bytes_sent "$http_referer" "$http_user_agent"';
    access_log off;
    sendfile on;

    keepalive_timeout 65;
}
```

接着来配置 `conf/vhost.conf` 文件：

```bash
server {
    listen 80;
    server_name lab.com lab.io;
    server_tokens off;

    access_log  /var/log/nginx/docker-access.log;
    error_log   /var/log/nginx/docker-error.log;

    root        /wwwroot/public;
    index       index.php index.html;

    location ~ \.php$ {
        try_files               $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass            php:9000;
        fastcgi_index           index.php;
        include                 fastcgi_params;
        fastcgi_param           SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param           PATH_INFO $fastcgi_path_info;
    }

    location = /get-health {
        access_log off;
        default_type text/html;
        return 200 'alive';
    }

    # Pass requests that don't refer directly to files in the filesystem to index.php
    location / {
    	try_files $uri $uri/ /index.php?$query_string;
    }

    # The following directives are based on best practices from H5BP Nginx Server Configs
    # https://github.com/h5bp/server-configs-nginx

    # Expire rules for static content
    location ~* \.(?:manifest|appcache|html?|xml|json)$ {
        add_header Cache-Control "max-age=0";
    }

    location ~* \.(?:rss|atom)$ {
        add_header Cache-Control "max-age=3600";
    }

    location ~* \.(?:jpg|jpeg|gif|png|ico|cur|gz|svg|mp4|ogg|ogv|webm|htc)$ {
        add_header Cache-Control "max-age=2592000";
        access_log off;
    }

    location ~* \.(?:css|js)$ {
        add_header Cache-Control "max-age=31536000";
        access_log off;
    }

    location ~* \.(?:ttf|ttc|otf|eot|woff|woff2)$ {
        add_header Cache-Control "max-age=2592000";
        access_log off;
    }

    # Gzip 交给 Traefik , 不针对文件类型做压缩
}



```

上面配置中的 `server_name` 需要改为你的目标站点名称。将两个配置文件保存好后，我们继续处理容器编排文件，完整的内容如下：

```yaml
version: "3.6"

services:

  nginx:
    image: ${DOCKER_NGINX_IMAGE}
    restart: always
    expose:
      - 80
    volumes:
      - ./logs:/var/log/nginx
      - ./conf/docker-nginx.conf:/etc/nginx/nginx.conf
      - ./conf/vhost.conf:/etc/nginx/conf.d/vhost.conf
      - ./wwwroot:/wwwroot
    links:
      - php:php
    extra_hosts:
      - "${DOCKER_DOMAIN_NAME}:127.0.0.1"
    networks:
      - traefik
    labels:
      - "traefik.enable=true"
      - "traefik.docker.network=traefik"
      - "traefik.http.routers.www-flarum.middlewares=https-redirect@file"
      - "traefik.http.routers.www-flarum.entrypoints=http"
      - "traefik.http.routers.www-flarum.rule=Host(`$DOCKER_DOMAIN_NAME`)"
      - "traefik.http.routers.ssl-flarum.middlewares=content-compress@file"
      - "traefik.http.routers.ssl-flarum.entrypoints=https"
      - "traefik.http.routers.ssl-flarum.tls=true"
      - "traefik.http.routers.ssl-flarum.rule=Host(`$DOCKER_DOMAIN_NAME`)"
      - "traefik.http.services.ngx-flarum-backend.loadbalancer.server.scheme=http"
      - "traefik.http.services.ngx-flarum-backend.loadbalancer.server.port=80"
    healthcheck:
      test: ["CMD-SHELL", "wget -q --spider --proxy off http://${DOCKER_DOMAIN_NAME}/get-health || exit 1"]
      interval: 5s
      retries: 12
    logging:
        driver: "json-file"
        options:
            max-size: "10m"

  php:
    image: ${DOCKER_PHP_IMAGE}
    restart: always
    expose:
      - 9000
    env_file: .env
    volumes:
      - ./logs:/var/log
      - ./wwwroot:/wwwroot
    networks:
      - traefik
    healthcheck:
      test: ["CMD-SHELL", "pidof php-fpm"]
      interval: 5s
      retries: 12
    logging:
      driver: "json-file"
      options:
        max-size: "10m"

networks:
  traefik:
    external: true
```

与之对应的 `.env` 配置内容：

```Toml
DOCKER_PHP_IMAGE=soulteary/flarum:v0.1.0-beta.12
DOCKER_NGINX_IMAGE=nginx:1.17.1-alpine

DOCKER_DOMAIN_NAME=lab.com

FLARUM_DB_HOST=flarum-db.lab.com
FLARUM_DB_NAME=flarum
FLARUM_DB_USER=flarum
FLARUM_DB_PASS=flarum

FLARUM_REDIS_HOST=flarum-redis.lab.com
FLARUM_REDIS_PORT=6379
FLARUM_REDIS_PASS=
FLARUM_REDIS_DBNO=0
FLARUM_REDIS_CONNECTION_PARAMETERS=
FLARUM_REDIS_CLIENT_OPTIONS=
FLARUM_REDIS_PREFIX=session:
FLARUM_REDIS_LOCKING=1
FLARUM_REDIS_SPIN_LOCK_WAIT=150000
FLARUM_REDIS_HANDLER_OPTIONS=
FLARUM_REDIS_TTS=3600

FLARUM_APP_DEBUG=true
FLARUM_APP_URL=//lab.com

```

同样使用 `docker-compose up -d` 启动服务，然后就能看到久违的安装界面了。

![又见面了，熟悉的Flarum安装界面](https://attachment.soulteary.com/2020/03/15/flarum-see-u-again.png)

参考上图和上面的 `.env` 配置，就能够完成 flarum 的安装了。

![安装完毕](https://attachment.soulteary.com/2020/03/15/install-finish.png)

## 安装收尾

默认安装完毕之后，会在 `wwwroot/config.php`  路径生成配置文件：

```php
<?php return array (
  'debug' => false,
  'database' => 
  array (
    'driver' => 'mysql',
    'host' => 'flarum-db.lab.com',
    'port' => 3306,
    'database' => 'flarum',
    'username' => 'flarum',
    'password' => 'flarum',
    'charset' => 'utf8mb4',
    'collation' => 'utf8mb4_unicode_ci',
    'prefix' => 'flarum_',
    'strict' => false,
    'engine' => 'InnoDB',
    'prefix_indexes' => true,
  ),
  'url' => 'https://lab.com',
  'paths' => 
  array (
    'api' => 'api',
    'admin' => 'admin',
  ),
);
```

为了更好的进行动态配置和代码集中管理，这里可以使用环境变量替换掉 hard code 的内容。

```php
<?php return array (
  'debug' => ($_SERVER['FLARUM_APP_DEBUG'] === 'true'),
  'database' =>
  array (
    'driver' => 'mysql',
    'host' => $_SERVER['FLARUM_DB_HOST'],
    'database' => $_SERVER['FLARUM_DB_NAME'],
    'username' => $_SERVER['FLARUM_DB_USER'],
    'password' => $_SERVER['FLARUM_DB_PASS'],
    'charset' => 'utf8mb4',
    'collation' => 'utf8mb4_unicode_ci',
    'prefix' => 'flarum_',
    'port' => '3306',
    'strict' => false,
    'engine' => 'InnoDB',
    'prefix_indexes' => true,
  ),
  'url' => $_SERVER['FLARUM_APP_URL'],
  'paths' =>
  array (
    'api' => 'api',
    'admin' => 'admin',
  ),
);
```

完成 config.php 文件的修改后，便可以将文件同样提交至代码仓库，进行保存管理。

另外，因为程序最初设计未考虑到容器环境，所以对于运行目录并没有单独进行设计，导致简单封装后，需要对目录进行权限调整；以及初次安装完毕后，从远端下载的前端资源文件，如果清理缓存后会导致界面不正常；以及前文提到的反复更新软件相关插件...

所以我们需要准备一个启动脚本：**start.sh**

```bash
#!/usr/bin/env bash

# 关闭服务
echo "尝试停止之前启动的服务"
docker-compose down --remove-orphans


# 确保容器镜像存在
cat .env | grep _IMAGE | cut -d '=' -f2 | while read image ; do
  if test -z "$(docker images -q $image)"; then
    echo "获取基础镜像"
    docker pull $image
  fi
done

# 清理之前存在的历史文件，并将新文件同步到执行目录中
if [ -d "wwwroot/vendor" ]; then
    echo "清理已经存在的 Vendor 文件"
    rm -rf wwwroot/vendor
fi
if [ -d "../codebase" ]; then
    echo "同步 Vendor 文件"
    cp -r ../codebase/vendor/ wwwroot/
fi

echo '备份前端资源'
cp -r wwwroot/public/assets/* ./backup-assets
mkdir -p ./backup-assets

# 清理 & 重建目录
echo '清理缓存目录'
rm -rf ./wwwroot/storage
mkdir -p ./wwwroot/storage/cache
mkdir -p ./wwwroot/storage/formatter
mkdir -p ./wwwroot/storage/less
mkdir -p ./wwwroot/storage/locale
mkdir -p ./wwwroot/storage/logs
mkdir -p ./wwwroot/storage/sessions
mkdir -p ./wwwroot/storage/tmp
mkdir -p ./wwwroot/storage/views

echo '还原前端资源'
rm -rf ./wwwroot/public/assets
mkdir -p ./wwwroot/public/assets
cp -r ./backup-assets/* ./wwwroot/public/assets/

# 重启服务
echo '重启服务'
docker-compose up -d

# 修正文件权限
docker ps -q -f status=running -f name=_php_1 |  while read container ; do
   echo "修正容器 $container 权限"
   docker exec $container chown -R www-data:www-data /wwwroot
   docker exec $container chmod -R 755 /wwwroot/storage
   docker exec $container chmod -R 755 /wwwroot/public/assets
done
```

## 支持中日韩文字搜索

其实更好的方案是使用 ELK 插件去做全文检索，但是其实使用 MySQL 开启全文检索，对于访问量不大的站点影响不大。

尤其是在几乎不需要付出额外的成本，使用的机器资源也相对较低的前提下。

随手写一个 PHP 脚本，执行下面两条命令，稍等片刻，flarum 就能够支持中文、日文的索引了。

```sql
ALTER TABLE flarum_posts DROP INDEX content;
CREATE FULLTEXT INDEX content ON `flarum_posts` (`content`) WITH PARSER ngram;

ALTER TABLE flarum_discussions DROP INDEX title;
CREATE FULLTEXT INDEX title ON `flarum_discussions` (`title`) WITH PARSER ngram;
```

## 卸载插件

为了提高可维护性，这里建议对不使用的插件进行卸载，以 ``flarum/auth-twitter` 和 `flarum/auth-facebook`` 为例，在 **codebase** 目录执行：

```bash
composer remove flarum/auth-twitter flarum/auth-facebook
```

成功卸载插件，将看到下面的提示。

```TeXT
Dependency "flarum/core" is also a root requirement, but is not explicitly whitelisted. Ignoring.
Dependency "flarum/core" is also a root requirement, but is not explicitly whitelisted. Ignoring.
Loading composer repositories with package information
Updating dependencies (including require-dev)
Package operations: 0 installs, 0 updates, 4 removals
  - Removing league/oauth2-facebook (2.0.1)
  - Removing league/oauth1-client (1.7.0)
  - Removing flarum/auth-twitter (v0.1.0-beta.12)
  - Removing flarum/auth-facebook (v0.1.0-beta.12)
Writing lock file
Generating autoload files
```

完成之后，记得重新执行上面的启动脚本，将程序源码做一次新的同步。

## 支持中文语言包

有一位网友(@csineneo)做的中文语言包，质量相对比较不错，和上面卸载无用插件类似，使用 composer 进行安装：

```bash
composer require csineneo/lang-simplified-chinese
```

安装完毕之后，将看到类似日志。

```TeXT
Using version ^1.12 for csineneo/lang-simplified-chinese
./composer.json has been updated
Loading composer repositories with package information
Updating dependencies (including require-dev)
Package operations: 1 install, 0 updates, 0 removals
  - Installing csineneo/lang-simplified-chinese (1.12.4): Downloading (100%)         
Writing lock file
Generating autoload files
```

同样的，需要在再次执行启动脚本，同步更新过的代码到flarum运行目录。

## 最后

除了搭建 Flarum 主体外，完成持续集成的环境也很重要，可以考虑使用[之前这篇文章](https://soulteary.com/2020/02/04/gitea-git-server-with-docker-and-traefik-v2.html)中的方案一个 Git Server，配置自动部署。

或许未来我会聊聊在十个月前，我们是如何对 Flarum 进行调整，使它适合用于多机环境的，以及如何打通微信扫码登录、如何使用更靠谱的附件上传...

至于什么时候写，或许得等到再有群友提问，能激发起我的兴趣的时候吧，:)

--EOF

