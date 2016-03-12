
# 使用 Docker 和 Node 快速实现一个在线的 QRCode 解码服务

本文将会介绍如何使用 Docker、Node、JavaScript、Traefik完成一个简单的二维码解析服务，全部代码在 300 行以内。

最近折腾文章相关的东西比较多，其中有一个现代化要素其实挺麻烦的，就是**二维码**。

<!-- more -->

![最终结果预览](https://attachment.soulteary.com/2018/12/09/preview.png)

不论是“生成动态、静态的二维码”，还是“对已经生成的二维码进行解析”，其实都不难实现。只是在日常工作中如果只是基于命令行去操作，会很不方便。

所以花了点时间，实现了一个简单的 QRCode 在线解析工具，在完成这个工具之后，原本需要“打开终端，定位文件，执行命令，等待结果”就简化成了“打开网页，`CTRL+V` 粘贴，片刻展示结果”，当然，因为额外提供了接口，所以也可以当一个无状态服务使用。

## 实现服务端核心解析逻辑

核心逻辑其实很简单，伪代码三行就差不多了，比如。

```js
const uploadContentByUser = req.body.files;
const decodeContent = decodeImage(uploadContentByUser);
const result = decodeQR.load(decodeContent);
```

但是实际使用的情况，出于性能的考虑，我不会过分使用新语法进行代码封装，更倾向尽可能使用“原生”的回调模式进行异步编程，避免各种“wrapper”造成不必要的损耗。

因为最终的目的是“在浏览器里一个粘贴/拖拽操作就完事”。所以我们需要将上面的核心逻辑展开，根据“简单项目不过度封装”的思想，代码会膨胀为下面三十行左右的样子。

```js
app.post('/api/decode', multipartMiddleware, function(req, res) {
  let filePath = '';

  try {
    if (req.files.imageFile.path) filePath = req.files.imageFile.path;
  } catch (e) {
    return res.json({code: 500, content: 'request params error.'});
  }

  fs.readFile(filePath, function(errorWhenReadUploadFile, fileBuffer) {
    if (errorWhenReadUploadFile) return res.json({code: 501, content: 'read upload file error.'});
    decodeImage(fileBuffer, function(errorWhenDecodeImage, image) {
      if (errorWhenDecodeImage) return res.json({code: 502, content: errorWhenDecodeImage});
      let decodeQR = new qrcodeReader();
      decodeQR.callback = function(errorWhenDecodeQR, result) {
        if (errorWhenDecodeQR) return res.json({code: 503, content: errorWhenDecodeQR});
        if (!result) return res.json({code: 404, content: 'gone with wind'});
        return res.json({code: 200, content: result.result, points: result.points});
      };
      decodeQR.decode(image.bitmap);
    });
  });
});
```

上面的逻辑很简单，主要做了下面几件事：

- 接受用户上传的文件
- 读取用户上传的文件
- 解析用户上传的文件
- 尝试将文件中的信息解码并反馈用户

其中依赖了一个 `express` 三方的中间件 `multipartMiddleware`，我将主要使用它来进行上传文件的请求序列化，源码十分简洁，一百行左右，有兴趣可以去浏览一下。

它的使用也十分简单，无需配置，只需要两行就能发挥作用。

```js
const multipart = require('connect-multiparty');
const multipartMiddleware = multipart();
```

当然，为了能够配合客户端 JavaScript 完成我们的最终目标，我们需要一些额外的代码，比如：提供一个浏览器可以浏览的页面。

这里额外提一点，如果使用类 express 的框架，一般会有一个 `static` 方法，让你设置一个静态文件目录，可以免编程路由逻辑对一些文件进行对外访问，比如这样：

```js
app.use(express.static(__dirname + '/static', {dotfiles: 'ignore', etag: false, extensions: ['html'], index: false, maxAge: '1h', redirect: false}));
```

但是，本例中我其实只需要一个入口页面就能满足需求，根本不需要外部资源，比如 `vue`、`react`、`jq`、`各种css框架`…

这个时候，我推荐直接将要展示的页面使用 `fs` API 进行内存缓存，直接提供用户即可，比如按照下面的代码进行编写，大概十行就能满足需求。

```js
const indexCache = fs.readFileSync('./index.html');

app.get('/', function(req, res) {
  res.redirect('/index.html');
});

app.get('/index.html', function(req, res) {
  res.setHeader('charset', 'utf-8');
  res.setHeader('Content-Type', 'text/html');
  res.send(indexCache);
});
```

当然，如果你想要和 `static` 方式的文件一样，在调试过程中，可以“热更新”文件的话，需要将这个 `indexCache` 改写成一个方法，在拦截用户请求之后，每次都去动态读取文件，或者更高阶一些，根据文件最后编辑时间戳，实现一个简单的 LRU 缓存。


## 实现客户端交互逻辑

在实现完毕接口后，我们把欠缺的前端交互逻辑补全。

这里因为没有什么重度的操作，界面也很简单，所以既不需要 `jQ` 这类库，也不需要 `Vue`、`React` 这类框架，直接写脚本就是了。

脑补我需要的界面，上面是一个数据交互的区域，下面是我的交互结果列表，因为页面也没几个元素，所以直接使用脚本进行元素的创建和操作吧。

```js
let uploadBox = document.createElement('textarea');
uploadBox.id = 'upload';
uploadBox.placeholder = 'Paste Here.';
document.body.appendChild(uploadBox);

let list = document.createElement('ul');
list.id = 'result';
document.body.appendChild(list);
```

浏览器端核心的操作有三个：

- 接受用户的拖拽和粘贴图片的操作
- 将用户给予的图片数据进行上传
- 对服务端接口解析的结果进行展示

我们先来实现第一个操作，拖拽、粘贴富交互功能，大概三十行代码就能解决战斗。

```js
function getFirstImage(data, isDrop) {
  let i = 0, item;
  let target = isDrop ?
      data.dataTransfer && data.dataTransfer.files :
      data.clipboardData && data.clipboardData.items;

  if (!target) return false;

  while (i < target.length) {
    item = target[i];
    if (item.type.indexOf('image') !== -1) return item;
    i++;
  }
  return false;
}

function getFilename(event) {
  return event.clipboardData.getData('text/plain').split('\r')[0];
}

uploadBox.addEventListener('paste', function(event) {
  event.preventDefault();
  const image = getFirstImage(event);
  if (image) return uploadFile(image.getAsFile(), getFilename(event) || 'image.png');
});

uploadBox.addEventListener('drop', function(event) {
  event.preventDefault();
  const image = getFirstImage(event, true);
  if (image) return uploadFile(image, event.dataTransfer.files[0].name || 'image.png');
});
```

如果你需要支持多张图片上传，服务端接口需要做一个简单的改动，我没有这个需求，就不做了，有兴趣可以实践下，理论上加两个循环就完事。

接着我们继续实现上传功能，因为现代的浏览器都支持了 `fetch`，所以实现起来也很简单，二十多行解决战斗：

```js
function getMimeType(file, filename) {
  if (!file) return console.warn('不支持该文件类型');
  const mimeType = file.type;
  const extendName = filename.substring(filename.lastIndexOf('.') + 1);
  if (mimeType !== 'image/' + extendName) return 'image/' + extendName;
  return mimeType;
}

function uploadFile(file, filename) {
  let formData = new FormData();
  formData.append('imageFile', file);

  let fileType = getMimeType(file, filename);
  if (!fileType || ['jpg', 'jpeg', 'gif', 'png', 'bmp'].indexOf(fileType) > -1) return console.warn('文件格式不正确');

  formData.append('mimeType', fileType);

  fetch('/api/decode', {method: 'POST', body: formData}).
      then((response) => response.json()).
      then((data) => {
        if (data.code === 200) return addResult(filename, data.content);
        return addResult(filename, data.content);
      }).
      catch((error) => addResult(filename, error));
}
```

最后，写几条样式规则，额外优化一下解析结果展示就完事了，比如能够更轻松的复制解析结果。

```js
list.addEventListener('mouseover', function(e) {
  let target = e.target;
  if (target && target.nodeName) {
    if (target.nodeName.toLowerCase() === 'input') {
      target.select();
    }
  }
});

function result(file, text) {
  let li = document.createElement('li');
  li.innerHTML = '<b>' + file + '</b>' + '<input value="' + text + '">';
  document.getElementById('result').appendChild(li);
}
```

## 将程序容器化

如果你认真阅读了上面的文章，你会发现，实际的程序只有两个文件，一个是服务端的 Node 程序，另外一个则是我们的客户端页面，但是实际上，我们还需要一个记录 Node 依赖的 `package.json` 以及一个用户构建容器镜像的 `Dockerfile`，最简化的目录结构如下：

```bash
.
├── Dockerfile
├── index.html
├── index.js
└── package.json
```

考虑实际维护，我们还需要额外创建一些其他的问题，不过都不重要，相关的文件内容，可以浏览我稍后提供的源码仓库。

此刻，当我们执行 `node index.js`，然后在浏览器中打开 `localhost:3000` 就能实现文章一开头我们提到的**一键粘贴完成对二维码的解析**操作了。

不过为了部署的便捷，我们还是需要将程序进行容器化操作。我们来着重浏览一下容器构建文件，同样很简单，几行就足够我们的使用。

```bash
FROM node:11.4.0-alpine
MAINTAINER soulteary <soulteary@gmail.com>

RUN apk update && apk add yarn
WORKDIR /app
COPY .  /app
RUN yarn

ENTRYPOINT [ "node", "index.js" ]
```

配合简单的构建命令：

```bash
docker build -t 'docker.soulteary.com/decode-qrcode.soulteary.com:0.0.1' .
```

稍等一两分钟，就能够获得一个可以脱离当前环境，随处运行的容器镜像了。如果你想让容器运行起来，也只需要一条命令，即可。

```bash
docker run -it -p 3000:3000 'docker.soulteary.com/decode-qrcode.soulteary.com:0.0.1'
```

如果每次都使用这样的命令，未免麻烦，我们不妨使用 `compose` 配合 `Traefik` 进行服务化。

## 配合 Traefik 进行服务化操作

配合 compose 和 Traefik 使用起来非常简单，我之前的文章有提过多次，所以这里就简单贴出配置文件示例：

```yaml
version: '3'

services:

  decode:
    image: docker.soulteary.com/decode-qrcode.soulteary.com:0.0.1
    expose:
      - 3000
    networks:
      - traefik
    labels:
      - "traefik.enable=true"
      - "traefik.port=3000"
      - "traefik.frontend.rule=Host:decode-qrcode.lab.com"
      - "traefik.frontend.entryPoints=http,https"

networks:
  traefik:
    external: true

```

然后使用 `docker-compose -f compose.yml up -d` 即可自动启动服务，并将服务自动注册到 Traefik 的服务发现上。

如果需要扩容，`scale decode=4` 即可，如果还不会操作，可以翻阅之前的文章，进一步学习，: )

## 最后

附上完整示例代码：[ https://github.com/soulteary/decode-your-qrcode ](https://github.com/soulteary/decode-your-qrcode)

最近结束了休假，换了新公司，手头事情比较多，写文章的速度会慢一些，不过没有关系，草稿箱里的东西积累的再多一些，文章的质量会再上一层楼，一起期待一下吧。
