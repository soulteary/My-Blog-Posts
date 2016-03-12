# 搭建高性能的私有 Composer 镜像服务

最近在完善一个基于 Composer 管理的开源软件：Flarum 。

Flarum 是一款可以说是完全基于插件构成的社区系统，在需要对其频繁修改更新的开发过程中，我们需要频繁修改 `composer` 配置文件，在不断 `composer install` 的情况下，较慢的依赖下载会严重影响开发调试效率。

你可能会说，使用有良好网络质量的服务器进行初始化、或者使用企业商业网络高速网络通道、或者阿里云之类的公网镜像不就好了。然而这样做也仅仅只能保障分钟级别的部署安装。并且非常不利于多人多环境部署调试。

本文将试着提出一个更简单的解决方案，来解决这个问题。

## 写在前面

提高安装效率的手段其实并不多：

- 购买更优质的网络带宽、服务器资源
- 替换访问速度慢的资源
- 尽可能提高安装过程中的缓存利用率
- 将软件使用增量方式更新，减少传输数据量

考虑购买成本、开发、维护成本，一上来就购置顶级的专线、优化改进构建脚本使用缓存、将程序完全打包成镜像是不合理的，因为除了带来巨额成本外，还会带来一些意想不到的问题：缓存内容状态是否“健康”、缓存文件一致性如何保障、代码资源类容器的后续管理？

所以“替换访问速度比较慢的资源”无疑是低成本改善问题的不二之选。简单的使用公网资源，流量几经流转时间花费的比较多，内容稳定性也不好保障，故搭建私有镜像是一个比较好的解决方案。

下面就来讲讲私有镜像的搭建。

## 软件包安装模式的改变

使用镜像之前，composer 会从各种来源安装软件包，比如 GitHub、SVN、GitLab、Zip、tarball… 下载软件包时的网络访问质量是一个很难保障的事情，尤其是当我们需要同时访问不同服务商分布在天南海北的服务器的时候。

