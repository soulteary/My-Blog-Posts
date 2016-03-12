# 使用 Docker 搭建你的Wiki（MediaWiki）

前面介绍了三款不同的 RSS 系统的快速搭建使用，接下来我将演示几种不同的 Wiki 系统，同样是借助 Docker 和 Traefik 进行快速搭建，本篇是第一篇，MediaWiki。

如果你有阅读过我之前写的文章，那么参考本篇文章将文章搭建起来，应该只需要3分钟或者更少，如果你没有看过，那么可以点击本文相关的文章标签，阅读过往的文章。

## 关于 MediaWiki

MediaWiki 是一款开源并且支持免费使用的软件，由PHP编写，被广泛使用在各种知识社区内，我们熟悉的维基百科就是运行在这套程序上的。

在开源生态里，MediaWiki 的周边生态十分庞大，各种工具和机器人资源相当丰富。

时刻四个月，2019年1月，它更新了 1.32 版，本文基于此版本撰写。

![MediaWiki 默认安装界面](https://attachment.soulteary.com/2019/01/28/first-screen.png)

私以为 Wiki 和 常规的笔记类软件最大不同在于内容是经过精心校对的，并且能够直观呈现树型结构形式之外的知识内容，文章内自动关联，搭配标签系统可以很容易的形成知识网络。

话不多说，开始实战。

使用 Compose 可以一键启动一个使用 SQLite 作为数据储存的 MediaWiki ，配置很简单，不到30行代码。

```yaml
version: "3"

services:
  # 如果你使用数据库，可以参考下面的地址，或者我文章中标记有 Docker 的历史文章
  # https://docs.docker.com/samples/library/mediawiki/
  mediawiki:
    restart: always
    image: mediawiki:1.32
    labels:
      - "traefik.enable=true"
      - "traefik.frontend.rule=Host:mediawiki.lab.io"
      - "traefik.frontend.passHostHeader=true"
      - "traefik.frontend.entryPoints=http,https"
    networks:
      - traefik
    expose:
      - 80
    volumes:
      # 默认上传位置
      - ./uploads/images:/var/www/html/images
      # 默认 SQLite 储存位置
      - ./data:/var/www/data
      # 当初始化安装完毕之后，将配置文件下载并保存到下面的位置，
      # 并去掉注释，重启应用
      # - ./LocalSettings.php:/var/www/html/LocalSettings.php

networks:
  traefik:
    external: true
```

第一次使用该配置启动程序，会引导你进行安装，主要是进行应用常规配置，以及初始化数据库。

![MediaWiki 安装结束](https://attachment.soulteary.com/2019/01/28/setup.png)

当你进行到最后一步的时候，程序会自动保存你所有操作，并生成一个配置文件。将该文件保存并移动到 `docker-compose.yml` 同级目录下，并使用 Compose 重启应用，安装就完成了。

![MediaWiki 正式使用](https://attachment.soulteary.com/2019/01/28/finish.png)

## 链接展示优化

安装完毕之后，如果觉得默认的链接不够优雅，希望能够去掉URL链接中的 `/index.php/` 内容，可以修改 `LocalSettings.php` 文件内容。

替换 `$wgScriptPath = "";` 为下面的配置内容即可。

```plain
$wgScriptPath = "";
$wgArticlePath = "/wiki/$1";
$wgUsePathInfo = true;
```

## 最后

如果你对本文聊到的 Docker 、Traefik 、Compose 还不是很熟悉，欢迎阅读我的以往文章，补全对上述技术的认识，希望我的文章可以对你有帮助。

接下来我会继续介绍几种不同的 Wiki 系统的安装配置、魔改，如果你也在考虑如何维护一套让自己用起来舒服的知识管理工具，可以继续关注，下回再见。

— EOF