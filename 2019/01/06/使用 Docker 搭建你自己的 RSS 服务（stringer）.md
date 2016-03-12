# 使用 Docker 搭建你自己的 RSS 服务（stringer）

在算法推荐满天飞的世界里，定制获取信息就显得比较另类了，但是它可能是更高效的手段。

本篇是我之前提到的三种常规的 RSS 服务搭建方式的第二篇，Stringer。

如果你有阅读过我之前写的文章，那么参考本篇文章将文章搭建起来，应该只需要3分钟或者更少，如果你没有看过，那么可以点击本文相关的文章标签，阅读过往的文章。

## 关于 Stringer

Stringer 是一款基于 Ruby 编写的 RSS 服务。相比同为 Ruby 编写的 feedbin ，它的架构更为简单，界面也更现代化。

我之前使用它的主要原因有：

- 支持有限时间的离线缓存。
- 支持 Fever API，允许用户在客户端阅读器上进行阅读。
- 代码量不大，技术栈简单。

但是它也有一些问题：

- 文档不够丰富，优化调试时，需要翻代码。
- 维护者虽然还在持续更新，但是活跃度不高。
- 如果想离线图片，那么需要修改代码实现，或者自己包装一层 Feed 源。

如果你想了解更多，可以访问[这里](https://github.com/swanson/stringer)，如果你只是想使用，那么请继续阅读。

## 封装一个新的 Docker 容器镜像

为什么要封装一个新的容器呢？

社区里有好心人对当前版本提供了新的代码提交，但是没有被合并到主干，评论里也有提到一些安全风险需要修正，官方容器镜像的代码内容比较旧。

而且对比自己封装镜像和官方镜像的差异，发现文件尺寸也差了接近一倍。并且官方使用的数据库版本比较旧（PQ v9.5），在使用的过程中，还需要手动进入容器进行辅助操作，太不环保了。

```bash
REPOSITORY                             TAG                   IMAGE ID            CREATED             SIZE
docker.lab.com/stringer                stable                dc8210ef3209        5 hours ago         603MB
stringer                               3712cf21              f1d712553750        5 hours ago         603MB
mdswanson/stringer                     latest                6858330db397        2 months ago        1.01GB

```

你可以使用下面的配置，构建属于你的新镜像。

```yaml
FROM ruby:2.3.3-alpine

ENV RACK_ENV=production
ENV PORT=8080

ENV LANG=en_US.UTF-8 \
LANGUAGE=en_US.UTF-8 \
LC_CTYPE=en_US.UTF-8 \
LC_ALL=en_US.UTF-8

EXPOSE 8080

# skip installing gem documentation
RUN mkdir -p /usr/local/etc \
  && { \
    echo 'install: --no-document'; \
    echo 'update: --no-document'; \
  } >> /usr/local/etc/gemrc


RUN apk add --no-cache --virtual \
    build-dependencies build-base linux-headers tzdata supervisor postgresql-dev sqlite-dev nodejs curl bash dcron && \
	rm -rf /var/cache/apk/* /tmp/* && \
	ln -fs /usr/share/zoneinfo/Asia/Shanghai /etc/localtime


WORKDIR /app
ADD Gemfile Gemfile.lock /app/
RUN bundle install --deployment

ENV SUPERCRONIC_URL=https://github.com/aptible/supercronic/releases/download/v0.1.3/supercronic-linux-amd64 \
    SUPERCRONIC=supercronic-linux-amd64 \
    SUPERCRONIC_SHA1SUM=96960ba3207756bb01e6892c978264e5362e117e

RUN curl -fsSLO "$SUPERCRONIC_URL" \
 && echo "${SUPERCRONIC_SHA1SUM}  ${SUPERCRONIC}" | sha1sum -c - \
 && chmod +x "$SUPERCRONIC" \
 && mv "$SUPERCRONIC" "/usr/local/bin/${SUPERCRONIC}" \
 && ln -s "/usr/local/bin/${SUPERCRONIC}" /usr/local/bin/supercronic

ADD docker/supervisord.conf /etc/supervisord.conf
ADD docker/start.sh /app/
ADD . /app

RUN addgroup -S stringer && \
    adduser -S -G stringer stringer && \
    chown -R stringer:stringer /app

USER stringer

CMD /app/start.sh
```

我这里将镜像构建后命名为 `docker.lab.com/stringer:stable` ，后续我们进行服务编排的时候，会使用。

## 使用 Docker 和 Traefik 提供服务

下面是我提供的服务应用配置，定义了中文界面，每十五分钟的更新频率。

```yaml
version: '2'

services:

  db:
    image: postgres:10.4-alpine
    expose:
      - 5432
    restart: always
    networks:
      - traefik
    volumes:
      - ./data/postgresql:/var/lib/postgresql
      # This needs explicit mapping due to https://github.com/docker-library/postgres/blob/4e48e3228a30763913ece952c611e5e9b95c8759/Dockerfile.template#L52
      - ./data/postgresql_data:/var/lib/postgresql/data
    environment:
      - POSTGRES_PASSWORD=pass
      - POSTGRES_USER=user
      - POSTGRES_DB=stringer

  web:
    image: docker.lab.com/stringer:stable
    depends_on:
      - db
    restart: always
    networks:
      - traefik
    expose:
      - 8080
    environment:
      - SECRET_TOKEN=YOUR_SECRET_TOKEN
      - PORT=8080
      - DATABASE_URL=postgresql://user:pass@db:5432/stringer
      - LOCALE=zh-CN
      - LOG_LEVEL=INFO
      - FETCH_FEEDS_CRON="*/15 * * * *"
      #- CLEANUP_CRON="0 0 * * *"
    labels:
      - "traefik.enable=true"
      - "traefik.port=8080"
      - "traefik.frontend.rule=Host:rss.lab.com"
      - "traefik.frontend.entryPoints=http,https"

networks:
  traefik:
    external: true
```

在使用 `docker-compose up` 将服务运行起来之后，我们打开浏览器，访问：`rss.lab.com` ，设置你的个人账号密码之后，你就能够拥有一个功能强大，界面友好的 RSS 订阅服务了。

![stringer rss 默认界面](https://attachment.soulteary.com/2019/01/06/stringer-1.png)

默认没有数据源，所以你需要添加一个数据源，如果你之前是 RSS 用户，可以直接使用 OPML 数据源导入的方式批量导入你的订阅，每一个列表元素前的红绿小点表示了网站数据是否通畅，如果你订阅的网站在国内因为网络原因不能访问，可以自己对它进行网络中转，再添加订阅。

![stringer rss 默认界面](https://attachment.soulteary.com/2019/01/06/stringer-2.png)

Stringer 和其他阅读器不同的是，它的理念只是倡导阅读，整理贮藏的操作需要用户额外使用其他的软件进行，阅读过的文章默认保存时间是一个月。

![stringer rss 默认界面](https://attachment.soulteary.com/2019/01/06/stringer-3.png)


## 最后

之前写文章总是考虑没有阅读基础的同学，而忽略了一直订阅、关注着我的同学，未来重复的内容，我将会和本文一样，给予简短的指引，不赘述基础建设，只聊主题相关的核心部分。

虽然这个服务搭建完毕了，但是并不能很好的服务于我们，因为在当前的网络大环境下，越来越多的网站“被迫封闭了起来”，不再支持 RSS 方式的订阅模式，至于如何解决，请耐心等待这三篇文章结束后，我提供的方案吧。

— EOF