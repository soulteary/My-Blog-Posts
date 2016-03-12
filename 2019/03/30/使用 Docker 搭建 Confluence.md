# 使用 Docker 搭建 Confluence

小型团队协同，或者想花钱买个省心，Confluence 是比较好的选择之一。但是最近安装 Confluence ，发现官方和网上的安装介绍都比较“落后”低效，所以有了本篇内容。

本文将介绍如何使用 **Docker Compose** 快速搭建 **Confluence** 、以及如何和 **Traefik** 一同使用，如果你看过之前的内容，跟随本文应该能在十分钟内解决战斗。

## 基础准备

- Docker Hub 上官方容器镜像：`https://hub.docker.com/r/atlassian/confluence-server/tags`
	- 这里会讲解两个有代表性的版本： `6.4` 和 `6.15`
- MySQL JDBC Connector : `https://dev.mysql.com/downloads/connector/j/5.1.html` 
	- 如果你也选择使用 MySQL 作为储存后端，需要下载此文件，一般情况下你会获得 **mysql-connector-java-5.1.47.tar.gz** 的压缩包，解压缩之后，获得 **mysql-connector-java-5.1.47.jar**，我们稍后会用到。

## 针对老版本软件的使用

先说老版本，如果你只是需要基础的 Wiki 功能，那么下面的配置文件应该能够满足你的需求。

```yaml
version: '3'

services:

  confluence:
    image: atlassian/confluence-server:6.4.3-alpine
    expose:
      - 8090
      - 8091
    networks:
      - traefik
    labels:
      - "traefik.enable=true"
      - "traefik.port=8090"
      - "traefik.frontend.rule=Host:${DOMAIN}"
      - "traefik.frontend.entryPoints=http,https"
    volumes:
      - ./data:/var/atlassian/application-data/confluence
      - ./mysql-connector-java-5.1.47.jar:/opt/atlassian/confluence/confluence/WEB-INF/lib/mysql-connector-java-5.1.47.jar

networks:
  traefik:
    external: true
```

将上面的文件保存为 **docker-compose.yml** 后，我们创建另外基础配置文件 **.env **，和上面的配置一样简单，文件内容可以是下面这样。

```TeXT
DOMAIN=wiki.lab.com
```

将 **docker-compose.yml** 、**.env**、**mysql-connector-java-5.1.47.jar** 放在同一目录，如果此刻你的 Traefik 已经就绪，那么执行 `docker-compose up` ，你的服务便启动起来了。

直接访问你配置好的域名，比如例子中的 `wiki.lab.com`，你就可以进行 Confluence 的 Web 界面配置啦。如果你还不会使用 Traefik ，那么可以翻阅历史文章，同样是一些十分钟以内的教程。

如果你选择将 Confluence 部署在公网，面对每天很是烦人的扫描器，不妨简单添加 `Basic Auth` 认证，将这些恶意请求拦截在外面。

因为使用了 Traefik ，所以添加这个功能十分简单，只需要两步：

第一步，在 **docker-compose.yml** 的 `labels` 字段内添加下面的内容。

```TeXT
- "traefik.frontend.auth.basic=${BASIC_AUTH}"
```

第二步，执行 `htpasswd -nb user user`，得到一段包含用户名和加密后的密码的文本字符串，譬如这样：`user:$apr1$MzgRxukq$MhYl/2JidzUNlHfyfIQF41`，接着将内容添加到 **.env** 中：

```TeXT
BASIC_AUTH=user:$apr1$MzgRxukq$MhYl/2JidzUNlHfyfIQF41
```

当再有扫描器想直接对应用进行扫描的时候，就会被 Basic Auth 挡在外面啦。

### 应用健康检查报错

当你安装完毕，开始使用的时候，会发现界面的右上角会提示一个警告信息。

> Can't check base URL

