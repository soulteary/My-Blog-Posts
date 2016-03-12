# 使用 Docker 搭建你自己的 RSS 服务（Miniflux）

在算法推荐满天飞的世界里，定制获取信息就显得比较另类了，但是它可能是更高效的手段。

本篇是我之前提到的三种常规的 RSS 服务搭建方式的第三篇，Miniflux。

如果你有阅读过我之前写的文章，那么参考本篇文章将文章搭建起来，应该只需要3分钟或者更少，如果你没有看过，那么可以点击本文相关的文章标签，阅读过往的文章。

## 关于 Miniflux

Miniflux 是一款基于 Go 编写的 RSS 服务。相比前两篇文章介绍的工具，它有以下特点：

- 程序设计极简，不处理任何订阅之外的事情。
- 程序无外部依赖，运行性能高。
- 支持自动抓取并缓存图片，加速浏览。
- 有限支持自动将摘要替换为全文进行抓取。
- 支持多账号登录，支持 Fever API ，允许客户端从外部登录。
- 支持集成 PinBoard 、Instapaper、 Pocket、Wallabag、Nunux Keeper 等服务。
- 提供 Open API、书签快速订阅脚本。
- 维护者和社区相对活跃，更新频率高。

但是它也有一些问题：

- 文档不够丰富，优化调试时，也需要翻代码。

如果你想了解更多，可以访问[这里](https://github.com/miniflux/miniflux)，如果你只是想使用，那么请继续阅读。

## 使用 Docker 和 Traefik 提供服务

官方代码版本更新比较勤快，可以使用官方容器镜像而无需二次封装新的镜像：`miniflux/miniflux:2.0.14` 。

下面是我提供的服务应用配置，定义了中文界面，RSS 资料缓存接近永久，应用升级版本时，自动升级并兼容新版本数据库字段。

```yaml
version: '3'

services:

  miniflux:
    image: miniflux/miniflux:2.0.14
    restart: always
    depends_on:
      - db
    expose:
      - 8080
    networks:
      - traefik
    environment:
      - BASE_URL=rss.orange.lab.com
      - ARCHIVE_READ_DAYS=36500
      - CLEANUP_FREQUENCY=36500
      - RUN_MIGRATIONS=1
      - CREATE_ADMIN=1
      - ADMIN_USERNAME=soulteary
      - ADMIN_PASSWORD=soulteary
      - PROXY_IMAGES=all
      - DATABASE_URL=postgres://miniflux:secret@db/miniflux?sslmode=disable
    labels:
      - "traefik.enable=true"
      - "traefik.port=8080"
      - "traefik.frontend.rule=Host:rss.lab.com"
      - "traefik.frontend.entryPoints=http,https"

  db:
    image: postgres:10.1-alpine
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

在使用 `docker-compose up` 将服务运行起来之后，我们打开浏览器，访问：`rss.lab.com` 。

使用配置中定义的管理员账号进行登录之后，你就能够拥有一个功能强大，界面友好的 RSS 订阅服务了。

推荐先进行界面设置，下面是我的配置，仅供参考。

![stringer rss 默认界面](https://attachment.soulteary.com/2019/01/22/miniflux-setting.png)

如果你希望手机、笔记本上进行同步阅读，可以配置 Fever API。

![stringer rss 默认界面](https://attachment.soulteary.com/2019/01/22/fever-api.png)

最后，订阅界面如下。

![stringer rss 默认界面](https://attachment.soulteary.com/2019/01/22/miniflux-ui.png)


## 最后

之前写文章总是考虑没有阅读基础的同学，而忽略了一直订阅、关注着我的同学，未来重复的内容，我将会和本文一样，给予简短的指引，不赘述基础建设，只聊主题相关的核心部分。

最近工作比较忙，没有太多时间写文章，这篇内容躺在草稿箱里快半个月了。接下来我将写几篇内容，聊聊如何解决 RSS 源不能够直接访问，或网站不支持 RSS 订阅的问题。

感谢持续订阅和支持我的朋友。

— EOF