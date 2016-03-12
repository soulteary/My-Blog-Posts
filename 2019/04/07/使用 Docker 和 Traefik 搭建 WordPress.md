# 使用 Docker 和 Traefik 搭建 WordPress

其实不止一次想重新提笔聊聊 **WordPress** ，然而之前因为定制代码量比较多，许多文章不得不搁置在草稿箱中。恰逢假期，整理草稿箱，从搭建开始聊起吧。

本文将使用 Docker、Compose、Traefik 对 WordPress 进行搭建，完整操作时间应该在十分钟内。



## 为什么选择 WordPress

每当聊起 CMS 类软件，聊起社区资源丰富，不由地会想到一个“万金油”：**WordPress** ，官方数据称：

> Over 60 million people have chosen WordPress to power the place on the web they call “home” 
> 
> Hundreds of thousands of developers, content creators, and site owners gather at monthly meetups in 436 cities worldwide.
> 
> WordPress 为 33% 的互联网提供支持。

许多人对它的印象还停留在执行速度慢、安全性差、代码臃肿的博客系统上。但是事实上，经过十几年的迭代，它的大版本来到了 5.0 （PHP 主流运行时也来到了 7.0 时代），性能早已不是问题、安全问题只要做适当的防护能杜绝绝大多数。

_emmm, 代码确实还是挺臃肿的。_

## 基于官方镜像

