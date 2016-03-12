# 使用 Docker 搭建私有软件仓库 Nexus 3

一年前，我曾经写过一篇[《迁移 Nexus 软件仓库拾遗》](https://soulteary.com/2018/10/08/how-to-migrate-nexus.html)，在文章中有提到一些常见的问题，最近在升级改造相关基础技术设施，觉得应该把经验记录下来，造福有相关需求的同学、团队。

这款 sonatype 公司出品的 Nexus Repository Manager，打 3.x 版本从15年开坑开始到现在，每半个月更新一次，非常值得信赖。目前官方数据显示，全球有超过十万的个人/团队在使用这个企业级的软件。

本文将基于 Docker 和 Traefik v2 聊聊如何搭建一个稳定高效的软件仓库，毕竟这两年里，这个仓库几乎不需要额外的打理，为我个人和团队默默提供着可靠的高性能私有服务。

## 写在前面

说起技术相关的“仓库”，我们一般会想到的是代码仓库，比如之前文章中写到的 [GitLab](https://soulteary.com/tags/gitlab.html)、[Gitea](https://soulteary.com/2020/02/04/gitea-git-server-with-docker-and-traefik-v2.html)、[Gogs](https://soulteary.com/2020/02/04/gogs-git-server-with-docker-and-traefik-v1.html)。

然而这些代码仓库一般只用于存储尚未编译处理的原始程序，而对于编译产物（artifact）的管理一般是不做处理的，即使有这类功能，也相对比较孱弱，比如当前的GitLab。

加之当前研发过程中，非常流行的高频率持续集成生产行为，软件仓库很多时候，除了作为最后的“交付储存池”，还需要肩负着一些额外的责任：

- 提供 “安全可靠的官方软件源镜像”
- 提供 “软件包安全扫描”
- 提供“软件包集中审计平台”
- ...

类似的高级需求，让软件仓库的竞争也激烈了起来，除了 Nexus 外，你或许还听说过 Harbor、Portus。

Nexus 的[官方定位](https://www.sonatype.com/nexus-repository-oss)是一款支持通用格式的软件仓库，对于存储格式**并不敏感**。

也就是说，你可以用它来托管 类似 Ubuntu 的 Linux 软件源、可以用来做 NPM 仓库、也可以用来提供 Maven、Docker、Go、Python、Ruby...你能想到的各种语言、软件所需要的“镜像”。

介绍的够多了，我们来正式进入搭建环节。

## 基础搭建

为了让应用域名和SSL证书能够更加容易的挂载到服务器上，并且便于后续管理。这里依旧使用 Traefik 2.x 版本作为应用网关，相关的内容，你可以从之前的一些文章中做简单了解，比如：[这篇](https://soulteary.com/2020/01/28/traefik-2-user-guide-pleasant-development-experience.html)和[这篇](https://soulteary.com/2020/02/01/configure-traefik-v2-based-web-server.html)。

这里我们启动一个域名为 `nexus.lab.io`，并且支持 HTTP 自动跳转 HTTPS 的全能仓库，进程遇到错误，会自动尝试重新启动。

满足上面需求的容器编排配置非常简单，只需要不到五十行代码。

```yaml
version: "3.6"

services:

  nexus3:
    container_name: nexus.lab.io
    image: sonatype/nexus3:3.21.1
    environment:
      - INSTALL4J_ADD_VM_PARAMS=-Xms2g -Xmx2g -XX:MaxDirectMemorySize=2g -Djava.util.prefs.userRoot=/nexus-data/javaprefs -Duser.timezone=Asia/Shanghai
    restart: always
    expose:
      - 8081
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /etc/timezone:/etc/timezone:ro
      - /var/run/docker.sock:/var/run/docker.sock
      - ./nexus-data:/nexus-data
    networks:
      - traefik
    labels:
      - "traefik.enable=true"
      - "traefik.docker.network=traefik"
      - "traefik.http.middlewares.nexus-bechind-proxy.headers.customrequestheaders.X-Forwarded-Proto=https"
      - "traefik.http.routers.nexus-web.middlewares=https-redirect@file"
      - "traefik.http.routers.nexus-web.entrypoints=http"
      - "traefik.http.routers.nexus-web.rule=Host(`nexus.lab.io`)"
      - "traefik.http.routers.nexus-web.service=nexus-backend"
      - "traefik.http.routers.nexus-ssl.middlewares=content-compress@file,nexus-bechind-proxy"
      - "traefik.http.routers.nexus-ssl.entrypoints=https"
      - "traefik.http.routers.nexus-ssl.tls=true"
      - "traefik.http.routers.nexus-ssl.rule=Host(`nexus.lab.io`)"
      - "traefik.http.routers.nexus-ssl.service=nexus-backend"
      - "traefik.http.services.nexus-backend.loadbalancer.server.scheme=http"
      - "traefik.http.services.nexus-backend.loadbalancer.server.port=8081"
    healthcheck:
      test: ["CMD-SHELL", "curl -f localhost:8081 || exit 1"]
    logging:
        driver: "json-file"
        options:
          max-size: "10m"

networks:
  traefik:
    external: true
```

将上面的内容保存为 `docker-compose.yml` ，使用熟悉的 `docker-compose up -d` 启动应用。

此刻可以使用 `docker-compose logs -f` 来观察应用初始化过程是否出现错误情况，并等待疯狂刷屏的日志停止。

```bash
nexus.lab.io | -------------------------------------------------
nexus.lab.io | 
nexus.lab.io | Started Sonatype Nexus OSS 3.21.1-01
nexus.lab.io | 
nexus.lab.io | -------------------------------------------------
```

不论是你看到类似上面的日志提示，还是使用 `docker ps` 看到下面标示为 `healthy` 的容器进程状态，都说明 Nexus 已经正常启动了起来。

```bash
a9b4ac5142e0        sonatype/nexus3:3.21.1                       "sh -c ${SONATYPE_DI…"   2 hours ago         Up 2 hours (healthy)      8081/tcp, 9100-9101/tcp                    nexus.lab.io
```

在浏览器中打开我们之前配置好的域名：`nexus.lab.io`，可以看到 Nexus 清爽的新版界面。

![Nexus 的新版界面](https://attachment.soulteary.com/2020/03/08/nexus-interface.png)

在去年的时候，Nexus的默认登陆账号和密码还是 `admin` 和 `admin123`。但是显然现在官方意识到这是个错误的策略。

今年新的版本中，初始化的新实例，虽然默认账号还是 `admin`，但是密码已经改为了随机生成的字符串，并且保存在只有拥有应用安装权限的人才能看到的地方：`/nexus-data/admin.password`。

![Nexus 的新版本登陆策略](https://attachment.soulteary.com/2020/03/08/nexus-admin-login.png)

因为我们使用容器启动 Nexus，并将 Nexus 的数据文件挂载到了本地磁盘，所以此时，我们可以选择两个方式来读取这个文件。

```bash
# 在启动应用的目录中执行
cat nexus-data/admin.password
# 或者直接使用 Docker CLI 执行容器命令
docker exec -it nexus.lab.io cat /nexus-data/admin.password
```

在输入了正确的初始账号和密码后，新版软件会人性化的引导我们设置新密码，以及设置是否允许匿名用户使用。

如果是个人使用，或者团队在内网使用，可以勾选“允许匿名访问”。

![Nexus 的新版本初始化设置](https://attachment.soulteary.com/2020/03/08/nexus-setup.jpg)

在聊一些高级使用方法之前，我们需要先了解它的一些基础使用。

## 基础使用

在正确登陆并进行过第一次初始化设置后，我们可以看到顶部的状态栏多了一个齿轮按钮。

![Nexus 登陆后的默认使用界面](https://attachment.soulteary.com/2020/03/08/nexus-has-login.png)

点击按钮，进入管理后台，默认会停留在 “Repository” 菜单，在左侧的侧边栏，选择二级菜单“Repositories”，可以看到默认已经添加了不少常用的软件仓库支持。

![Nexus 默认支持仓库](https://attachment.soulteary.com/2020/03/08/nexus-default-dashboard.png)

这里可能暂时没有你想用的代码仓库，但是没有关系，先从了解它提供软件仓库服务的基础流程开始吧，毕竟所有仓库都大同小异。

我们点开 `maven-group` 这个项目，可以清晰的看到这个 maven 软件仓库是如何工作的：

- 先从 `maven-release` 获取软件包，找不到的话，继续查找下一个类别的项目，这个仓库是我们默认发布软件使用的。
- 接着从 `maven-snapshots` 获取软件包，找不到的话，继续查找下一个类别的项目，这个仓库是我们发布调试版本软件包使用的。
- 最后从 `maven-central` 获取官方源软件包，找不到的话，则宣告 “404 Not Found”。（默认源：`https://repo1.maven.org/maven2/`）

你当然可以选择添加更多来源的 仓库类型，比如“阿里/腾讯镜像”、“公司生产环境”、“公司测试环境”等等，以及调整Nexus的获取响应顺序来改变你在安装软件包时的体验和预期结果。

至于如何配置 Maven 仓库，应该不需要我教你了吧。至此 Nexus 的基础搭建就完成了。

## 最后

考虑到内容篇幅，本篇内容就先到此为止。

接下来的内容，我将介绍如何使用 Nexus 搭建 Docker 仓库、NPM 仓库，以及一些设置细节。

--EOF












