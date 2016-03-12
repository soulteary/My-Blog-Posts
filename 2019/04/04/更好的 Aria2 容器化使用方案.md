# 更好的 Aria2 容器化使用方案

日常偶尔会下载资源，趁着整理 HomeLab 的机会，把 Aria2 封装成了容器镜像。

本文将介绍如何搭配 Traefik 快速使用 Aria2 ，以及如何调整不兼容 Traefik 的应用与之兼容。

## 镜像介绍

在开始讲解如何做之前，需要先简单介绍一下镜像的构成。

对于长期运行的应用/服务来说，代码是否透明公开很重要，我已经将代码上传至 GitHub：[代码仓库地址](https://github.com/soulteary/traefik-aria2-with-webui)。

使用到的技术栈/工具主要有以下内容：

- Aria2 WebUI
	  - 一款Aria2 Web管理界面 [ziahamza/webui-aria2](https://github.com/ziahamza/webui-aria2)
- Aria2
	  - Aria2 下载工具 [ndthuan/aria2-alpine](https://github.com/ndthuan/aria2-alpine)
- Node HTTP Proxy
	  - Node.js 版的代理模块[nodejitsu/node-http-proxy](https://github.com/nodejitsu/node-http-proxy)
- Traefik
	  - 优秀的负载均衡/服务发现工具 [containous/traefik](https://github.com/containous/traefik)
- Compose
	  - 简单好用的容器编排工具 [docker/compose](https://github.com/docker/compose)
- Docker
	  - 简单好用的容器工具 [docker](https://github.com/docker)

## 改造过程

网上盛行使用一个容器同时提供 **HTTP** + **ARIA2** 服务，但是这种胖容器其实不符合“单一进程单一容器”的原则，更通俗的说，这样使用容器，和使用 VM 虚拟机没有差别。

所以在使用的过程中，我们需要单独运行这两个部分，使用 `docker-compose.yml` 定义的话，或许最简单的示例就是下面这样了：

```yaml
version: '3'

services:

  web:
    container_name: web
    image: ${WEBUI_IMAGE}
    expose:
      - 8888

  aria2:
    container_name: aria2
    image: ${ARIA2_IMAGE}
    volumes:
      - ./downloads:/downloads
    expose:
      - 6800

```

但如果你将 **Aria2 WebUI** 直接使用在容器里，你会发现这个 Web 管理界面将 RPC 端口写死为了 6800 ，因为项目设计的时候，只考虑了同机部署，并且是 Aria2 在默认配置执行的情况。

之所以这样处理，是因为项目本身是一套使用 Angular 编写的 SPA 程序，不具备服务端接口处理能力，但是实际上，项目在生产环境运行的时候，作者有提供一个简单的 Node Server，是可以支持一定程度的接口定制的。

而我们使用容器将 Web UI 和 Aria2 进行隔离，相当于在两台不同的服务器中执行这两个应用，所以这里我们要进行一些改造。

### WebUI 支持和 Aria2 分离部署

影响 WebUI 使用 Aria2 RPC 接口的主要有两处引用，分别在：

```yaml
/app/src/js/services/configuration.js
/app/src/js/services/rpc/rpc.js
```

先处理 `configuration.js` 的内容：

```js
host: location.protocol.startsWith("http") ? location.hostname : "localhost",
    path: "/jsonrpc",
    port: 6800,
    encrypt: false,
```

找到程序中上面的代码部分，将端口 `6800` 修改为 `80`。

接着处理 `rpc.js` 的内容：

```js
if (["http", "https"].indexOf(uri.protocol()) != -1 && uri.host() != "localhost") {
                configurations.push(
                    {
                        host: uri.host(),
                        path: "/jsonrpc",
                        port: 6800,
                        encrypt: false
                    },
```

同样找到这段代码，将端口 `6800` 修改为 `80`。

将代码改造完毕，你会发现WebUI中所有的请求都被定位到了当前域名下的 `80` 端口，前面提过，这个界面是一套前端 SPA 应用，是缺乏接口处理能力的。

所以，接下来我们要对服务端进行处理，我使用 ES6 重写了作者的 Node Server，主要支持了 `HTTP`/`WS` 协议转发至 `aria2` 容器对应的端口。

```js
const { createServer } = require("http");
const url = require("url");
const { join, extname } = require("path");
const { exists, statSync, readFile } = require("fs");

const port = parseInt(process.argv[2], 10) || 8888;
const baseDir = join(process.cwd(), "docs");

const httpProxy = require('http-proxy');

const { WEB_PROXY, WS_PROXY } = process.env;
const webProxy = httpProxy.createProxyServer({ target: WEB_PROXY });
const wsProxy = httpProxy.createServer({ target: WS_PROXY, ws: true });

createServer(function (request, response) {
    const uri = url.parse(request.url).pathname;
    let filename = join(baseDir, uri);

    let contentType = "text/html";
    switch (extname(filename)) {
        case ".js":
            contentType = "text/javascript";
            break;
        case ".css":
            contentType = "text/css";
            break;
        case ".ico":
            contentType = "image/x-icon";
            break;
        case ".svg":
            contentType = "image/svg+xml";
            break;
    }

    if (filename.endsWith("jsonrpc")) {
        webProxy.web(request, response);
        return;
    }

    exists(filename, function (exists) {
        if (!exists) {
            response.writeHead(404, { "Content-Type": "text/plain" });
            response.write("404 Not Found\n");
            response.end();
            return;
        }

        if (statSync(filename).isDirectory()) filename += "/index.html";

        readFile(filename, "binary", function (err, file) {
            if (err) {
                response.writeHead(500, { "Content-Type": "text/plain" });
                response.write(`${err}\n`);
                response.end();
                return;
            }
            response.writeHead(200, { "Content-Type": contentType });
            response.write(file, "binary");
            response.end();
        });
    });
}).on('upgrade', function (req, socket, head) {
    wsProxy.ws(req, socket, head);
}).listen(port);

console.log(`WebUI Aria2 Server is running on http://localhost:${port}`);
```

当用户访问管理界面的时候，静态资源会直接使用 Node.js 的 HTTP 模块提供服务。而当用户进行下载状态查询，新增下载任务的时候，将通过 Node.js 调用 Aria2 的接口进行处理，并将结果使用 `HTTP` 或 `WS` 反馈给用户。

### 构建镜像

为了方便部署和维护，我们需要将上面的改动记录在 Dockerfile 中。

```js
FROM node:11.12-alpine
LABEL AUTHOR soulteary

ADD webui-aria2/package.json /app/package.json
ADD webui-aria2/package-lock.json /app/package-lock.json

WORKDIR /app

RUN npm install -g yarn && yarn

ADD webui-aria2/ /app

ADD ./patches/configuration.js /app/src/js/services/configuration.js
ADD ./patches/rpc.js /app/src/js/services/rpc/rpc.js

RUN yarn build

RUN yarn add http-proxy
ADD ./patches/node-server.js /app/node-server.js

CMD [ "node", "./node-server.js" ]
```

执行 `docker build -t soulteary/traefik-aria2-with-webui .` ，稍等片刻你将得到封装好的容器镜像。

接下来便是使用容器编排工具，正式使用啦。

### 编排应用

Aria2 的镜像，可以使用 `ndthuan/aria2-alpine`，无须重新封装，当然，如果你需要一些新的特性，也可以参考镜像提供的 Dockerfile，进行重新构建。

创建一个 `docker-compose.yml`，使用下面的示例代码：

```yaml
version: '3'

services:

  web:
    container_name: web
    image: ${WEBUI_IMAGE}
    expose:
      - 8888
    networks:
      - traefik
    environment:
      - WEB_PROXY=http://aria2:6800
      - WS_PROXY=ws://aria2:6800
    labels:
      - "traefik.enable=true"
      - "traefik.port=8888"
      - "traefik.frontend.rule=Host:${BIND_HOSTS}"
      - "traefik.frontend.entryPoints=http,https"

  aria2:
    container_name: aria2
    image: ${ARIA2_IMAGE}
    volumes:
      - ./downloads:/downloads
    expose:
      - 6800
    networks:
      - traefik
    labels:
      - "traefik.enable=false"

networks:
  traefik:
    external: true
```

然后再创建一个 `.env` 文件，在其中填入：

```Toml
WEBUI_IMAGE=soulteary/traefik-aria2-with-webui
ARIA2_IMAGE=ndthuan/aria2-alpine

BIND_HOSTS=download.lab.com,download.lab.io
```

使用 `docker-compose up`，你将看到类似下面的日志：

```Toml
Creating web   ... done
Creating aria2 ... done
Attaching to aria2, web
aria2    | 2019-04-04T04:28:41Z 48a00033e530 confd[7]: INFO Backend set to env
aria2    | 2019-04-04T04:28:41Z 48a00033e530 confd[7]: INFO Starting confd
aria2    | 2019-04-04T04:28:41Z 48a00033e530 confd[7]: INFO Backend nodes set to
aria2    | 2019-04-04T04:28:41Z 48a00033e530 confd[7]: INFO Target config /etc/aria2.conf out of sync
aria2    | 2019-04-04T04:28:41Z 48a00033e530 confd[7]: INFO Target config /etc/aria2.conf has been updated
aria2    |
aria2    | 04/04 04:28:41 [WARN] Neither --rpc-secret nor a combination of --rpc-user and --rpc-passwd is set. This is insecure. It is extremely recommended to specify --rpc-secret with the adequate secrecy or now deprecated --rpc-user and --rpc-passwd.
aria2    |
aria2    | 04/04 04:28:41 [NOTICE] IPv4 RPC: listening on TCP port 6800
aria2    |
aria2    | 04/04 04:28:41 [NOTICE] IPv6 RPC: listening on TCP port 6800
web      | WebUI Aria2 Server is running on http://localhost:8888
```

在浏览器中打开你在 `.env` 中定义的域名，不出意外，你将看到属于你的下载应用。

![搭建好的 Aria Web UI 界面](https://attachment.soulteary.com/2019/04/04/final.png)

## 最后

改变习惯不易，尤其是你周围的环境都在使用其他看起来更“标准成熟”的方案时。

但是一旦当你用惯了 Traefik + Docker 之后，你会发现你的服务搭建效率远比使用 Nginx 加 vhost 高的多。

最近快忙晕了，临近小长假，补全了这篇拖了十天的文章，仓促成文，难免有纰漏，欢迎留言指正。