![使用镜像之前](https://attachment.soulteary.com/2019/08/23/before-mirror.jpg)

这个过程往往会持续几分钟甚至十几分钟，有的时候还会更多。最难过的是，如果我们需要多次部署安装，或者在新的服务器上进行安装时，这个时间损耗会不断放大，而且还不能够保障多台服务器安装结果一致，因为不确定软件包是否被完整下载。

而如果我们使用一个镜像服务将上述从各种地方获取的软件包提前获取，部署在距离我们需要安装软件包比较近的服务器上，时间损耗将可以有效控制在分钟级别以内，比如十几秒～几十秒。

![使用镜像之后](https://attachment.soulteary.com/2019/08/23/after-mirror.jpg)

## 选择提供镜像服务的应用

国内国外有不少开发者提供了 composer 的镜像工具，本文将使用官方出品的工具：satis 。

相比较国内外其他社区/团队出品的工具，这个工具更加的小巧，配合 CI 使用也非常简单，只需要修改 json 文件就能够完成软件包的管理。

搭配 Nginx 使用可以实现高性能的私有包仓库。

## 容器编排配置

 compose 配置如下：

```yaml
version: '3'

services:

  # 官方没有打 TAG，用 latest
  # https://hub.docker.com/r/composer/satis/tags
  composer:
    image: composer/satis:latest
    command: -vvv build /satis.json /wwwroot
    links:
      - nginx:nginx
    networks:
      - traefik
    volumes:
      - ./wwwroot:/wwwroot
      - ./composer:/composer
      - ./satis.json:/satis.json:ro
    depends_on:
      - nginx

  # repo web server
  nginx:
    image: nginx:1.15.10-alpine
    restart: always
    networks:
      - traefik
    expose:
      - 80
    volumes:
      - ./wwwroot:/var/www/html:ro
      - ./conf:/etc/nginx/conf.d:ro
      - ./logs:/var/log/nginx
    labels:
      - "traefik.enable=true"
      - "traefik.frontend.rule=Host:composer.lab.com"
      - "traefik.frontend.entryPoints=http,https"
    healthcheck:
      test: ["CMD-SHELL", "wget -q --spider --proxy off http://localhost || exit 1"]
      interval: 5s
      retries: 12

networks:
  traefik:
    external: true
```

可以看到我还是一如既往的选择了 Traefik 作为了服务网关，提供证书挂载和服务发现，如果你还不了解 Traefik，可以查看[历史文章](https://soulteary.com/tags/traefik.html)。

当然，如果你不希望使用 Traefik ，上面的配置中的 nginx 部分可以修改为下面这样（安装软件包时使用访问地址也要酌情修改哦）：

```yaml
# repo web server
nginx:
  image: nginx:1.15.10-alpine
  restart: always
  networks:
    - traefik
  ports:
    - 80:80
  volumes:
    - ./wwwroot:/var/www/html:ro
    - ./conf:/etc/nginx/conf.d:ro
    - ./logs:/var/log/nginx
```

上面配置中的 `nginx.conf` 配置，可以写的佛系一些，因为本来它也就只需要提供 Web 服务而已：

```TeXT
server {
    listen 80;

    root /var/www/html;
    index index.html;

    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;

    location /favicon.ico {
        access_log off;
    }
}
```

最后，参考下面的 `satis.json` 文件的内容，修改其中的 “url” 和 `require` 字段中的软件包，生成属于你的配置文件。

```json
{
    "name": "repo/dev",
    "homepage": "https://composer.lab.com",
    "repositories": [
        {
            "type": "composer",
            "url": "https://mirrors.aliyun.com/composer/"
        }
    ],
    "archive": {
        "directory": "dist",
        "format": "tar",
        "prefix-url": "https://composer.lab.com",
        "skip-dev": true
    },
    "require": {
        "php": ">=7.1",
        "flarum/approval": "^0.1.0",
        "flarum/auth-facebook": "^0.1.0",
        "flarum/auth-github": "^0.1.0",
        "flarum/auth-twitter": "^0.1.0",
        "flarum/tags": "^0.1.0",
        "flarum/core": "^0.1.0-beta.9",
        "predis/predis": "^1.1",
        "league/oauth2-client": "^2.4.1",
        "ramsey/uuid": "^3.5.2",
        "league/flysystem": "^1.0.32"
    },
    "require-dependencies": true,
    "require-dev-dependencies": true
}
```

接着使用 `docker-compose up`  启动服务，等待软件包被缓存完毕就可以正式使用了。

如果你的请求量很高，可以使用 `docker-compose scale nginx=4` 水平扩展几个实例，达到极高性能的需求：

```text
WARNING: The scale command is deprecated. Use the up command with the --scale flag instead.
Starting runner-prod-composer_nginx_1 ... done
Creating runner-prod-composer_nginx_2 ... done
...
```

如果你不满足只镜像你的项目依赖的包，希望进行全网全量软件包镜像，可以删除配置文件中的 `require` 字段。

```json
{
    "name": "repo/dev",
    "homepage": "https://composer.lab.com",
    "repositories": [
        {
            "type": "composer",
            "url": "https://mirrors.aliyun.com/composer/"
        }
    ],
    "archive": {
        "directory": "dist",
        "format": "tar",
        "prefix-url": "https://composer.lab.com",
        "skip-dev": true
    },
    "require-dependencies": true,
    "require-dev-dependencies": true
}
```

## 启动服务

使用 `docker-compose up` 启动服务，第一次启动时间会长一些，毕竟要从天南海北将软件包进行下载，如果一切顺利的话，你将看到类似下面的日志：

```TeXT
Creating runner-composer_nginx_1 ... done
Creating runner-composer_composer_1 ... done
Attaching to prod-composer_nginx_1, prod-composer_composer_1
composer_1  | Scanning packages
composer_1  | The php >=7.1 requirement did not match any package
composer_1  | Creating local downloads in '/wwwroot/dist'
composer_1  | Dumping package 'phpdocumentor/reflection-docblock' in version '2.0.0'.
...
...
composer_1  |   - Installing phpdocumentor/reflection-docblock (4.3.1): Downloading (100%)
composer_1  | Dumping package 'phpunit/phpunit' in version '4.8.36'.
composer_1  | Dumping package 'sebastian/recursion-context' in version '1.0.0'.
composer_1  | Dumping package 'doctrine/instantiator' in version '1.2.0'.
composer_1  |   - Installing doctrine/instantiator (1.2.0): Downloading (100%)
composer_1  | Dumping package 'phpdocumentor/reflection-docblock' in version '2.0.5'.
composer_1  | wrote packages to /wwwroot/include/all$20dc965137ca8bd578a65dbc0eb7138e6dfed6bc.json
composer_1  | Writing packages.json
composer_1  | Pruning include directories
composer_1  | Writing web view
prod-composer_composer_1 exited with code 0
```

当然，如果你使用的是全量软件包镜像模式，日志会类似下面这样：

```TeXT
Creating prod-composer_nginx_1 ... done
Creating prod-composer_composer_1 ... done
Attaching to prod-composer_nginx_1, prod-composer_composer_1
composer_1  | No explicit requires defined, enabling require-all
composer_1  | Scanning packages
...
...
prod-composer_composer_1 exited with code 0
```

一旦看到 `exited with code 0`，就说明软件包镜像完成，可以正式使用了。

## 使用私有镜像

如果你已经按照上文进行的配置，访问你定义的私有镜像仓库地址：`https://composer.lab.com`，你会看到类似下面的界面。

![镜像完毕的软件包状态](https://attachment.soulteary.com/2019/08/23/repo-website.png)

实际使用起来，还是很简单的，只需要在项目的 `composer.json` 的 `repositories` 字段中添加：

```json
{
...
    "repositories": [
        {
            "type": "composer",
            "url": "https://composer.lab.com",
            "options": {
                "ssl": {
                    "verify_peer": false,
                    "verify_peer_name": false,
                    "allow_self_signed": true
                }
            }
        },
        {
            "packagist.org": false
        }
    ]
}
```

上面的配置禁用了来自公网官方仓库的软件包获取，允许使用标准CA证书和自签名证书。

最后执行 `composer install` 进行软件包下载&安装即可。

## 最后

下一篇文章聊聊如何搭配 CI 系统，使用 `composer`。

—EOF
