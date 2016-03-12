# 修理 Ghost 中文输入法的 BUG

去年的时候，我曾写过一篇文章 [《 将 Ghost 迁移 Hugo 背后的事 》](https://soulteary.com/2019/06/15/migrating-ghost-behind-hugo.html) 里面描述了Ghost 当前对于非英文用户的主要问题。

其中最令人诟病的便是编辑器对于 CJK 三种语言输入法“吃字” BUG 的问题，这个问题影响 Ghost 从 2.x 到现在的 3.x 版本。产生问题的原因是，核心编辑器组件 *0.11.1* - *0.12.x* 版本中对于输入法事件没有进行逻辑覆盖，维护者和 Ghost 官方时至今日依旧没有做出任何变更，不管用户提了多少“IME BUG”的求助和 PR 。

最近有项目需要一个文档编辑器，于是考虑“修理修理”它，看看能不能用起来。

<!-- more -->

## 关于这个持续了两年的BUG

- 2018 年的时候，有人提交了 PR [https://github.com/bustle/mobiledoc-kit/pull/661](https://github.com/bustle/mobiledoc-kit/pull/661) ，使用 `event.isIME()` 方法了问题修正，但是受到了非标准实现的质疑，后续这个问题便被挂了两年之久。
- 2019 年末，又有人提交了新的 PR [https://github.com/TryGhost/mobiledoc-kit/pull/11](https://github.com/TryGhost/mobiledoc-kit/pull/11)，使用 `event.isComposing()` 方法进行问题修正，但是不知道是单纯因为 CI 测试未通过，还是带有客户群歧视，这个 PR 也被搁置了。

## 前置准备

本文依赖一些软件或前置知识，如果不是很了解，可以参考以往的文章。

- 需要 Docker、docker-compose
- 可选 traefik

### Traefik 的使用

Traefik 的具体使用，可以参考以往的文章，比如：[使用服务发现改善开发体验](https://soulteary.com/2018/06/11/use-server-side-discovery-improve-development.html)、[更完善的 Docker + Traefik 使用方案](https://soulteary.com/2018/08/28/better-use-of-docker-and-traefik.html) 等，更多内容，可以翻看历史内容的标签，这里不过多赘述。

本文只需要关注编排文件中的 `labels` 和 `networks` 字段配置就足够啦。对不同容器服务的 `networks` 字段，声明包含相同的内容，则可以让不同应用所处于的网络一致。

```yaml
networks:
  - traefik
```

比如上面的声明，会让容器服务都处于名为 `traefik` 的网络环境中。

## 早期的修正方案

去年年初的时候，忍不了这个 BUG 的时候，我在官方主仓库一个 IME BUG 的 ISSUE 里，我提了一个解决方案，告诉大家把当时的 Ghost 项目的 `package.json` 中的 `@tryghost/mobiledoc-kit` 替换为 `@bugfix/mobiledoc-kit` ，然后重新构建资源就能解决问题，时至今日这条评论还能收到一些点赞。

[https://github.com/TryGhost/Ghost/issues/9801#issuecomment-457904129](https://github.com/TryGhost/Ghost/issues/9801#issuecomment-457904129)

![早先的解决方案](https://attachment.soulteary.com/2020/01/19/previous-issue.png)

但是到去年下半年的时候，这个方案便由于官方设计变更而失效了，而且这样做也不利于后续跟进官方 bugfix 的版本，太笨重不够灵活。

## 当前的修正方案

要解决的问题主要是在客户端运行的脚本，治标又治本的方案是对于有问题的脚本进行 patch ，然后重新构建项目，让页面加载新的脚本资源即可。

但是官方编译项目设计的非常不环保不绿色，项目拆分设计不是十分合理，构建脚本 tricks和硬编码巨多，使用 `grunt` 搭配 `git submoudle` 非常不利于 debug。而且要全局安装一堆构建工具，还需要锁定 Node 运行版本在老版本，编译效率更是慢到令人发指（Mac Book Pro 2019 i9 2.4GHz 编译感觉时间巨慢长）...

为了避免后续浪费更多时间在折腾项目架构的问题上，这里考虑将定制软件的容器编译环境，交由性能更高的服务器进行编译，并抽取编译产物对每个版本的 Ghost 进行资源替换，来解决这个陈年 BUG。

我把这个方案上传到了 GitHub：[https://github.com/soulteary/youling](https://github.com/soulteary/youling)，方便你进行进一步定制改造，如果有必要的话，可以持续跟进几个版本的官方更新。

下面来聊聊方案的详细内容。

## 定制构建镜像生成“补丁”

官方编辑器补丁文件，我上传到了 [GitHub](https://github.com/soulteary/youling/blob/master/patches/mobiledoc-kit/event-manager.js)，可以自取。构建环境踩坑过程不表，下面是构建容器的 Dockerfile：

```bash
FROM node:12-alpine
LABEL maintainer="soulteary@gmail.com"
 
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US.UTF-8
ENV LC_ALL=en_US.UTF-8

RUN echo '' > /etc/apk/repositories && \
    echo "https://mirror.tuna.tsinghua.edu.cn/alpine/v3.10/main"         >> /etc/apk/repositories && \
    echo "https://mirror.tuna.tsinghua.edu.cn/alpine/v3.10/community"    >> /etc/apk/repositories && \
    echo "Asia/Shanghai" > /etc/timezone

RUN apk update && apk add git && \
    yarn global add knex-migrator grunt-cli ember-cli bower

COPY patches/mobiledoc-kit/event-manager.js /patches/mobiledoc-kit/event-manager.js

RUN git clone https://github.com/TryGhost/mobiledoc-kit.git /mobiledoc-kit && \
    cd /mobiledoc-kit && \
    git checkout 3b0f375d32f7183a4eee9cce5373ebabeb249165 && \
    cp /patches/mobiledoc-kit/event-manager.js /mobiledoc-kit/src/js/editor/event-manager.js && \
    yarn && \
    cp -r /mobiledoc-kit/dist /patches/mobiledoc-kit/dist && \
    rm -rf /mobiledoc-kit

RUN git clone --recurse-submodules https://github.com/TryGhost/Ghost.git /Ghost && \
    cd /Ghost && \
    git checkout 3.3.0 && \
    yarn setup

RUN rm -rf /Ghost/core/client/node_modules/\@tryghost/mobiledoc-kit/dist && \
    cp -r /patches/mobiledoc-kit/dist /Ghost/core/client/node_modules/\@tryghost/mobiledoc-kit/

WORKDIR /Ghost

RUN grunt prod

EXPOSE 2368

CMD ["npm", "start"]
```

执行 `docker build -t soulteary/ghost:3.3.0` ，如果是本地构建，泡杯茶休息会，大概十分钟左右镜像就好了。当然，你也可以执行 `docker pull soulteary/ghost:3.3.0` 直接获取已经构建好的镜像。

接着，创建一个 `docker-compose.assets.yml` 用于提取构建镜像中的静态资源。

```yaml
version: '3'
services:

  build-ghost-assets:
    image: soulteary/ghost:3.3.0
    container_name: ghost-assets
    volumes:
      - ./patches/ghost-assets/loop.js:/Ghost/index.js
```

由于拷贝资源的镜像必须在拷贝的时候存活，而 Ghost 启动必须配置数据库，不然就报错退出，所以这里创建一个 HTTP Server 来解决问题。

```js
const http = require("http");

const server = http.createServer(function(req, res) {
  res.writeHead(200);
  res.end("Hold on for sync assets");
});

server.listen(2368);
```

将下面的命令保存为 `sync-assets.sh`，执行之后，项目目录中就会出现 3.3.0 版本 Ghost 的编辑器静态资源补丁了。

```js
#!/usr/bin/env bash

docker-compose -f docker-compose.assets.yml down && docker-compose -f docker-compose.assets.yml up -d

rm -rf docker-assets && mkdir -p docker-assets

docker cp ghost-assets:/Ghost/core/built ./docker-assets/built
docker cp ghost-assets:/Ghost/core/server/web/admin/views ./docker-assets/admin-views

docker-compose -f docker-compose.assets.yml down
```

## 验证方案效果

想要验证补丁效果，可以本地启动一套 Ghost，首先创建一个 `docker-compose.db.yml` 启动本地数据库。

```yaml
version: '2'
services:

  db:
    image: mysql:5.7
    container_name: ghost-db
    networks:
      - traefik
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: ghost
    volumes:
      - ./localdb:/var/lib/mysql

networks:
  traefik:
    external: true
```

等待日志中出现 `ready for connections.`表示数据库就绪了：

```TeXT
ghost-db | 2020-01-18T17:24:04.154079Z 0 [Note] Event Scheduler: Loaded 0 events
ghost-db | 2020-01-18T17:24:04.154348Z 0 [Note] mysqld: ready for connections.
ghost-db | Version: '5.7.28'  socket: '/var/run/mysqld/mysqld.sock'  port: 3306  MySQL Community Server (GPL)
```

接着创建 `docker-compose.yml` 并输入下面的内容：

```yaml
version: "3.6"

services:

  www-ghost-local:
    image: ghost:3.3.0
    expose:
      - 2368
    environment:
      url: https://soulteary.io
      database__client: mysql
      database__connection__host: ghost-db
      database__connection__user: root
      database__connection__password: ghost
      database__connection__database: ghost
      NODE_ENV: production
    volumes:
      - ./docker-assets/built:/var/lib/ghost/versions/3.3.0/core/built:ro
      - ./docker-assets/admin-views:/var/lib/ghost/current/core/server/web/admin/views:ro
      - ./config.production.json:/var/lib/ghost/config.production.json:ro
      - ./content/adapters:/var/lib/ghost/versions/3.3.0/content/adapters
      - ./content/apps:/var/lib/ghost/versions/3.3.0/content/apps
      - ./content/images:/var/lib/ghost/versions/3.3.0/content/images
      - ./content/logs:/var/lib/ghost/content/logs
      - ./content/settings:/var/lib/ghost/versions/3.3.0/content/settings
    extra_hosts:
      - "soulteary.io:127.0.0.1"
    networks:
      - traefik
    labels:
      - "traefik.enable=true"
      - "traefik.port=2368"
      - "traefik.frontend.rule=Host:soulteary.io"
      - "traefik.frontend.entryPoints=https,http"

networks:
  traefik:
    external: true
```

使用 `docker-compose up` 启动应用，等待应用启动就绪：

```TeXT
www-ghost-local_1  | [2020-01-18 17:25:10] INFO Ghost is running in production...
www-ghost-local_1  | [2020-01-18 17:25:10] INFO Your site is now available on https://soulteary.io/
www-ghost-local_1  | [2020-01-18 17:25:10] INFO Ctrl+C to shut down
www-ghost-local_1  | [2020-01-18 17:25:10] INFO Ghost boot 9.472s
```

![打开Ghost 的管理后台](https://attachment.soulteary.com/2020/01/19/dashboard.png)

访问 `http://soulteary.io/ghost` 进行项目的初始化，设置管理员账号后，随手创建一篇文章就能进行测试啦。

![一切都正常了](https://attachment.soulteary.com/2020/01/19/final-test.gif)

## 最后

做开源软件不易，但是如果目的不只是简单做做营销PR，拉一些流量。而是想造福、方便更多人，让社区生态更好，那么就应该摆出开放包容的姿态，适当谦虚接受社区批评和建议，何况大家都帮你写好了代码，只要点一下 Merge 按钮就好了。

--EOF
