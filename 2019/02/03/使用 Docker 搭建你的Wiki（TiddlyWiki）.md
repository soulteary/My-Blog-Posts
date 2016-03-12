# 使用 Docker 搭建你的Wiki（TiddlyWiki）

前面介绍了三款不同的 RSS 系统的快速搭建使用，接下来我将演示几种不同的 Wiki 系统，同样是借助 Docker 和 Traefik 进行快速搭建，本篇是第三篇，TiddlyWiki，除了讲述搭建之外，简单演示如何优化你的容器镜像。

如果你有阅读过我之前写的文章，那么参考本篇文章将文章搭建起来，应该只需要3分钟或者更少，如果你没有看过，那么可以点击本文相关的文章标签，阅读过往的文章。

## 关于 TiddlyWiki

TiddlyWiki 同样是一款开源并且支持免费使用的软件，由JavaScript编写，同样天生跨平台，被广泛用于个人知识整理。

作者来自牛津大学，开发这个Wiki软件许多年，凭借软件的一些独特的优势，因此有了不少铁粉，那么这款软件有什么不同于其他Wiki软件的特点呢？

- 最新版（5.x）软件支持两种运行模式：
	- HTML 单页面（SPA 应用）
	- Node.js (Web 应用)
- 单页面模式几乎不需要任何编程能力，只需要双击页面文件即可立刻开始使用，存储云盘或U盘中可以做到随身携带。
- Wiki 条目高度可定制，对于常见的公式、图表、代码高亮等功能支持良好。
- 提供各种常用功能插件、语言包、不同风格的主题，可以切换传统Wiki。
- 目前提供客户端（基于NW.js）/ 各种奇怪的运行方式（比如跑在手机里）。

下面是软件的官方站点，以及对应的中文汉化版本。汉化版本软件版本比较低，不过常见功能使用没有太大变化。

