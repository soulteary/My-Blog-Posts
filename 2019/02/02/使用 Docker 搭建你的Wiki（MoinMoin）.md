# 使用 Docker 搭建你的Wiki（MoinMoin）

前面介绍了三款不同的 RSS 系统的快速搭建使用，接下来我将演示几种不同的 Wiki 系统，同样是借助 Docker 和 Traefik 进行快速搭建，本篇是第二篇，MoinMoin。

如果你有阅读过我之前写的文章，那么参考本篇文章将文章搭建起来，应该只需要3分钟或者更少，如果你没有看过，那么可以点击本文相关的文章标签，阅读过往的文章。

## 关于 MoinMoin

MediaWiki 是一款开源并且支持免费使用的软件，由Python编写，同样天生跨平台，并被广泛使用在各种知识社区内。

当然你可能会觉得陌生，但是当说提及几个站点之后，你可能会大呼原来如此：

- [Python 官方Wiki](https://wiki.python.org/)
- [Ubuntu 官方社区](https://help.ubuntu.com/community/)
- [Debian 官方Wiki](https://wiki.debian.org/)
- [WireShark 官方Wiki](https://wiki.wireshark.org/)

除此之外，还有 GNOME、WineHQ、ID3、GCC、GRUB 等一堆大名鼎鼎的软件都使用了它。

![使用 MoinMoin 部署的网站之一](https://attachment.soulteary.com/2019/02/02/preview.png)

套用“互联网圈”的话，如果说 MediaWiki 做的是 C 端市场，那么 MoinMoin 主打的则是 B 端的企业服务。

但是在开源生态里，MoinMoin 的周边生态就不比  MediaWiki 了，不过好在全面够用，想了解的同学可以[戳此访问](http://moinmo.in/MoinMoinExtensions)。

他目前的稳定版本是 **v1.9.10** ，**v2.0**版本正在开发的路上，有需求的同学可以去 GitHub 上[了解更多](https://github.com/moinwiki/moin)，本文基于稳定版本撰写。

话不多说，开始实战。

## 使用 Compose 运行 MoinMoin

配合下面的配置文件，使用 Compose 可以一键启动一个使用 文本文件 作为数据储存的 MoinMoin ，配置很简单，20行代码左右。

```yaml
version: "3"

services:

  # https://hub.docker.com/r/olavgg/moinmoin-wiki/
  moinmoin:
    restart: always
    image: olavgg/moinmoin-wiki:1.9.10.1
    environment:
      - NOSSL=1
    labels:
      - "traefik.enable=true"
      - "traefik.frontend.rule=Host:moinmoin.lab.io"
      - "traefik.frontend.passHostHeader=true"
      - "traefik.frontend.entryPoints=http,https"
    networks:
      - traefik
    expose:
      - 80
    volumes:
      - ./data:/usr/local/share/moin/data

networks:
  traefik:
    external: true

```

当使用 `docker-compose up` 将应用启动之后，便可以通过我们配置的域名进行访问了，本例中地址为 `moinmoin.lab.io` 。（我使用 Traefik 提供服务发现，如果你不会操作，请访问我的历史文章，了解 Traefik 如何使用。）

在展示程序界面和常规操作之前，我们说一下我们的数据存放地址，以及未来插件要在哪里进行存放和应用。

在上面的配置文件 `docker-compose.yml` 的同级目录会自动生成 `data` 目录，在目录内会包含用户数据相关的内容，如下所示：

```plain
data
├── cache
│   ├── README
│   ├── __session__
│   ├── spellchecker.dict
│   └── wikiconfig
├── dict
│   └── dummy_dict
├── edit-log
├── event-log
├── initialized
├── intermap.txt
├── meta
├── pages
│   ├── BadContent
│   └── FrontPage
├── plugin
│   ├── action
│   ├── converter
│   ├── events
│   ├── filter
│   ├── formatter
│   ├── macro
│   ├── parser
│   ├── theme
│   ├── userprefs
│   └── xmlrpc
└── user
```

如果你需要应用官方市场的插件或者主题，请放置于 `data/plugin/` 目录内的指定文件夹中，并重启 MoinMoin。

我们的 Wiki 条目数据会被存放在 `data/pages` 中，所以请定期对该位置数据进行备份保存。

## MoinMoin 的常规操作

打开浏览器，可以看到 MoinMoin 已经运行起来了。

![MoinMoin 已经就绪](https://attachment.soulteary.com/2019/02/02/standby.png)

双击任意一个“帖子”，可以直接进入编辑器界面。

![MoinMoin 编辑器界面](https://attachment.soulteary.com/2019/02/02/editor.png)

点击保存，第一条 Wiki 条目的更新操作就完成了。

![MoinMoin 的第一条 Wiki 条目](https://attachment.soulteary.com/2019/02/02/first-wiki.png)

点击条目中的信息链接，可以直观的查看到内容的变更记录，并执行不同版本的对比，获取更多的信息。

![MoinMoin 执行条目对比](https://attachment.soulteary.com/2019/02/02/diff.png)

## 最后

如果你对本文聊到的 Docker 、Traefik 、Compose 还不是很熟悉，欢迎阅读我的以往文章，补全对上述技术的认识，希望我的文章可以对你有帮助。

接下来我会继续介绍几种不同的 Wiki 系统的安装配置、魔改，如果你也在考虑如何维护一套让自己用起来舒服的知识管理工具，可以继续关注，下回再见。

— EOF