官方知识库中有[提到](https://confluence.atlassian.com/confkb/can-t-check-base-url-warning-in-confluence-6-1-or-later-884707131.html)这个问题，如果你使用的也是低版本（6.6）之前，其实可以通过配置 `Hosts` 来解决问题。

比如在 **docker-compose.yml** 中添加一段声明，让应用服务器查找本机上应用地址，而非一定要访问公网地址的应用，参考配置如下：

```yaml
version: '3'

services:

  confluence:
    image: atlassian/confluence-server:6.4.3-alpine
    expose:
      - 8090
      - 8091
    networks:
      - traefik
    labels:
      - "traefik.enable=true"
      - "traefik.port=8090"
      - "traefik.frontend.rule=Host:${DOMAIN}"
      - "traefik.frontend.entryPoints=http,https"
    volumes:
      - ./data:/var/atlassian/application-data/confluence
      - ./mysql-connector-java-5.1.47.jar:/opt/atlassian/confluence/confluence/WEB-INF/lib/mysql-connector-java-5.1.47.jar
    extra_hosts:
      - "${DOMAIN}:127.0.0.1"

networks:
  traefik:
    external: true
```

是不是十分简单，如果你的需求是基础使用，上述的配置应该已经能够满足你的需求了。

## 针对新版本软件的使用

接着我们聊聊如何使用最新版本的软件，因为我们使用了容器，所以更新版本十分简单，在配置文件中修改镜像的版本号就好了。比如，我想将 `6.4.3` 这个低版升级到其他版本，只需要将配置中的 `6.4.3` 改为 `6.15.1` 即可，例如 `atlassian/confluence-server:6.15.1-alpine`。

其他的基本和老版本软件使用一致。不过这里会有几个小问题，需要额外解决一下。

### 数据库不能正确连接

> WARN: Establishing SSL connection without server's identity verification is not recommended. According to MySQL 5.5.45+, 5.6.26+ and 5.7.6+ requirements SSL connection must be established by default if explicit option isn't set. For compliance with existing applications not using SSL the verifyServerCertificate property is set to 'false'. You need either to explicitly disable SSL by setting useSSL=false, or set useSSL=true and provide truststore for server certificate verification.

要解决这个问题，可以选择配置加密的 MySQL 连接，并更新容器中的证书，也可以选择添加参数，关闭强制使用加密连接请求，后者更简单，如果要求不高，可以这么做。

编辑 **data/confluence.cfg.xml** 文件中的 `hibernate.connection.url` ，在连接地址后添加 `?useSSL=false` 参数，重启应用即可。

### Traefik Basic Auth 和 Tomcat 发生联动

新版本的软件逻辑中，有针对请求中带有 `Basic Auth` 进行额外处理：如果在上面配置了 Basic Auth ，那么应用会提示验证失败，不能登录系统。

这个显然不是我们添加 Basic Auth 的用意，并且实际使用中，也不推荐直接将 Confluence 的认证接口对外。

解决方案很简单，在 `docker-compose.yml` 中添加一行 `- "traefik.frontend.auth.basic.removeHeader=true"` ，Traefik 的验证信息将仅针对 Traefik 使用，在反向代理应用的时候，HTTP 请求中的验证信息会被删除掉。

同样的，重启应用，这个问题就解决了。

### 稍微麻烦一些的健康检查

因为我们使用 Traefik 挂载证书，应用实际运行在代理服务器背后，当使用管理员访问控制台，会看到一个警告信息。

> 您的 URL 不匹配
> 
> Confluence 的基本URL设置为http://wiki.lab.com，但您正从https://wiki.lab.com访问 Confluence。

考虑应用的正常使用，我们通常会将协议进行修正，比如将站点基础URL修正为 `https` 。但是在修正之后，你会收到另外一个警告。

> Tomcat 配置不正确
> 
> Tomcat server.xml 配置不正确：
> scheme 应为 'https'
> proxyName 应为 ‘YOUR\_DOMAIN\_URI’
> proxyPort 应为 '443'

原因是比较新的版本的应用，健康检查逻辑附带了端口和协议判断，低版本可以直接使用 Traefik 反代挂载证书的幸福快乐日子一去不复返。

解决问题需要分为三步。

第一步，将容器内的 Tomcat 运行配置 `server.xml` 拷贝到本地（da5582a01879 为 docker ps 获取的容器PID）。

```bash
docker cp da5582a01879:/opt/atlassian/confluence/conf/server.xml .
```

第二步，将配置中端口为 8090 的 Connector 的配置更新为下面的内容（尤其注意最后一行内容）：

```xml
<Connector
    port="8090"
    connectionTimeout="20000"
    redirectPort="8443"
    maxThreads="48" minSpareThreads="10"
    enableLookups="false"
    acceptCount="10"
    debug="0"
    URIEncoding="UTF-8"
    protocol="org.apache.coyote.http11.Http11NioProtocol"
    proxyName="wiki.lab.com" proxyPort="443" scheme="https"/>
```

第三步，更新 `docker-compose.yml` 配置文件。

在 `volumes` 字段中添加内容：

```yaml
- ./server.xml:/opt/atlassian/confluence/conf/server.xml
```

同时删除 `extra_hosts` 字段内容。

重启应用，一切正常。

### 完整的配置文件

为了方便使用，这里给出完整的参考配置。

```yaml
version: '3'

services:

  confluence:
    image: atlassian/confluence-server:6.15.1-alpine
    expose:
      - 8090
      - 8091
    networks:
      - traefik
    labels:
      - "traefik.enable=true"
      - "traefik.port=8090"
      - "traefik.frontend.rule=Host:${DOMAIN}"
      - "traefik.frontend.entryPoints=http,https"
      - "traefik.frontend.auth.basic.removeHeader=true"
      - "traefik.frontend.auth.basic=${BASIC_AUTH}"
    volumes:
      - ./data:/var/atlassian/application-data/confluence
      - ./mysql-connector-java-5.1.47.jar:/opt/atlassian/confluence/confluence/WEB-INF/lib/mysql-connector-java-5.1.47.jar
      - ./server.xml:/opt/atlassian/confluence/conf/server.xml


networks:
  traefik:
    external: true
```

## 最后

虽然对于团队来说 Confluence 是一个不错的方案，但是实际针对个人/拥有定制能力的团队而言，使用完全开源免费的 WordPress 或许会更好，下一篇我将介绍 WordPress 用作知识管理用途的一些定制处理。