- [官方演示站点](https://tiddlywiki.com/)
- [简体中文站点](http://tiddlywiki.cn/)
- [繁体中文站点](http://web.nlhs.tyc.edu.tw/~lss/wiki/TiddlyWikiTutorialTW.html)

本文将使用 Node.js 模式进行 Wiki 站点的建设，一来性能更好，二来可以让整个应用变为同构类型，二次开发效率也更高，三来，单文件版本应该不需要一篇实践文档。

软件目前版本是 **v5.1.19** ，自 2013年版本号从 1.x 跳跃到 5.0 后，作者已经开发了6个年头，[版本记录见此](https://github.com/Jermolene/TiddlyWiki5/releases?after=v5.0.0-alpha.17)，本文基于该稳定版本撰写。

话不多说，开始实战。

## 编写基础 Docker 镜像

下面是我们的 **Dockerfile** ：

```plain
FROM node:11.9.0-alpine

RUN npm install -g tiddlywiki@5.1.19

EXPOSE 8080

VOLUME [ "/app" ]

WORKDIR /app

CMD [ "tiddlywiki", ".", "--listen", "host=0.0.0.0" ]
```

使用 `Docker build` 命令构建镜像，这里我们暂定镜像名称为 `docker.lab.com/tiddlywiki:5.1.19` 。

```bash
docker build -t docker.lab.com/tiddlywiki:5.1.19 .
```

如果你在构建过程中觉得很慢，可以使用国内淘宝团队维护的镜像，将第一条 `RUN` 指令后的命令替换为：

```plain
RUN npm install -g tiddlywiki@5.1.19 --registry=https://registry.npm.taobao.org
```

## 使用 Compose 运行 TiddlyWiki

配合下面的配置文件，使用 Compose 可以一键启动一个使用 文本文件 作为数据储存的 TiddlyWiki ，配置很简单，20行代码左右。

```yaml
version: '3'

services:

  tiddly:
    image: docker.lab.com/tiddlywiki:5.1.19
    restart: always
    labels:
      - "traefik.enable=true"
      - "traefik.port=8080"
      - "traefik.frontend.rule=Host:tiddlywiki.lab.io"
      - "traefik.frontend.entryPoints=https,http"
      - "traefik.frontend.headers.customResponseHeaders=Access-Control-Allow-Origin:*"
    networks:
      - traefik
    volumes:
      - ./app:/app
    expose:
      - 8080
    command: tiddlywiki . --init server

networks:
  traefik:
    external: true
```

不过如果直接使用这样的配置和容器镜像，使用起来会有一些小麻烦，我们需要执行“两次” `docker-compose up`：

- 第一次，直接使用上面的配置文件，运行后，初始化默认的Wiki数据目录。
- 第二次，删除 `command` 指令后的内容，让容器镜像以默认命令执行，启动服务。

作为一个有追求的折腾控，我们当然要避免这样的情况，太不环保了。

那么我们来解决这个问题吧。

## 编写进阶版本的容器镜像

下面是新的 **Dockerfile** ：

```plain
FROM node:11.9.0-alpine
MAINTAINER soulteary@gmail.com

RUN npm install -g tiddlywiki@5.1.19

EXPOSE 8080

VOLUME [ "/app" ]

WORKDIR /app

COPY entrypoint.sh /entrypoint.sh

RUN chmod +x /entrypoint.sh

CMD [ "/entrypoint.sh" ]
```

与之前的版本相比，我们将提供一个新的“入口文件”，`entrypoint.sh` 文件内容如下：

```bash
#!/usr/bin/env sh

if [ ! -f "/app/tiddlywiki.info" ]; then
    tiddlywiki /app --init server
fi

tiddlywiki /app --listen host=0.0.0.0
```

重新使用 `docker build` 命令构建镜像之后，我们来修正之前的 `docker-compose.yml` 配置文件。

## 再次使用 Compose 运行 TiddlyWiki

```yaml
version: '3'

services:

  tiddly:
    image: docker.lab.com/tiddlywiki:5.1.19
    restart: always
    labels:
      - "traefik.enable=true"
      - "traefik.port=8080"
      - "traefik.frontend.rule=Host:tiddlywiki.lab.io"
      - "traefik.frontend.entryPoints=https,http"
      - "traefik.frontend.headers.customResponseHeaders=Access-Control-Allow-Origin:*"
    networks:
      - traefik
    volumes:
      - ./app:/app
    expose:
      - 8080

networks:
  traefik:
    external: true
```

将配置保存为 `docker-compose.yml` ，然后执行 `docker-compose up` 启动应用，在配置文件同级目录将会自动创建一个名为 `app` 的目录，其中将保存站点的配置文件，以及我们未来所有的 Wiki 条目数据，一切都是全自动的。

数据目录结构如下：

```plain
app
├── tiddlers
│   ├── $__SiteTitle.tid
│   └── $__StoryList.tid
└── tiddlywiki.info
```

数据备份很简单，只需要定期对该目录进行文件备份即可，如果后面有机会，或许可以写一篇专门用于文件备份的文章。

当使用 `docker-compose up` 将应用启动之后，便可以通过我们配置的域名进行访问了，本例中地址为 `tiddlywiki.lab.io` 。（我使用 Traefik 提供服务发现，如果你不会操作，请访问我的历史文章，了解 Traefik 如何使用。）

## TiddlyWiki 概览

打开浏览器，可以看到 TiddlyWiki 已经运行起来了。

![TiddlyWiki 已经就绪](https://attachment.soulteary.com/2019/02/03/first-screen.png)

点击右侧的“齿轮”可以进入设置页面，除了常规操作之外，还能够配置插件、语言包、主题等。

接下来我们以配置 TiddlyWiki 为中文为例，打开设置面板的插件标签页，点击开插件后，选择语言包分类，找到中文语言包后，点击“安装”按钮。

![获取官方提供的插件](https://attachment.soulteary.com/2019/02/03/plugin.png)

下载完毕之后，页面顶部会多出一条黄色的提示，依次点击提示条中的“保存”和“刷新”按钮。

![下载语言包](https://attachment.soulteary.com/2019/02/03/download-translate.png)

然后回到配置面板的首页，向下滚动页面，找到语言配置项，选择中文，稍等 1、2秒后，语言包便配置生效了。

![修改 TiddlyWiki 中使用的语言](https://attachment.soulteary.com/2019/02/03/apply-translate.png)

其他插件的下载配置也类似，回到首页，我们可以看到界面右侧的工具栏已经变成中文。

而文章内容使用什么语言书写，便会显示什么内容，如果你想做多语言站点，可以摸索一下，TiddlyWiki 同样支持。

![汉化后的工具栏](https://attachment.soulteary.com/2019/02/03/tool.png)

接下来便可以开始正式编写Wiki之旅了，点击工具栏齿轮旁边的“+”号，可以创建新文章，而点击页面中已存在内容的钢笔图标，则可以对已存在的内容进行修改。

![TiddlyWiki 编辑界面](https://attachment.soulteary.com/2019/02/03/editor.png)

更多内容大家可以自行探索。

## 最后

如果你对本文聊到的 Docker 、Traefik 、Compose 还不是很熟悉，欢迎阅读我的以往文章，补全对上述技术的认识，希望我的文章可以对你有帮助。

接下来我会继续介绍几种不同的 Wiki 系统的安装配置、魔改，如果你也在考虑如何维护一套让自己用起来舒服的知识管理工具，可以继续关注，下回再见。

— EOF