# 使用 Docker 搭建你自己的 RSS 服务（FreshRSS）

在算法推荐满天飞的世界里，定制获取信息就显得比较另类了，但是它可能是更高效的手段。

接下来我将演示三种常规的 RSS 服务的搭建方式，本篇是第一篇，FreshRSS。

如果你有阅读过我之前写的文章，那么参考本篇文章将文章搭建起来，应该只需要3分钟或者更少，如果你没有看过，那么可以点击本文相关的文章标签，阅读过往的文章。

## 关于 FreshRSS

FreshRSS 是一款基于 PHP 编写的 RSS 服务。相比较同是由 PHP 编写、名气更大的 **TT-RSS**，FreshRSS 的因为持续耕耘 GitHub 开源社区，功能和迭代保持的更好。

我之前使用它的主要原因有：

- 支持离线缓存，包括图片离线访问（需要使用 ImageProxyExtension 插件）。
- 支持 Fever API，允许用户在客户端阅读器上进行阅读。
- 支持插件，也方便用户编写插件进行定制化使用。

如果你想了解更多，可以访问[这里](https://github.com/FreshRSS/FreshRSS)，如果你只是想使用，那么请继续阅读。

## 使用 Docker 和 Traefik 进行服务

在本文成文的时候，我发现官方社区在十几天前也有人提交了如何使用 `Traefik` 搭建服务，不过，显然我提供的方案更简单一些，[关于这次提交](https://github.com/FreshRSS/FreshRSS/pull/2189)。

下面是我提供的服务应用配置，考虑到服务的可维护性，这里我将数据库和应用进行了拆分，如果你喜欢 `bundle` ，可以将两个配置进行合并。

```yaml
version: '3'

services:

  nginx:
    image: freshrss/freshrss:1.13.0
    restart: always
    container_name: freshrss
    environment:
      CRON_MIN: 17,47
    networks:
      - traefik
    labels:
      - "traefik.enable=true"
      - "traefik.frontend.rule=Host:rss.lab.com"
      - "traefik.frontend.entryPoints=http,https"
    volumes:
      - ./data:/var/www/FreshRSS/data
      - ./extensions:/var/www/FreshRSS/extensions

networks:
  traefik:
    external: true

```

使用 `docker-compose up` 将服务运行起来之后，我们继续折腾数据库，下面是数据库配置。

```yaml
version: '3'

services:

  mariadb:
    image: mariadb:10.3.8
    restart: always
    container_name: rss-db
    networks:
      - traefik
    environment:
      MYSQL_DATABASE: freshrss
      MYSQL_USER: freshrss
      MYSQL_PASSWORD: pass
      MYSQL_ROOT_PASSWORD: soulteary
    volumes:
      - ./data:/var/lib/mysql
    labels:
      - "traefik.enable=false"

  phpmyadmin:
    image: phpmyadmin/phpmyadmin:4.8.2
    restart: always
    networks:
      - traefik
    environment:
      MYSQL_USER: ttrss
      MYSQL_PASSWORD: ttrss
      MYSQL_ROOT_PASSWORD: soulteary
      PMA_HOST: rss-db
    labels:
      - "traefik.frontend.rule=Host:rss-pma.lab.com"
      - "traefik.enable=true"

networks:
  traefik:
    external: true
```

同样的，使用 `docker-compose up` 将服务运行起来，打开浏览器，访问：`rss.lab.com` ，简单配置之后，你就能够拥有一个功能强大，界面友好的 RSS 订阅服务了。

![FreshRSS 最终结果预览](https://attachment.soulteary.com/2019/01/05/freshrss.png)

## 最后

之前写文章总是考虑没有阅读基础的同学，而忽略了一直订阅、关注着我的同学，未来重复的内容，我将会和本文一样，给予简短的指引，不赘述基础建设，只聊主题相关的核心部分。

虽然这个服务搭建完毕了，但是并不能很好的服务于我们，因为在当前的网络大环境下，越来越多的网站“被迫封闭了起来”，不再支持 RSS 方式的订阅模式，至于如何解决，请耐心等待这三篇文章结束后，我提供的方案吧。

— EOF