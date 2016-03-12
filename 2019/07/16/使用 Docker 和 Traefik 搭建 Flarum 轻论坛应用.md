# 使用 Docker 和 Traefik 搭建 Flarum 轻论坛应用

最近在做社区类型的项目，出于后续从市场招人成本的考虑，不得不优选市场招聘培养难度较低的 PHP，再三挑选，选择了这款还在 *beta* 状态的软件。

这是一款 Beta 了差不多 5 年的软件，在 GitHub 上拥有接近一万 star 的耀眼成绩，[第一条提交记录](https://github.com/flarum/flarum/commit/8293099b25312176d87b08f0b9ddc1fcfa75f18c) 是2014年末的  v0.1.0-beta 发布。

本文将介绍使用容器如何简单快速的搭建 Flarum ，如果你还不熟悉 Traefik，请翻阅[之前的文章](https://soulteary.com/tags/traefik.html)。

## 写在前面

关于选型的顾虑，我想此刻看到文章的你，也一定有所考虑。

前面提到这款软件还在 beta ，处于不是特别稳定的状况。而且前一阵主要的贡献者在论坛里发布了一条消息，宣布“[farewell](https://discuss.flarum.org/d/20590-farewell-and-what-s-next-for-flarum)”，看起来是情况不是特别乐观，那么除了招聘维护成本的考虑之外，为什么还要选择它呢：

- 交互体验和项目架子搭的还不错，基本面做的都还行。
- 作为开源应用迭代了五年，有提交记录和开放的代码，程序不是特别复杂，相对好追溯和解决问题。
- 项目文档虽然不全，但是基础的部分还差不多是有的。
- 应用市场虽然东西不多，但是插件机制已经建设好了，功能扩展相对比较简单。
- 同语言实现的、功能比较强大的某两款国产软件，一个论坛停止运营，一个计划重构（原因你懂的），显然当当下时间点都不值得托付。
- WordPress 的 BuddyPress 生态可以考虑，但是做同样需求改动量会更大一些，因为冗余内容更多。

## 构建 PHP 容器

官方[安装文档](https://flarum.org/docs/install.html#server-requirements)对于环境要求是这样的

- 支持 URL Rewrite 的服务器软件：Apache、Nginx…
- PHP 7.1+，以及 `dom`、 `gd`、 `json`、 `mbstring`、 `openssl`, 、 `pdo_mysql`、 `tokenizer` 组件。
- `MySQL 5.6+` 或 `MariaDB 10.0.5+`
- `Composer`

所以，Docker Hub 默认的提供的 PHP 镜像是使用不了的，需要进行额外配置，安装以上需要的软件。我们默认不允许在已经运行起来的软件中执行插件卸载（删除程序部分文件），所以最后一点提到的 composer 可以在本地安装，或者干脆不进行安装。

这里以 `PHP-FPM-ALPINE` 7.3.2 为例：

```bash
FROM php:7.3.2-fpm-alpine

ENV LANG en_US.UTF-8
ENV LANGUAGE en_US.UTF-8
ENV LC_ALL=en_US.UTF-8

RUN echo '' > /etc/apk/repositories && \
    echo "https://mirror.tuna.tsinghua.edu.cn/alpine/v3.9/main"         >> /etc/apk/repositories && \
    echo "https://mirror.tuna.tsinghua.edu.cn/alpine/v3.9/community"    >> /etc/apk/repositories && \
    echo "Asia/Shanghai" > /etc/timezone

RUN apk --no-cache --no-progress update && \
    apk --no-cache --no-progress upgrade

RUN apk add libpng libpng-dev libjpeg-turbo-dev libwebp-dev zlib-dev libxpm-dev

RUN docker-php-ext-install pdo pdo_mysql mbstring gd

ENTRYPOINT ["docker-php-entrypoint"]

STOPSIGNAL SIGQUIT

EXPOSE 9000
CMD ["php-fpm"]
```

使用 `build` 命令将容器构建起来：

```bash
docker build -t php-fpm-flarum:7.3.2  -f Dockerfile .
```

使用 `docker images` 查看构建后的 PHP 镜像，一百兆出头。

```bash
REPOSITORY                               TAG                     IMAGE ID            CREATED             SIZE
php-fpm-flarum                           7.3.2                   ef5ad124a35d        3 minutes ago       105MB
```

## 搭建数据库

上一小节有提到过，官方对于数据库方面的要求是：`MySQL 5.6+` 或 `MariaDB 10.0.5+`，所以你可以根据自己喜好来搞，如果有性能要求，建议使用云厂商的 RDS 产品。

下面给出一个数据库编排文件示例：

```yaml
version: '3.6'

services:

  database:
    image: ${DB_IMAGE}
    restart: always
    container_name: ${DB_HOST}
    ports:
      - 3306:3306
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

将上面的配置保存为 `docker-compose.yml` ，继续编写配置需要的 `.env` 文件。我这边为了测试方便，就都用弱密码了，实际使用应改成安全度较高的复杂密码。

```yaml
DB_IMAGE=mysql:5.7.26
DB_HOST=flarum
DB_NAME=flarum
DB_USER=flarum
DB_PASS=flarum
DB_ROOT_PASS=flarum
```

然后执行 `docker-compose up -d` 数据库就运行起来啦。

## 搭建应用运行框架

时至今日，官方提供的安装方案也从传统的软件压缩包变成了一条简约的命令：

```bash
composer create-project flarum/flarum . --stability=beta
```

但是这样做对于持续迭代的项目的开发部署体验是不友好的，即使使用版本锁定功能，项目构建还是时间会变长，把代码都打到容器里由显得笨重，并且造成了相同应用的代码割裂，维护成本颇高。

而且后续需要在程序框架上做一些改动，还要解决和未来的版本更新合并的问题，并不只是简单的安装使用就完事了，所以这里需要将应用代码储存下来。

### 安装 Composer PHP 包管理软件

因为软件发布模式变化，所以我们下载软件包需要使用 Composer (PHP 环境安装不赘述)。

```bash
wget -O composer-setup.php https://getcomposer.org/installer

php composer-setup.php --install-dir=bin --filename=composer
```

Composer 在国内下载比较慢，这里可以选择借助[阿里云的加速镜像](https://developer.aliyun.com/composer)，使用方法很简单，就一条命令。

```bash
composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/
```

### 下载 Flarum 程序代码

接着使用下面的命令将软件下载至 flarum  目录。

```bash
`composer create-project flarum/flarum ./flarum --stability=beta`
```

完成安装后的目录结构是下面的样子：

```TeXT
flarum
├── CHANGELOG.md
├── LICENSE
├── README.md
├── composer.json
├── composer.lock
├── extend.php
├── flarum
├── public
│   ├── assets
│   └── index.php
├── storage
│   ├── cache
│   ├── formatter
│   ├── less
│   ├── locale
│   ├── logs
│   ├── sessions
│   ├── tmp
│   └── views
└── vendor
```

现在软件还运行不起来，我们需要继续配置应用的运行环境。

### 配置软件运行环境

在上面的操作都就绪之后，我们就可以进行程序运行环境搭建了。这里使用 Nginx 作为 PHP 的前端，整个环境搭建非常简单。

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
      - ./wwwroot:/wwwroot
    links:
      - php:php
    extra_hosts:
      - "${DOCKER_DOMAIN_NAME}:127.0.0.1"
    networks:
      - traefik
    labels:
      - "traefik.enable=true"
      - "traefik.port=80"
      - "traefik.frontend.rule=Host:${DOCKER_DOMAIN_NAME}"
      - "traefik.frontend.entryPoints=https,http"
    healthcheck:
      test: ["CMD-SHELL", "wget -q --spider --proxy off http://${DOCKER_DOMAIN_NAME}/get-health || exit 1"]
      interval: 5s
      retries: 12
    logging:
        driver: "json-file"
        options:
            max-size: "100m"

  php:
    image: ${DOCKER_PHP_IMAGE}
    restart: always
    expose:
      - 9000
    volumes:
      - ./logs:/var/log
      - ./wwwroot:/wwwroot
    extra_hosts:
      - "${DOCKER_DOMAIN_NAME}:127.0.0.1"
    networks:
      - traefik
    healthcheck:
      test: ["CMD-SHELL", "pidof php-fpm"]
      interval: 5s
      retries: 12
    logging:
      driver: "json-file"
      options:
        max-size: "100m"

networks:
  traefik:
    external: true
```

搭配使用的 `.env` 可以这么写：

```yaml
DOCKER_DOMAIN_NAME=flarum.lab.com
DOCKER_PHP_IMAGE=php-fpm-flarum:7.3.2
DOCKER_NGINX_IMAGE=nginx:1.17.1-alpine
```

Nginx 的配置则可以参考下面的文件：

```TeXT
user nginx;
worker_processes 1;
error_log /var/log/nginx/error.log warn;
pid /var/run/nginx.pid;
events { worker_connections 1024; }

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;
    log_format main '$http_x_forwarded_for - $remote_user [$time_local] "$request" $status $body_bytes_sent "$http_referer" "$http_user_agent"';
    access_log off;
    sendfile on;
    keepalive_timeout 65;

    server {
        listen 80;
        server_name flarum.lab.com;
        server_tokens off;
        access_log /var/log/nginx/docker-access.log;
        error_log /var/log/nginx/docker-error.log;
        root /wwwroot/public;
        index index.php index.html;

        location ~ \.php$ {
            try_files $uri =404;
            fastcgi_split_path_info ^(.+\.php)(/.+)$;
            fastcgi_pass php:9000;
            fastcgi_index index.php;
            include fastcgi_params;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            fastcgi_param PATH_INFO $fastcgi_path_info;
        }

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
        location = /get-health {
            access_log off;
            default_type text/html;
            return 200 'alive';
        }
        # 可以考虑交给后面的程序去处理，比如 traefik ...
        # Gzip compression
        gzip on;
        gzip_comp_level 5;
        gzip_min_length 256;
        gzip_proxied any;
        gzip_vary on;
        gzip_types  application/atom+xml
                    application/javascript
                    application/json
                    application/ld+json
                    application/manifest+json
                    application/rss+xml
                    application/vnd.geo+json
                    application/vnd.ms-fontobject
                    application/x-font-ttf
                    application/x-web-app-manifest+json
                    application/xhtml+xml
                    application/xml
                    font/opentype
                    image/bmp
                    image/svg+xml
                    image/x-icon
                    text/cache-manifest
                    text/css
                    text/plain
                    text/vcard
                    text/vnd.rim.location.xloc
                    text/vtt
                    text/x-component
                    text/x-cross-domain-policy;
    }
}
```

将之前下载完毕的 flarum 程序放置到 `wwwroot` 目录中，便可以开始程序的安装了。

使用 `docker-compose up -d` 将程序运行起来，访问 `flarum.lab.com` 对程序进行配置。

## 安装应用

简单填写安装界面需要的要素后，点击安装按钮。

![亲切的安装界面](https://attachment.soulteary.com/2019/07/16/install.png)

片刻之后，程序安装就完毕了，可以看到界面还是十分清爽的。

![清爽的界面](https://attachment.soulteary.com/2019/07/16/web-ui.png)

管理后台比较简单，很难满足我们的日常使用需求，不过好在现在可以编写插件进行功能增强。

![管理后台](https://attachment.soulteary.com/2019/07/16/dashboard.png)

## 对程序运行框架做优化

程序的安装部分已经结束了，但是考虑到后续的维护，我们还需要做一些额外的工作。

### 让应用自适应运行环境

应用安装完毕，我们再次查看应用目录，发现目录中多了一个 `config.php` 文件。

```TeXT
wwwroot
├── CHANGELOG.md
├── LICENSE
├── README.md
├── composer.json
├── composer.lock
├── config.php
├── extend.php
├── flarum
├── public
│   ├── assets
│   └── index.php
├── storage
│   ├── cache
│   ├── formatter
│   ├── less
│   ├── locale
│   ├── logs
│   ├── sessions
│   ├── tmp
│   └── views
└── vendor
```

文件内容如下，记录了程序运行配置：

```php
<?php return array (
  'debug' => false,
  'database' =>
  array (
    'driver' => 'mysql',
    'host' => 'flarum',
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
  'url' => 'https://flarum.lab.com',
  'paths' =>
  array (
    'api' => 'api',
    'admin' => 'admin',
  ),
);
```

考虑到程序可能运行在不同的环境中（生产、测试、开发），数据库、网站地址等信息存在变化的可能，如果它能够自动读取环境变量可以免除维护多份配置的麻烦事，并简化发布过程。

对它进行简单的修改：

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
  ),
  'url' => $_SERVER['FLARUM_APP_URL'],
  'paths' =>
  array (
    'api' => 'api',
    'admin' => 'admin',
  ),
);
```

同时也要对 `docker-compose.yml` 进行简单的修改，把 `.env` 当作环境变量“注入”到应用中。

```yaml
  php:
    image: ${DOCKER_PHP_IMAGE}
    restart: always
    expose:
      - 9000
    env_file: .env
```

最后，在 `.env` 里声明上面配置文件需要的变量名称即可。

```TeXT
DOCKER_DOMAIN_NAME=flarum.lab.com
DOCKER_PHP_IMAGE=php-fpm-flarum:7.3.2
DOCKER_NGINX_IMAGE=nginx:1.17.1-alpine

FLARUM_DB_HOST=flarum
FLARUM_DB_NAME=flarum
FLARUM_DB_USER=flarum
FLARUM_DB_PASS=flarum

FLARUM_APP_DEBUG=true
FLARUM_APP_URL=//flarum.lab.com
```

### 继续拆分项目结构

前文提过，如果我们要持续修改完善这个处于 beta 状态下的软件，需要对其代码进行保存维护，但是如果将环境和代码放置一处，修改调试的效率不免太低。

所以我们可以对上面的应用目录进行简化操作，将“应用代码”和“基础环境”进行拆分，未来调试发布仅需要更新文件即可，而不必对环境进行重新部署、重启等重操作。

如果你也有类似需求，也可以参考下图进行拆分，将应用软件的基础环境、Vendor 代码进行分拆。

![拆分项目结构](https://attachment.soulteary.com/2019/07/16/split-dir.png)


简化后的目录如下：

```TeXT
.
├── LICENSE
├── README.md
├── .env
├── conf
│   └── docker-nginx.conf
├── logs
├── update.sh
└── wwwroot
    ├── config.php
    ├── extend.php
    ├── flarum
    ├── public
    │   └── index.php
    ├── storage
    └── vendor
```

可以看到，`wwwroot` 目录被简化到只留下四个 PHP 文件，和几个空目录。

对程序进行修改发布，则可以使用 CI 配合 `update.sh` 更新脚本使用，将程序需要的 `vendor` 依赖文件进行同步更新，更新脚本可以参考下面：

```bash
#!/usr/bin/env bash

# 关闭服务
echo "stop services."
docker-compose down --remove-orphans

# 确保容器镜像存在
cat .env | grep _IMAGE | cut -d '=' -f2 | while read image ; do
  if test -z "$(docker images -q $image)"; then
    echo "prepare docker images."
    docker pull $image
  fi
done

# 清理之前存在的历史文件，并将新文件同步到执行目录中
if [ -d "wwwroot/vendor" ]; then
    echo "cleanup wwwroot/vendor."
    rm -rf wwwroot/vendor
fi
if [ -d "/data/forum-test-vendor" ]; then
    echo "update vendor."
    cp -r /data/forum-test-vendor/vendor/ wwwroot/
    cp /data/forum-test-vendor/composer.* wwwroot/
fi


# 清理 & 重建目录
echo 'cleanup directory'
rm -rf ./wwwroot/storage
mkdir -p ./wwwroot/storage/cache
mkdir -p ./wwwroot/storage/formatter
mkdir -p ./wwwroot/storage/less
mkdir -p ./wwwroot/storage/locale
mkdir -p ./wwwroot/storage/logs
mkdir -p ./wwwroot/storage/sessions
mkdir -p ./wwwroot/storage/tmp
mkdir -p ./wwwroot/storage/views

# 重启服务
echo 'restart services.'
docker-compose up -d

# 修正文件权限
docker ps -q -f status=running -f name=flarum_php |  while read container ; do
   echo "fix container $container own && mod."
   docker exec $container chown -R www-data:www-data /wwwroot
   docker exec $container chmod -R 755 /wwwroot/storage
done
```

在进行了目录拆分后，项目的测试发布时间将可以从之前的分钟级缩短到 5～10s 秒。

## 最后

关于这个软件的折腾细节还有不少，后续陆续慢慢写出来吧。

引用我前一段时间在朋友圈发的内容作为结束：

> 合作单位在关键时刻因不可抗力取消了合作，周五机器删除代码、数据库下线。所有人五十天的忙碌付诸东流，项目按计划上线压力骤大。

> 只好重定方案，从MVP做起。不得不再次变身救火队员👩‍🚒。

> 从收拾遗留问题、到做新架构设计、搭建各种环境和基础设施、调通后端基本流程，整了差不多三天。目测再搞一周应该就差不多啦！🎉

> 感谢各种开源软件，让问题变得快速可解，让我这个月还能空出时间办个“大事”。
