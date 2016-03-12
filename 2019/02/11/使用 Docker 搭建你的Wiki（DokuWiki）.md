# 使用 Docker 搭建你的Wiki（DokuWiki）

前面介绍了三款不同的 RSS 系统的快速搭建使用，接下来我将演示几种不同的 Wiki 系统，同样是借助 Docker 和 Traefik 进行快速搭建，本篇是第四篇，DokuWiki。

如果你有阅读过我之前写的文章，那么参考本篇文章将文章搭建起来，应该只需要3分钟或者更少，如果你没有看过，那么可以点击本文相关的文章标签，阅读过往的文章。

## 关于 DokuWiki

DokuWiki 同样是一款开源并且支持免费使用的软件，由 PHP 编写，第一个提交版本在2004年，作为开源产品已经被稳定迭代了15个年头 [^1]，同样天生跨平台，并被广泛使用在各种知识社区内，尤其适合中小团队和个人作为知识整理软件使用。

记得第一份工作，在新浪云团队的时候，内部的 Wiki 便是基于 Doku 搭建的。

DokuWiki 和前面介绍的 MoinMoin 很类似，默认使用纯文本作为数据记录的方式，所以占用服务器资源很低。

官方目前还在迭代，不过因为维护时间很长，迭代频率相对比较慢，距离当下最新的版本是 **2018年4月22日** ，本文基于此版本进行撰写，感兴趣的同学可以围观：[官方项目仓库](https://github.com/splitbrain/dokuwiki)。

话不多说，开始实战。

## 使用 Compose 运行 DokuWiki

DokuWiki 同样没有提供官方容器镜像，但是在 DockerHub 搜索的时候发现，Bitnami 有封装好的镜像 `bitnami/dokuwiki` ，我个人比较信任这个团队，从2013年开始使用他们的服务到现在，一直没有什么大问题。

这里图个省事，就不进行镜像封装了，想学习封装的同学可以翻阅之前的文章内容，不放心镜像的同学，可以围观镜像源代码地址，进行安全审查：[https://github.com/bitnami/bitnami-docker-dokuwiki](https://github.com/bitnami/bitnami-docker-dokuwiki)

配合下面的配置文件，使用 Compose 可以一键启动一个使用 文本文件 作为数据储存的 DokuWiki ，配置很简单，30 行代码左右。

```yaml
version: '3'

services:

  dokuwiki:
    container_name: doku.lab.io
    restart: always
    image: 'bitnami/dokuwiki:0.20180422.201901061035-r12'
    labels:
      - "traefik.enable=true"
      - "traefik.frontend.rule=Host:doku.lab.io"
      - "traefik.frontend.passHostHeader=true"
      - "traefik.frontend.entryPoints=https,http"
      - "traefik.frontend.headers.SSLRedirect=true"
      - "traefik.frontend.headers.STSSeconds=315360000"
      - "traefik.frontend.headers.frameDeny=true"
      - "traefik.frontend.headers.SSLProxyHeaders=X-Forwarded-Proto:https"
      - "traefik.frontend.redirect.regex=^https?://doku.lab.io/(.*)"
      - "traefik.frontend.redirect.replacement=https://doku.lab.io/$${1}"
      - "traefik.frontend.headers.customResponseHeaders=Access-Control-Allow-Origin:*"
    environment:
      - DOKUWIKI_FULL_NAME=soulteary
      - DOKUWIKI_EMAIL=soulteary@gmail.com
      - DOKUWIKI_WIKI_NAME=Wiki
      - DOKUWIKI_USERNAME=soulteary
      - DOKUWIKI_PASSWORD=soulteary
    networks:
      - traefik
    expose:
      - 80
    volumes:
      - ./data:/bitnami

networks:
  traefik:
    external: true
```

和之前不同的是，这里演示了如何使用 `Docker Label` 定义 `Traefik` 的一些额外能力，比如自动挂载 `ssl/tls` 证书，域名重定向。

当使用 `docker-compose up` 将应用启动之后，你会看到下面的日志，耐心等待 `dokuwiki successfully initialized` 出现在日志中，之后便可以通过我们配置的域名进行访问了，本例中地址为 `doku.lab.io` 。（我使用 Traefik 提供服务发现，如果你不会操作，请访问我的历史文章，了解 Traefik 如何使用。）

```plain
Creating doku.lab.io ... done
Attaching to doku.lab.io
doku.lab.io |
doku.lab.io | Welcome to the Bitnami dokuwiki container
doku.lab.io | Subscribe to project updates by watching https://github.com/bitnami/bitnami-docker-dokuwiki
doku.lab.io | Submit issues and feature requests at https://github.com/bitnami/bitnami-docker-dokuwiki/issues
doku.lab.io |
doku.lab.io | nami    INFO  Initializing apache
doku.lab.io | apache  INFO  ==> Patching httpoxy...
doku.lab.io | apache  INFO  ==> Configuring dummy certificates...
doku.lab.io | nami    INFO  apache successfully initialized
doku.lab.io | nami    INFO  Initializing php
doku.lab.io | nami    INFO  php successfully initialized
doku.lab.io | nami    INFO  Initializing libphp
doku.lab.io | nami    INFO  libphp successfully initialized
doku.lab.io | nami    INFO  Initializing dokuwiki
doku.lab.io | dokuwik INFO  Passing wizard, please be patient
doku.lab.io | dokuwik INFO
doku.lab.io | dokuwik INFO  ########################################################################
doku.lab.io | dokuwik INFO   Installation parameters for dokuwiki:
doku.lab.io | dokuwik INFO     username: soulteary
doku.lab.io | dokuwik INFO     user fullname: soulteary
doku.lab.io | dokuwik INFO     Password: **********
doku.lab.io | dokuwik INFO     Email: soulteary@gmail.com
doku.lab.io | dokuwik INFO     Wiki Name: Wiki
doku.lab.io | dokuwik INFO   (Passwords are not shown for security reasons)
doku.lab.io | dokuwik INFO  ########################################################################
doku.lab.io | dokuwik INFO
doku.lab.io | nami    INFO  dokuwiki successfully initialized
doku.lab.io | INFO  ==> Starting dokuwiki...
doku.lab.io | [Mon Feb 11 09:11:14.374658 2019] [ssl:warn] [pid 101] AH01909: localhost:443:0 server certificate does NOT include an ID which matches the server name
doku.lab.io | [Mon Feb 11 09:11:14.381884 2019] [ssl:warn] [pid 101] AH01909: localhost:443:0 server certificate does NOT include an ID which matches the server name
doku.lab.io | [Mon Feb 11 09:11:14.447186 2019] [ssl:warn] [pid 101] AH01909: localhost:443:0 server certificate does NOT include an ID which matches the server name
doku.lab.io | [Mon Feb 11 09:11:14.455003 2019] [ssl:warn] [pid 101] AH01909: localhost:443:0 server certificate does NOT include an ID which matches the server name
doku.lab.io | [Mon Feb 11 09:11:14.494463 2019] [mpm_prefork:notice] [pid 101] AH00163: Apache/2.4.37 (Unix) OpenSSL/1.1.0j PHP/7.1.26 configured -- resuming normal operations
doku.lab.io | [Mon Feb 11 09:11:14.494539 2019] [core:notice] [pid 101] AH00094: Command line: 'httpd -f /bitnami/apache/conf/httpd.conf -D FOREGROUND'
```

在展示程序界面和常规操作之前，我们还是先说一下数据存放地址，以及未来插件要在哪里进行存放和应用。

在上面的配置文件 `docker-compose.yml` 的同级目录会自动生成 `data` 目录，在目录内会包含用户数据、环境配置相关的内容，如下所示：

```plain
data
├── apache
│   └── conf
├── dokuwiki
│   ├── conf
│   ├── data
│   └── lib
│       ├── images
│       ├── plugins
│       └── tpl
└── php
    └── conf
```

如果你需要应用官方市场的插件或者主题，请放置于 `data/dokuwiki/lib/plugin/` 目录内的指定文件夹中，和 `MoinMoin` 不同的是，不需要重启容器进行，直接刷新浏览器页面，插件就能够自动加载了。

我们的 Wiki 条目数据会被存放在 `data/dokuwiki/data` 中，所以请定期对该位置数据进行备份保存。

## DokuWiki 的常规操作

打开浏览器，可以看到 DokuWiki 已经运行起来了。

![DokuWiki 安装就绪](https://attachment.soulteary.com/2019/02/11/install-finish.png)

在当前页面右侧可以看到编辑菜单，点击后可以进入编辑界面。

![DokuWiki 创建Wiki条目](https://attachment.soulteary.com/2019/02/11/create-wiki.png)

默认不使用插件，语法需要使用特殊语法 [官方语法参考](https://www.dokuwiki.org/wiki:syntax)。

![DokuWiki 编辑器页面](https://attachment.soulteary.com/2019/02/11/editor.png)

点击保存，第一条 Wiki 条目的更新操作就完成了。

![DokuWiki 的第一条 Wiki 条目](https://attachment.soulteary.com/2019/02/11/done.png)

再次点击条目中的信息链接，可以直观的查看到内容的变更记录，并执行不同版本的对比，获取更多的信息。

![DokuWiki 执行条目对比](https://attachment.soulteary.com/2019/02/11/diff.png)

至于安装插件、配置主题等，可以点击顶部菜单栏 `Admin`，比较直观就不赘述了。

![DokuWiki 管理界面](https://attachment.soulteary.com/2019/02/11/admin-dashboard.png)

## 其他

如果你想要对链接进行优化，可以参考这里 [Pull Request](https://github.com/bitnami/bitnami-docker-dokuwiki/pull/49/files) 修改 `data` 目录中的 Apache 配置文件即可。

## 最后

如果你对本文聊到的 Docker 、Traefik 、Compose 还不是很熟悉，欢迎阅读我的以往文章，补全对上述技术的认识，希望我的文章可以对你有帮助。

接下来我会继续介绍几种不同的 Wiki 系统的安装配置、魔改，如果你也在考虑如何维护一套让自己用起来舒服的知识管理工具，可以继续关注，下回再见。

— EOF

[^1]:	[https://www.dokuwiki.org/old\_changes#release\_2004-07-04](https://www.dokuwiki.org/old_changes#release_2004-07-04)