官方提供了[容器镜像](https://hub.docker.com/_/wordpress)，镜像下载可以直接使用下面的命令：

```bash
docker pull wordpress
```

但是为了更好的配置使用，我们使用 `compose` 的方式进行编排，将下面的内容保存为 `docker-compose.yml` ：

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

  pma:
    image: ${PMA_IMAGE}
    restart: always
    networks:
      - traefik
    environment:
      MYSQL_USER: ${DB_USER}
      MYSQL_PASSWORD: ${DB_PASS}
      MYSQL_ROOT_PASSWORD: ${DB_ROOT_PASS}
      PMA_HOST: ${DB_HOST}
    labels:
      - "traefik.enable=true"
      - "traefik.frontend.rule=Host:${PMA_DOMAIN}"

networks:
  traefik:
    external: true
```

如果你还不会使用 Traefik ，可以翻看我之前的文章，这里不做过多赘述。

为了可维护性，我们将容器镜像版本信息，应用域名，数据库配置等抽象为单独的环境配置文件 `.env`，内容示例：

```TeXT
WP_IMAGE=wordpress:5.1.1-php7.3-apache
WP_DOMAINS=wp.lab.com,wp.lab.io
WP_DB_PREFIX=wp

DB_IMAGE=mariadb:10.3.8
DB_HOST=wp-db
DB_NAME=wordpress
DB_USER=wordpress
DB_PASS=wordpress
DB_ROOT_PASS=soulteary

PMA_IMAGE=phpmyadmin/phpmyadmin:4.8.2
PMA_DOMAIN=pma.wp.lab.com,pma.wp.lab.io
```

当两个文件都保存完毕之后，我们执行 `docker-compose up` 命令，你将会看到许多日志信息，当看到类似下面的信息时，WordPress 环境便准备就绪啦。

```TeXT
wp-db      |
wp-db      | MySQL init process done. Ready for start up.
wp-db      |
wp-db      | 2019-04-06 16:26:48 0 [Note] mysqld (mysqld 10.3.8-MariaDB-1:10.3.8+maria~jessie) starting as process 1 ...
wp-db      | 2019-04-06 16:26:48 0 [Note] InnoDB: Using Linux native AIO
wp-db      | 2019-04-06 16:26:48 0 [Note] InnoDB: Mutexes and rw_locks use GCC atomic builtins
wp-db      | 2019-04-06 16:26:48 0 [Note] InnoDB: Uses event mutexes
wp-db      | 2019-04-06 16:26:48 0 [Note] InnoDB: Compressed tables use zlib 1.2.8
wp-db      | 2019-04-06 16:26:48 0 [Note] InnoDB: Number of pools: 1
wp-db      | 2019-04-06 16:26:48 0 [Note] InnoDB: Using SSE2 crc32 instructions
wp-db      | 2019-04-06 16:26:48 0 [Note] InnoDB: Initializing buffer pool, total size = 256M, instances = 1, chunk size = 128M
wp-db      | 2019-04-06 16:26:48 0 [Note] InnoDB: Completed initialization of buffer pool
wp-db      | 2019-04-06 16:26:48 0 [Note] InnoDB: If the mysqld execution user is authorized, page cleaner thread priority can be changed. See the man page of setpriority().
wp-db      | 2019-04-06 16:26:48 0 [Note] InnoDB: 128 out of 128 rollback segments are active.
wp-db      | 2019-04-06 16:26:48 0 [Note] InnoDB: Creating shared tablespace for temporary tables
wp-db      | 2019-04-06 16:26:48 0 [Note] InnoDB: Setting file './ibtmp1' size to 12 MB. Physically writing the file full; Please wait ...
wp-db      | 2019-04-06 16:26:48 0 [Note] InnoDB: File './ibtmp1' size is now 12 MB.
wp-db      | 2019-04-06 16:26:48 0 [Note] InnoDB: 10.3.8 started; log sequence number 1630833; transaction id 21
wp-db      | 2019-04-06 16:26:48 0 [Note] Plugin 'FEEDBACK' is disabled.
wp-db      | 2019-04-06 16:26:48 0 [Note] InnoDB: Loading buffer pool(s) from /var/lib/mysql/ib_buffer_pool
wp-db      | 2019-04-06 16:26:48 0 [Note] Server socket created on IP: '::'.
wp-db      | 2019-04-06 16:26:48 0 [Note] InnoDB: Buffer pool(s) load completed at 190406 16:26:48
wp-db      | 2019-04-06 16:26:48 0 [Warning] 'proxies_priv' entry '@% root@e97787886b74' ignored in --skip-name-resolve mode.
wp-db      | 2019-04-06 16:26:48 0 [Note] Reading of all Master_info entries succeded
wp-db      | 2019-04-06 16:26:48 0 [Note] Added new Master_info '' to hash table
wp-db      | 2019-04-06 16:26:48 0 [Note] mysqld: ready for connections.
wp-db      | Version: '10.3.8-MariaDB-1:10.3.8+maria~jessie'  socket: '/var/run/mysqld/mysqld.sock'  port: 3306  mariadb.org binary distribution
wp_1       | AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 172.23.0.8. Set the 'ServerName' directive globally to suppress this message
```

此时启动浏览器，打开我们配置文件中配置好的域名（`WP_DOMAIN`），便可以开始著名的“三分钟”安装了。

![环境就绪，可以开始安装了](https://attachment.soulteary.com/2019/04/07/install-start.png)

填写适当信息，一路 Next ，WordPress 就安装成功了。

![WordPress 安装就绪](https://attachment.soulteary.com/2019/04/07/install-complete.png)

后续便是具体的应用配置，以及性能、安全方面的优化啦。

![WordPress 欢迎界面](https://attachment.soulteary.com/2019/04/07/welcome.png)

## 其他

如果你有操作数据库的需求，又不想下载数据库工具或者使用命令行进行操作，可以使用 **PHPMyAdmin ** ，同样的，在浏览器中打开之前配置文件中的 PMA 域名地址（`PMA_DOMAIN`），就可以进行操作了。

不过需要注意的是，需要使用 `root` 和 `root password` 进行登录，因为默认情况下，Mariadb 未对其他用户账号进行远程访问授权。

![数据库管理界面](https://attachment.soulteary.com/2019/04/07/pma.png)

## 最后

作为重新提笔 WordPress 第一篇，内容看起来会简单一些，还请见谅，下一篇将以本篇为基础进行扩展，聊聊容器化部署的细节。