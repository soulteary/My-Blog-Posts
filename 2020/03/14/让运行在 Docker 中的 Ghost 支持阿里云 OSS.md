# 让运行在 Docker 中的 Ghost 支持阿里云 OSS

最近在优化 Ghost 作为线上使用的内容管理后台，作为线上使用的系统，不同于内部 MIS ，可靠性和应用性能需要有一定保障。

解决性能问题，最简单的方案便是进行水平扩展，而我们知道，如果想要让一个服务做到水平可扩展，除了要将应用运行状态单独持久化外，也必须做到文件储存的持久化，云平台的对象储存就是一个很好的文件持久化方案。

Ghost 是一个典型的单体应用，v3.x 版本的容器化文档其实不多，而介绍如何使用 Aliyun OSS 的文档更是没有，折腾过程还是挺有趣的，记录下来，希望能够帮助到后面有需求的同学。

## 写在前面

[官方文档](https://ghost.org/docs/concepts/storage-adapters/#available-custom-storage-adapters)在使用三方自定义储存部分其实写的不是很好：

1. 文档有效性不敢恭维，虽然内容中提到支持阿里云，但是列表中的[阿里云OSS插件](https://github.com/MT-Libraries/ghost-oss-store)仅针对于 1.x 版本，其中阿里云的 SDK 也比较旧，当时的 Node 环境也很陈旧。
2. 自定义文档缺少技术细节、以及完整描述，需要通过实践和阅读源码去验证。
3. 完全没提到如何在容器镜像，尤其是官方镜像中使用插件。

本文将通过相对流程化的容器方案，来解决以上问题。

之前的文章[《从定制 Ghost 镜像聊聊优化 Dockerfile》](https://soulteary.com/2020/03/09/optimize-dockerfile-from-custom-ghost-image.html)、[修理 Ghost 中文输入法的 BUG](https://soulteary.com/2020/01/19/bugfix-for-ghost-editor-cjk-input.html) 有提过，“如何对 Ghost 进行容器化封装”，感兴趣的同学可以了解下。

在“反复横跳”踩了一堆坑之后，相对稳妥的低成本维护方案便是为 Ghost 编写适合当前版本的储存插件，并制作基于官方容器镜像的补丁镜像了。

在编写插件之前，需要先确认官方环境中的 Node 版本，以确定符号语法：

```bash
docker run --rm -it --entrypoint /usr/local/bin/node ghost:3.9.0-alpine -v
```

执行完上述命令，你将得到 **v12.16.1** 的结果，看来可以直接使用 `async/await` 来编写插件减少代码量了。

## 编写 OSS 储存插件

参考[官方模版](https://ghost.org/docs/concepts/storage-adapters/#creating-a-custom-storage-adapter)，以及阿里云 OSS SDK 完成储存插件大概十几分钟就搞定了，相关代码我已经上传至 [GitHub](https://github.com/soulteary/ghost-aliyun-oss-store)，如果需要二次封装，可以参考使用。

```js
/**
 * Ghost v3 Storage Adapter (Aliyun OSS)
 * @author soulteary(soulteary@gmail.com)
 */

const AliOSS = require("ali-oss");
const GhostStorage = require("ghost-storage-base");
const { createReadStream } = require("fs");
const { resolve } = require("path");

class AliOSSAdapter extends GhostStorage {
  constructor(config) {
    super();
    this.config = config || {};
    this.oss = new AliOSS({
      region: config.region,
      accessKeyId: config.accessKeyId,
      accessKeySecret: config.accessKeySecret,
      bucket: config.bucket
    });

    this.ossURL = `${config.bucket}.${config.region}.aliyuncs.com`;
    this.regexp = new RegExp(`^https?://${this.ossURL}`, "i");
    this.domain = config.domain || null;
    this.notfound = config.notfound || null;
  }

  async exists(filename, targetDir = this.getTargetDir("/")) {
    try {
      const { status } = await this.oss.head(resolve(targetDir, filename));
      return status === 404;
    } catch (err) {
      return false;
    }
  }
  delete() {
    // it's unnecessary
    // Ghost missing UX
  }
  serve() {
    return function(req, res, next) {
      next();
    };
  }

  async read(options) {
    try {
      const { meta } = await this.oss.head(options.path);
      if (meta && meta.path) {
        return meta.path;
      } else {
        return this.notfound;
      }
    } catch (err) {
      console.error(`Read Image Error ${err}`);
      return this.notfound;
    }
  }

  async save(image, targetDir = this.getTargetDir("/")) {
    try {
      const filename = await this.getUniqueFileName(image, targetDir);
      const { url } = await this.oss.put(filename, createReadStream(image.path));

      if (url && url.indexOf(`://${this.ossURL}`) > -1) {
        return this.domain ? url.replace(this.regexp, this.domain) : url;
      } else {
        return this.notfound;
      }
    } catch (err) {
      console.error(`Upload Image Error ${err}`);
      return this.notfound;
    }
  }
}

module.exports = AliOSSAdapter;
```

这里支持的配置内容有：

```json
{
  "storage": {
    "active": "ghost-aliyun-oss-store",
    "ghost-aliyun-oss-store": {
      "accessKeyId": "YOUR_ACCESS_KEY_ID",
      "accessKeySecret": "YOUR_ACCESS_SERCET",
      "bucket": "YOUR_BUCKET_NAME",
      "region": "oss-cn-beijing",
      "domain": "https://your-public-domian",
      "notfound": "https://s3-img.meituan.net/v1/mss_3d027b52ec5a4d589e68050845611e68/ff/n0/0k/4n/3s_73850.jpg"
    }
  }
}
```

其中 `domian`、是可选项，如果你需要使用 CDN 域名，请在这个字段里配置。

## 封装支持 OSS 插件的镜像

为了保证运行镜像性能足够高、尺寸相对较小，我们需要使用 [docker multistage build](https://docs.docker.com/develop/develop-images/multistage-build/)方案。

先定义基础镜像，并安装刚刚编写的 Ghost Aliyun OSS 插件。

```bash
FROM ghost:3.9.0-alpine as oss
LABEL maintainer="soulteary@gmail.com"
WORKDIR $GHOST_INSTALL/current
RUN su-exec node yarn --verbose add ghost-aliyun-oss-store
```

接着定义运行使用的镜像。

```bash
FROM ghost:3.9.0-alpine
LABEL maintainer="soulteary@gmail.com"
COPY --chown=node:node --from=oss $GHOST_INSTALL/current/node_modules $GHOST_INSTALL/current/node_modules
RUN mkdir -p $GHOST_INSTALL/current/content/adapters/storage/
RUN echo "module.exports = require('ghost-aliyun-oss-store');" > $GHOST_INSTALL/current/content/adapters/storage/ghost-aliyun-oss-store.js
```

参考之前的两篇文章，如果想解决“不能正常进行中文输入”的问题，并且提取出了构建后的内容，可以在镜像中添加下面的内容：

```bash
COPY ./docker-assets/admin-views  $GHOST_INSTALL/current/core/server/web/admin/views
COPY ./docker-assets/built/assets $GHOST_INSTALL/current/core/built/assets
```

如果你不希望将配置单独抽象为文件，可以添加下面的内容。

```bash
RUN set -ex; \
    su-exec node ghost config storage.active ghost-aliyun-oss-store; \
    su-exec node ghost config storage.ghost-aliyun-oss-store.accessKeyId YOUR_ACCESS_KEY_ID; \
    su-exec node ghost config storage.ghost-aliyun-oss-store.accessKeySecret YOUR_ACCESS_SERCET; \
    su-exec node ghost config storage.ghost-aliyun-oss-store.bucket YOUR_BUCKET_NAME; \
    su-exec node ghost config storage.ghost-aliyun-oss-store.region oss-cn-beijing; \
    su-exec node ghost config storage.ghost-aliyun-oss-store.domain https://your-public-domian; \
    su-exec node ghost config storage.ghost-aliyun-oss-store.notfound https://s3-img.meituan.net/v1/mss_3d027b52ec5a4d589e68050845611e68/ff/n0/0k/4n/3s_73850.jpg; \
    su-exec node ghost config privacy.useUpdateCheck false; \
    su-exec node ghost config privacy.useGravatar false; \
    su-exec node ghost config privacy.useRpcPing false; \
    su-exec node ghost config privacy.useStructuredData false; \
```

当然，为了更方便更新内容，抽象为单独的文件是更好的选择，比如像下面这样编写 `config.production.json` 配置文件。

```json
{
  "server": {
    "port": 2368,
    "host": "0.0.0.0"
  },
  "privacy": {
    "useUpdateCheck": false,
    "useGravatar": false,
    "useRpcPing": false,
    "useStructuredData": false
  },
  "storage": {
    "active": "ghost-aliyun-oss-store",
    "ghost-aliyun-oss-store": {
      "accessKeyId": "YOUR_ACCESS_KEY_ID",
      "accessKeySecret": "YOUR_ACCESS_SERCET",
      "bucket": "baai-news-upload",
      "region": "oss-cn-beijing",
      "domain": "https://your-public-domian",
      "notfound": "https://s3-img.meituan.net/v1/mss_3d027b52ec5a4d589e68050845611e68/ff/n0/0k/4n/3s_73850.jpg"
    }
  }
}
```

## 完整的容器编排文件

上面聊了许多定制化的选项，那么一个最小可用的容器编排配置是什么样的呢？其实大概不到十行，就足以满足我们的基础需求。

```bash
FROM ghost:3.9.0-alpine as oss
WORKDIR $GHOST_INSTALL/current
RUN su-exec node yarn --verbose add ghost-aliyun-oss-store

FROM ghost:3.9.0-alpine
LABEL maintainer="soulteary@gmail.com"
COPY --chown=node:node --from=oss $GHOST_INSTALL/current/node_modules $GHOST_INSTALL/current/node_modules
RUN mkdir -p $GHOST_INSTALL/current/content/adapters/storage/
RUN echo "module.exports = require('ghost-aliyun-oss-store');" > $GHOST_INSTALL/current/content/adapters/storage/ghost-aliyun-oss-store.js
```

将上面的内容保存为 `Dockerfile`，如果需要其他的功能，可以参考上面的内容进行适当修改。

```bash
docker build -t soulteary/ghost-with-oss:3.9.0 -f Dockerfile .
```

执行上面的命令，稍等片刻，一个衍生自Ghost官方镜像，支持 OSS 的容器镜像就构建完毕了。

## 如何使用镜像

这里给出一个完整编排文件供大家参考，如果不想使用 Traefik，只需要将端口单独暴露出来即可。

至于 Traefik 如何使用，参考我[以往的文章](https://soulteary.com/tags/traefik.html)，熟悉之后，你将会发现一片新的天地。

```yaml
version: "3.6"

services:

  ghost-with-oss:
    image: soulteary/ghost-with-oss:3.9.0
    expose:
      - 2368
    environment:
      url: https://ghost.lab.io
      database__client: mysql
      database__connection__host: ghost-db
      database__connection__port: 3306
      database__connection__user: root
      database__connection__password: ghost
      database__connection__database: ghost
      NODE_ENV: production
    volumes:
      # 这里参考前篇文章，或者本篇文章内容，选择性使用
      # 解决 Ghost 中文输入的问题
      # - ./docker-assets/built/assets:/var/lib/ghost/versions/current/core/built/assets:ro
      # - ./docker-assets/admin-views:/var/lib/ghost/current/core/server/web/admin/views:ro
      - ./config.production.json:/var/lib/ghost/config.production.json    
    extra_hosts:
      - "ghost.lab.io:127.0.0.1"
    networks:
      - traefik
    labels:
      - "traefik.enable=true"
      - "traefik.docker.network=traefik"
      - "traefik.http.routers.ghostweb.entrypoints=http"
      - "traefik.http.routers.ghostweb.middlewares=https-redirect@file"
      - "traefik.http.routers.ghostweb.rule=Host(`ghost.lab.io`)"
      - "traefik.http.routers.ghostssl.middlewares=content-compress@file"
      - "traefik.http.routers.ghostssl.entrypoints=https"
      - "traefik.http.routers.ghostssl.tls=true"
      - "traefik.http.routers.ghostssl.rule=Host(`ghost.lab.io`)"
      - "traefik.http.services.ghostbackend.loadbalancer.server.scheme=http"
      - "traefik.http.services.ghostbackend.loadbalancer.server.port=2368"

networks:
  traefik:
    external: true
```

将上面内容保存为 `docker-compose.yml`，使用 `docker-compose up -d` 启动应用，最后访问配置里定义的域名即可开始使用这个支持 OSS 功能的 Ghost 。

当然，如果你没有线上数据库，也可以使用 `docker-compose` 启动一个数据库：

```yaml
version: '3'
services:

  db:
    image: mysql:5.7
    container_name: ghost-db
    expose:
      - 3306
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

## 最后

本篇内容，以封装 Ghost 定制镜像简单说明了如何基于官方镜像进行扩展，并简单示范了 Docker Multistage Build，以及 Ghost 3.x 版本如何使用 Aliyun OSS。

或许下一篇内容会聊聊，Ghost 这类原本不支持 SSO 单点登录的应用如何快速接入 SSO。

--EOF
