# 使用 Docker 和 Traefik v2 搭建 Confluence 7.3

之前写过三篇如何使用“[容器化方案来搭建 Confluence](https://soulteary.com/tags/confluence.html)”，本文将基于最近最新推出的 Confluence 7.3  来演示如何使用新版的软件。

如果你想要给公司团队或者个人搭建 Wiki，可以参考之前关于[如何搭建  Wiki 的实战文章](https://soulteary.com/tags/wiki.html)，里面记录了如何高效完成搭建，并避过踩坑的方法。

以往已经上车使用的用户，也可以参考本文进行升级。

## 写在前面

在独立运行维护多个 Confluence 实例一年多之后，经历了多次的升级维护，确认了使用容器比宿主机直接安装是更为有效的维护方式。

因为首先，你的操作能够都被版本化的记录下来，如同管理代码一般；其次，需要去维护和管理的内容，只是极少一部分变量，而非整个环境。

官方对于容器是积极但是还不够严谨，从过往一年来看：

- 在 2019 年里，confluence 官方因为一个严重漏洞，重新发布并覆盖了以往所有的容器镜像，导致文件权限和卷挂载出现过问题。
- 在 2019 年里，官方镜像缺少必要参数，导致用户不得不修改文件，并挂载到容器内部。
- 在 2020 年初，官方升级插件市场的证书，而未更新容器根证书，导致容器启动服务插件下载失败。
- 在 2020 年初，官方放弃了 alpine 镜像发布。
- ...

虽然问题多多，但是总体而言，还是很值得期待的毕竟高频率的周更/月更，加活跃的社区和市场，支持“mb4字符集”，构建出了一个可以碾压任何同类的商业化 Wiki 产品。

对于个人而言，只需要每年付费 10$ ，就能满足一个10人初期团队的使用，并且在 2020 年，使用 2G 内存的服务器也能愉快的运行 Confluence 了。

当然，我更推荐 4G及以上的配置。

## 基础准备

- Docker Hub 上官方容器镜像：`https://hub.docker.com/r/atlassian/confluence-server/tags`
	- 本文将基于 7 系列新版本： `7.3`
- MySQL JDBC Connector : `https://dev.mysql.com/downloads/connector/j/5.1.html` 
	- 如果你也选择使用 MySQL 作为储存后端，需要下载此文件，一般情况下你会获得 **mysql-connector-java-5.1.47.tar.gz** 的压缩包，解压缩之后，获得 **mysql-connector-java-5.1.47.jar**，我们稍后会用到。
- 一些中文字体，比如 `simsun.ttc`、`simkai.ttf`等，如果你需要使用“导出文档为 PDF、Word”功能，并且文档包含中文，为了渲染正常，你需要提供一些中文字体。

## 基础容器化

参考去年写的文章[《使用 Docker 搭建 Confluence》](https://soulteary.com/2019/03/30/construct-confluence-with-docker.html)、以及今年[“Traefik 2.x 版本升级”](https://soulteary.com/tags/traefik.html)的文章指引，不难写出下面的基础配置。

```yaml
version: '3'

services:

  confluence:
    image: atlassian/confluence-server:7.3.2-ubuntu
    container_name: confluence-app
    expose:
      - 8090
      - 8091
    networks:
      - traefik
    labels:
      - "traefik.enable=true"
      - "traefik.docker.network=traefik"
      - "traefik.http.routers.wiki-web.middlewares=https-redirect@file"
      - "traefik.http.routers.wiki-web.entrypoints=http"
      - "traefik.http.routers.wiki-web.rule=Host(`wiki.lab.com`)"
      - "traefik.http.routers.wiki-ssl.middlewares=content-compress@file"
      - "traefik.http.routers.wiki-ssl.entrypoints=https"
      - "traefik.http.routers.wiki-ssl.tls=true"
      - "traefik.http.routers.wiki-ssl.rule=Host(`wiki.lab.com`)"
      - "traefik.http.services.wiki-backend.loadbalancer.server.scheme=http"
      - "traefik.http.services.wiki-backend.loadbalancer.server.port=8090"
    environment:
      - 'CATALINA_OPTS=-Duser.timezone=GMT+08 -Dconfluence.document.conversion.fontpath=/usr/local/share/fonts/'
      - 'JVM_MINIMUM_MEMORY=1024m'
      - 'JVM_MAXIMUM_MEMORY=2048m'
    volumes:
      - ./data:/var/atlassian/application-data/confluence
      - ./deps/confluence/mysql-connector-java-5.1.47.jar:/opt/atlassian/confluence/confluence/WEB-INF/lib/mysql-connector-java-5.1.47.jar
      - ./fonts:/usr/local/share/fonts
      - /etc/localtime:/etc/localtime:ro
    extra_hosts:
      - 'wiki.lab.com:127.0.0.1'

networks:
  traefik:
    external: true
```

上面的配置几乎完美，将上面的内容保存为 **docker-compose.yml** 后，使用 `docker-compose up -d` 启动应用，你就能够得到一个新版本的 Confluence 了。

## 需要解决的问题

但是这样运行起来的 Confluence 会遇到一些问题：

- 后台登陆提示你需要修正代理配置
- 插件市场因为官方变更服务域名，镜像证书比较陈旧导致无法使用

接下来，我们就来解决这些问题。

### 后台提示需要修正域名配置

这个问题常常出现在使用了反向代理、负载均衡给 Confluence 挂载证书的情况下，在以往的版本中，我们需要添加 `server.xml` 并进行文件只读锁定，来解决这个问题。

官方文档稍显陈旧，但是也记录过这个问题：[《Can't check base URL warning in Confluence 6.6 or later》](https://confluence.atlassian.com/confkb/can-t-check-base-url-warning-in-confluence-6-6-or-later-939718433.html)。

但是在新版本中，我们可以通过设置容器运行环境变量来解决这个问题，不过这里有一个 Tricks 的事情，如果你不创建并挂载 `server.xml` 这个文件，你将无法解决这个问题。

首先创建 `server.xml` 文件：

```xml
<?xml version="1.0" encoding="utf-8"?>

<Server port="8000"
        shutdown="SHUTDOWN">

  <Listener className="org.apache.catalina.startup.VersionLoggerListener"/>
  <Listener className="org.apache.catalina.core.AprLifecycleListener"
            SSLEngine="on"/>
  <Listener className="org.apache.catalina.core.JreMemoryLeakPreventionListener"/>
  <Listener className="org.apache.catalina.mbeans.GlobalResourcesLifecycleListener"/>
  <Listener className="org.apache.catalina.core.ThreadLocalLeakPreventionListener"/>

  <Service name="Catalina">

    <Connector port="8090"
               maxThreads="100"
               minSpareThreads="10"
               connectionTimeout="20000"
               enableLookups="false"
               protocol="HTTP/1.1"
               redirectPort="8443"
               acceptCount="10"
               secure="false"
               scheme="https"
               proxyName="wiki.lab.com"
               proxyPort="443"

               relaxedPathChars="[]|"
               relaxedQueryChars="[]|{}^\`&quot;&lt;&gt;"
               bindOnInit="false"
               maxHttpHeaderSize="8192"
               useBodyEncodingForURI="true"
               disableUploadTimeout="true" />

    <Engine name="Standalone"
            defaultHost="localhost"
            debug="0">
      <Host name="localhost"
            debug="0"
            appBase="webapps"
            unpackWARs="true"
            autoDeploy="false"
            startStopThreads="4">
        <Context path=""
                 docBase="../confluence"
                 debug="0"
                 reloadable="false"
                 useHttpOnly="true">
          <!-- Logging configuration for Confluence is specified in confluence/WEB-INF/classes/log4j.properties -->
          <Manager pathname=""/>
          <Valve className="org.apache.catalina.valves.StuckThreadDetectionValve"
                 threshold="60"/>
        </Context>

        <Context path="${confluence.context.path}/synchrony-proxy"
                 docBase="../synchrony-proxy"
                 debug="0"
                 reloadable="false"
                 useHttpOnly="true">
          <Valve className="org.apache.catalina.valves.StuckThreadDetectionValve"
                 threshold="60"/>
        </Context>

      </Host>
    </Engine>

  </Service>

</Server>
```

接着在容器编排文件中添加挂载命令：

```yaml
volumes:
  - ./deps/confluence/server.xml:/opt/atlassian/confluence/conf/server.xml
```

最后，在编排文件中添加环境变量：

```yaml
environment:
  - 'CATALINA_CONNECTOR_PROXYNAME=wiki.lab.com'
  - 'CATALINA_CONNECTOR_PROXYPORT=443'
  - 'CATALINA_CONNECTOR_SCHEME=https'
```

最后，重启你的应用就好了。

### 插件市场提示不能访问

这个问题其实挺麻烦的，我实际运行的时候，主进程没有报任何错误，但是根据以往封装镜像的经验，判断是 JRE 证书信任问题，找到了官方相关的一些资料

- [《The Atlassian Marketplace Server is not Reachable Due to Peer Not Authenticated》](https://confluence.atlassian.com/confkb/the-atlassian-marketplace-server-is-not-reachable-due-to-peer-not-authenticated-321850263.html?utm_medium=hercules-issue-view&utm_source=SAC&utm_content=420060)
- [《SSL Handshake Error When Connecting to Atlassian Marketplace》](https://confluence.atlassian.com/confkb/ssl-handshake-error-when-connecting-to-atlassian-marketplace-978215061.html)

这个问题其实应该说是一个老问题了，从[2012 年](https://community.atlassian.com/t5/Marketplace-Apps-Integrations/Atlassian-Marketplace-server-is-not-reachable/qaq-p/81125)、[2014 年](https://community.atlassian.com/t5/Jira-questions/The-Atlassian-Marketplace-server-is-not-reachable-in-the/qaq-p/333714)以及之后频出。而上面这些标记为 `7.3` 版本使用的资料其实只是一个线索，不能直接使用。

想要知道原因吗？且往下看。

#### 如何添加并信任新的证书

想信任新的证书，先得先获取新的证书文件，使用 `openssl` 工具将证书保存为文件。

```bash
openssl s_client -connect marketplace.atlassian.com:443 < /dev/null | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > marketplace.atlassian.com.crt
```

以往的文章，针对的是老版本的 JDK（低于10），使用 `keytool` 导入证书就行了。

```bash
keytool -import -trustcacerts -alias proxy_root -file marketplace.atlassian.com.crt -keystore $JAVA_HOME/jre/lib/security/cacerts -storepass changeit -noprompt
```

上面的操作，可以解决 confluence 5.x / 6.x 的问题，而在 confluence 7.x 中，[JDK 升级为了 11](https://hub.docker.com/layers/atlassian/confluence-server/7.3.1-ubuntu/images/sha256-b29572c6485da93cd9abf9fa6be79346578b12caa37f4d49fd36250b74c7005f?context=explore)，储存位置有了变化。Oracle 官方博客有记录[《OpenJDK 10 Now Includes Root CA Certificates》](https://blogs.oracle.com/jtc/openjdk-10-now-includes-root-ca-certificates)这个变更。所以命令需要变更为：

```bash
keytool -import -trustcacerts -alias proxy_root -file marketplace.atlassian.com.crt -keystore $JAVA_HOME/lib/security/cacerts -storepass changeit -noprompt
```

#### 让“补丁操作”持久化

如果是宿主机安装的话，执行上面的操作，然后重启进程就好了。

但是我们使用的是运行过程“无状态”的容器，对于官方容器作出任何改变，并重启，这些“变更”会自然而然的随着容器沙盒销毁而销毁。（如果没有使用特殊的 daemon 进程方案的话）

所以这里需要基于官方镜像，定制一个补丁镜像，内容很简单。

```bash
FROM atlassian/confluence-server:7.3.2-ubuntu
LABEL maintainer="soulteary@gmail.com"
 
USER root

RUN openssl s_client -connect marketplace.atlassian.com:443 < /dev/null | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > /tmp/marketplace.atlassian.com.crt
RUN keytool -import -trustcacerts -alias proxy_root -file /tmp/marketplace.atlassian.com.crt -keystore $JAVA_HOME/lib/security/cacerts -storepass changeit -noprompt

USER confluence
```

使用 `docker build -t confluence-server:7.3.2-ubuntu-fix .` 将新的容器镜像命名为 ** confluence-server:7.3.2-ubuntu-fix**。

然后在编排文件中，替换镜像名称，再次启动容器，插件市场就能正常访问了。

```yaml
image: confluence-server:7.3.2-ubuntu-fix
```

## 修正后的编排配置

为了方便使用，这样提供一个完整的配置文件。

```yaml
version: '3'

services:

  confluence:
    image: confluence-server:7.3.2-ubuntu-fix
    container_name: confluence-app
    expose:
      - 8090
      - 8091
    networks:
      - traefik
    labels:
      - "traefik.enable=true"
      - "traefik.docker.network=traefik"
      - "traefik.http.routers.wiki-web.middlewares=https-redirect@file"
      - "traefik.http.routers.wiki-web.entrypoints=http"
      - "traefik.http.routers.wiki-web.rule=Host(`wiki.lab.com`)"
      - "traefik.http.routers.wiki-ssl.middlewares=content-compress@file"
      - "traefik.http.routers.wiki-ssl.entrypoints=https"
      - "traefik.http.routers.wiki-ssl.tls=true"
      - "traefik.http.routers.wiki-ssl.rule=Host(`wiki.lab.com`)"
      - "traefik.http.services.wiki-backend.loadbalancer.server.scheme=http"
      - "traefik.http.services.wiki-backend.loadbalancer.server.port=8090"
    environment:
      - 'CATALINA_OPTS=-Duser.timezone=GMT+08 -Dconfluence.document.conversion.fontpath=/usr/local/share/fonts/'
      - 'JVM_MINIMUM_MEMORY=1024m'
      - 'JVM_MAXIMUM_MEMORY=2048m'
      - 'CATALINA_CONNECTOR_PROXYNAME=wiki.lab.com'
      - 'CATALINA_CONNECTOR_PROXYPORT=443'
      - 'CATALINA_CONNECTOR_SCHEME=https'
    volumes:
      - ./data:/var/atlassian/application-data/confluence
      - ./deps/confluence/mysql-connector-java-5.1.47.jar:/opt/atlassian/confluence/confluence/WEB-INF/lib/mysql-connector-java-5.1.47.jar
      - ./fonts:/usr/local/share/fonts
      - ./deps/confluence/server.xml:/opt/atlassian/confluence/conf/server.xml
      - /etc/localtime:/etc/localtime:ro
    extra_hosts:
      - 'wiki.lab.com:127.0.0.1'

networks:
  traefik:
    external: true
```

## 其他

我个人建议作为生产环境使用，务必使用云服务商的云数据，安全性和可靠性更好，但是如果个人使用，本地启动一个数据库容器实例，也不是不可，参考官方建议文档，可以将数据库启动参数调整为：

```yaml
command: --character-set-server=utf8mb4 --collation-server=utf8mb4_bin --default-storage-engine=INNODB --max_allowed_packet=256M --innodb_log_file_size=2GB --transaction-isolation=READ-COMMITTED --binlog_format=row
```

另外，如果你遇到了中文文件不能正常渲染的问题，可以参考下面这篇文章进行缓存清理：

- [《容器化 Confluence 使用拾遗》](https://soulteary.com/2019/04/19/talk-about-confluence-with-docker.html)

## 最后

每次升级调整 Confluence 的过程，都是一种花式踩坑的过程，过程虽然很麻烦，但是踩过之后，依旧会忍不住的感叹，个人使用每年 10$ 花的真值。 

--EOF