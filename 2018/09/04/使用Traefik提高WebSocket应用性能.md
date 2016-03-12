# 使用 Traefik 提高 WebSocket 应用性能

说起 `Node.js` 的 `WebSocket` 方案，可选的方案有许多种，其中许多方案都提供将 `WS` 服务端口和 `HTTP` 服务复用的方案，然而这种方案真的是最佳选择吗。

不论是专业做实时通信的 [socket.io](https://github.com/socketio/socket.io) ，还是用户量最大的 `Express` 的热门中间件 [express-ws](https://github.com/HenningM/express-ws) 都支持端口复用，比如 `WS` 和 `HTTP` 复用 `80` 端口， `WSS` 和 `HTTPS` 复用 `443` 端口。

这里以 `express-ws` 底层封装的 [ws](https://github.com/websockets/ws) 库为例，来简单剖析，`socket.io` 实现类似不过分层较多，有兴趣可以围观代码。

不过在聊 `Traefik` 之前，我们先得聊聊 `Node.js` 和 `Websocket`。

## 关于同域名端口复用

先说结论，优点：

1. 使用简单，尤其是整个项目代码量少的时候。
2. 服务域名复用，不需要额外进行域名解析。
3. 能够简单获取 `HTTP` 请求中的会话信息，进行简单的验证操作，能够代码级复用逻辑。

缺点也很明显：

1. 因为复用端口，对于每个数据都需要甄别是应该交给 `Express` 处理还是 `WS` 处理，存在性能损耗，如果需要进行压缩等操作，会有更多的损耗。
2. 相同域名不易进行业务水平扩展，比如需要支持更多的实时业务，原本扩容3实例的 `WS` 服务即可，由于耦合，不得不将整个服务进行扩展，存在更多资源的损耗。
3. 由于耦合，复杂度相比较“各自独立”的版本高，在维护过程，如果修改底层代码，难免会让两个服务都不够健壮稳定。

### 从代码实现角度围观端口复用

[express-ws](https://github.com/HenningM/express-ws/blob/master/src/index.js) 进行端口复用的时候，会进行大量 hacks 操作，包括扩展路由、改写请求地址添加特殊标记、重写默认响应头...

下面这段示例是官方给出的端口复用的例子。

```js
var express = require('express');
var app = express();
var expressWs = require('express-ws')(app);

app.use(function (req, res, next) {
  console.log('middleware');
  req.testing = 'testing';
  return next();
});

app.get('/', function(req, res, next){
  console.log('get route', req.testing);
  res.end();
});

app.ws('/', function(ws, req) {
  ws.on('message', function(msg) {
    console.log(msg);
  });
  console.log('socket', req.testing);
});

app.listen(3000);
```

实际使用的时候，访问 `WS` 的 `/`，会访问 `Express` 的 `/.websocket?{QUERY}`，并使用中间件注入处理过程的方式，抢在默认处理前使用 `ws` [替换处理过程](https://github.com/websockets/ws/blob/master/lib/websocket-server.js)，修改响应头，输出处理后的内容，并调用 `res.end` 结束流程。

在路由越来越多、请求量越来越多的情况下，会存在很多不必要的损耗。

## 如何进行服务拆分

如果不需要端口复用，其实直接使用 `ws` 来监听独立的新端口即可，参考官方示例，可以很轻松的写出这样一个例子：

```js
const WebSocket = require('ws');

const wss = new WebSocket.Server({
  port: 8080,
  perMessageDeflate: {
    zlibDeflateOptions: { // See zlib defaults.
      chunkSize: 1024,
      memLevel: 7,
      level: 3,
    },
    zlibInflateOptions: {
      chunkSize: 10 * 1024
    },
    // Other options settable:
    clientNoContextTakeover: true, // Defaults to negotiated value.
    serverNoContextTakeover: true, // Defaults to negotiated value.
    clientMaxWindowBits: 10,       // Defaults to negotiated value.
    serverMaxWindowBits: 10,       // Defaults to negotiated value.
    // Below options specified as default values.
    concurrencyLimit: 10,          // Limits zlib concurrency for perf.
    threshold: 1024,               // Size (in bytes) below which messages
                                   // should not be compressed.
  }
});

wss.on('connection', function connection(ws) {
  ws.on('message', function incoming(message) {
    console.log('received: %s', message);
  });

  ws.send('something');
});

server.listen(8080);
```

`HTTP` 服务监听在另外一个端口，可以参考 `Express` 最简单的示例:

```js
const express = require('express')
const app = express()

app.get('/', (req, res) => res.send('Hello World!'))

app.listen(3000, () => console.log('Example app listening on port 3000!'))
```

这里分别将代码片段进行保存，当你分别使用 `Node.js` 执行它的时候，你将会得到监听 `3000` 端口和 `8080` 端口的简单服务，支持使用 `WS` 和 `HTTP` 进行数据交互。

### 这样的服务的优势和不足

优势：

1. 可以轻松针对不同协议的服务进行扩容操作。
2. 彼此运行时资源隔离，安全性和稳定性更好。
3. 可以使用相同域名、不同端口部署，也可以使用不同域名，默认端口进行部署，部署选择也更多。

劣势：

1. 在不依赖 `RDS` 、`Redis Cache` 等方案的前提下，请求之间的数据难以共享。
2. 如果需要都使用默认端口进行部署，那么需要额外进行一个域名的解析。

## 搭配 Traefik 使用

我将 `SSL` 证书挂载和 `HTTP` 压缩放在 `Traefik` 端处理，相比较 `Node.js` 来做，一来可以保障业务代码功能独立纯粹，二来性能确实不如它，而且维护起来也比较麻烦(证书管理)。

对于接入网关的服务，只要声明提供 `HTTP` 和 `WS` 的端口和对应的域名即可，程序启动之后，`Traefik` 会自动将应用挂载到对应域名上，并支持 `HTTP(S)` 和 `WS(S)` 的服务。

为图简便，我将上面的代码片段保存为一个基础镜像，交付给编排工具使用。

如果你将上面的代码片段保存为一个文件，可以试试下面的配置：

```yml
version: '3'

services:

  node:
    image: docker.lab.com/example.lab.com:0.0.1
    restart: always
    labels:
      - "traefik.enable=true"
      - "traefik.web.port=3000"
      - "traefik.web.frontend.rule=Host:web.soulteary.com"
      - "traefik.ws.port=8080"
      - "traefik.ws.frontend.rule=Host:ws.soulteary.com"
    networks:
      - traefik
    expose:
      - 3000
      - 8080
    extra_hosts:
      - "web.soulteary.com:127.0.0.1"
      - "ws.soulteary.com:127.0.0.1"

networks:
  traefik:
    external: true
```

使用上面的配置运行之后，你会发现原本的 `3000` 端口和 `8080` 端口，都被“改写”成为了 `80` 和 `443` 端口上了，Web 应用使用的时候，便不用额外写入“丑陋”的端口号了，但是这样的配置不利于服务扩展，在端口复用优劣小节中我提到过。

那么，如果你有意将代码进行拆分，那么可以试试下面的配置：

```yml
version: '3'

services:

  web:
    image: docker.lab.com/example.lab.com:0.0.1
    restart: always
    labels:
      - "traefik.enable=true"
      - "traefik.web.port=3000"
      - "traefik.web.frontend.rule=Host:web.soulteary.com"
    networks:
      - traefik
    expose:
      - 3000
    extra_hosts:
      - "web.soulteary.com:127.0.0.1"

  ws:
    image: docker.lab.com/example.lab.com:0.0.1
    restart: always
    labels:
      - "traefik.enable=true"
      - "traefik.ws.port=8080"
      - "traefik.ws.frontend.rule=Host:ws.soulteary.com"
    networks:
      - traefik
    expose:
      - 8080
    extra_hosts:
      - "ws.soulteary.com:127.0.0.1"

networks:
  traefik:
    external: true
```

扩容也很简单，如果你要以 2:3 的比例运行不同协议的话，只需要：

```bash
docker-compose scale web=2 ws=3
```

## 其他

如果你还在使用 `ajax polling` 或许这个方案可以给你更好的体验。

如果你对 Traefik 期望有更多的了解，也欢迎和我沟通讨论。