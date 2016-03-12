
本文将会介绍如何使用 Docker、Node、JavaScript、Traefik 完成一个简单的目录索引服务，全部代码在 300 行以内。相关代码已开源至 GitHub ，文末有链接，感兴趣可以自取。

实现一个目录索引站点并不是什么难事，但是即便如此，需要考虑的事情也有很多，要实现非阻塞IO、要实现文件缓存、要实现SSL等等一系列稍微有些麻烦的事情，如何能在尽可能少编写代码的情况下，完成这个需求呢。

其实很简单，借助完善靠谱的开源项目们，本文最终实现例子效果如下。

![最终结果预览](https://attachment.soulteary.com/2018/12/16/preview.png)

## 实现核心逻辑

说到 Web 目录索引服务，我们一般会想到的就是大名鼎鼎的 Nginx 或者它的竞品们了。而它其中一个默认模块便提供了这个目录列表的功能: `ngx_http_autoindex_module`。

这个模块十分简单，在此我就不过度展开，有兴趣可以翻阅 Nginx 官方文档，了解这个模块提供的几个简单的指令。

对某个路由下的页面开启 `autoIndex` 可以轻松实现列目录的功能，比如这样：

```bash
location / {
    autoindex_format        html;
    autoindex_localtime     on;
    autoindex_exact_size    on;
    autoindex               on;
}
```

如果你简单使用上面的逻辑，你会得到一个黑底白字的页面，虽然能用，但是未免太过丑陋，查看生成文档源代码(由于代码高亮问题，使用 `xpre` 代替 `pre`)：

```html
<html>
<head><title>Index of /</title></head>
<body>
<h1>Index of /</h1><hr><xpre><a href="../">../</a>
<a href="a/">a/</a>                                                 16-Dec-2018 13:39                   -
<a href="b/">b/</a>                                                 16-Dec-2018 13:39                   -
<a href="c/">c/</a>                                                 16-Dec-2018 13:39                   -
</xpre><hr></body>
</html>
```

这个时候一般会有两个方案对默认的界面进行美化：

1. 编译一个支持定义模板的 Nginx 插件。
2. 使用 `ngx_http_addition_module` 模块手动进行模板美化。

第一种方案需要额外编译，有一定的额外维护成本、以及后续升级改造的不稳定因素。我们选择第二种方式，比如将上面的逻辑改造为：

```bash
location / {
    add_before_body         /autoindex/header.html;
    add_after_body          /autoindex/footer.html;

    autoindex_format        html;
    autoindex_localtime     on;
    autoindex_exact_size    on;
    autoindex               on;
}
```

代码生效后，你将得到这样的文档结果： 

```html
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>小站</title>
    <style>your code here</style>
</head>
<body><html>
<head><title>Index of /</title></head>
<body>
<h1>Index of /</h1><hr><xpre><a href="../">../</a>
<a href="a/">a/</a>                                                 16-Dec-2018 13:39                   -
<a href="b/">b/</a>                                                 16-Dec-2018 13:39                   -
<a href="c/">c/</a>                                                 16-Dec-2018 13:39                   -
</xpre><hr></body>
</html>
<table><thead><tr><th width="40%">Name</th><th width="30%">Date</th><th width="10%">Size</th></tr></thead><tbody></tbody><tfoot><tr><th colspan="3"><a href="https://soulteary.com" target="_blank">Proudly Powered By Nginx, Design By @soulteary</a></th></tr></tfoot></table>
<script>your code here</script>
</body>
</html>
```

这时你会发现样式似乎是正常了，但是会出现三个额外的问题：

1. 文档闭合不标准，存在多个文档闭合标签。
2. Nginx AutoIndex 默认生成的 HTML 文档存在内联样式标签，无法像三方模块一样进行页面定制。
3. 默认生成文档结构不利于SEO以及不利于页面样式定制。

但是很庆幸，Nginx 还提供了一个内置模块：`ngx_http_sub_module`。
这个模块拥有编程语言中 `replace` 函数的作用，配合少量的替换操作，我们可以将文档轻松改造成我们想要的结构。

```bash
location / {
    add_before_body         /autoindex/header.html;
    add_after_body          /autoindex/footer.html;

    autoindex_format        html;
    autoindex_localtime     on;
    autoindex_exact_size    on;
    autoindex               on;

    sub_filter          '<html>\r\n<head><title>Index of $uri</title></head>' '';
    sub_filter          '<body bgcolor="white">' '';
    sub_filter          '</body>' '';
    sub_filter          '<hr>' '';
    sub_filter          '<hr>' '';
    sub_filter          '</html>' '';
    sub_filter_once     on;

    charset             utf-8;
}
```

此刻，再查看文档，会发现文档已经十分适合进行样式改造了。

```html
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>小站</title>
    <style>your code here</style>
</head>
<body>
<body>
<h1>Index of /</h1><xpre><a href="../">../</a>
<a href="a/">a/</a>                                                 16-Dec-2018 13:39                   -
<a href="b/">b/</a>                                                 16-Dec-2018 13:39                   -
<a href="c/">c/</a>                                                 16-Dec-2018 13:39                   -
</xpre>

<table><thead><tr><th width="40%">Name</th><th width="30%">Date</th><th width="10%">Size</th></tr></thead><tbody></tbody><tfoot><tr><th colspan="3"><a href="https://soulteary.com" target="_blank">Proudly Powered By Nginx, Design By @soulteary</a></th></tr></tfoot></table>
<script>your code here</script>
</body>
</html>
```

在有一个良好的文档基础之后，我们可以使用 `JavaScript` 对它进行简单的增强，考虑到最基础浏览器的兼容问题，我们使用 `ES5` 标准进行逻辑书写，下面不到二十行的代码，可以让我们使用文档中的 `pre` 标签作为数据源，重新生成适合排版的模板。

```js
var dataSets = document.getElementsByTagName('pre')[0].innerHTML.split('\n');
var directoryUp = false;
var tpl = '';

for (var i = 0, j = dataSets.length - 1; i < j; i++) {
  var line = dataSets[i];
  if (line.indexOf('../') === -1) {
    line = line.match(/^(.*)\s+(\S+\s\S+)\s+(\S+)/);
    tpl += '<tr>' +
        '<td>' + line[1] + '</td>' +
        '<td>' + '<span class="date" datetime="' + new Date(line[2]) + '">' + line[2] + '</span>' + '</td>' +
        '<td>' + line[3] + '</td>' +
        '</tr>';
  } else {
    if (location.pathname !== '/') directoryUp = true;
  }
}

if (directoryUp) tpl = '<tr><td colspan="3"><a href="..">..</a></td></tr>' + tpl;
document.getElementsByTagName('tbody')[0].innerHTML = tpl;
```

当然，如果你想拥有更适合阅读的时间戳，可以引入一个名为 `timeago.js` 的脚本，配合下面的代码，让 Nginx 输出的时间戳变可读性变的更好。

```js
timeago().render(document.querySelectorAll('.date'));
```

## 借助容器快速服务化

因为我们并未对 Nginx 进行任何改造，所以我们可以很省事的直接使用 Nginx 官方镜像提供我们的目录索引服务，这里推荐使用 `alpine` 镜像，小巧好用，比如下面的镜像，连带系统到软件，不到 **20 MB**。

```plain
nginx:1.15.7-alpine
```

为了简单，我直接使用 `compose` 和 `Traefik` 完成搭建应用的最后一步，相关的说明之前的博客有写，我就不赘述了，还是不太会使用的同学请翻阅历史文档。

```yaml
version: '3'

services:

  nginx:
    image: nginx:1.15.7-alpine
    restart: always
    labels:
      - "traefik.enable=true"
      - "traefik.port=80"
      - "traefik.frontend.rule=Host:demo.soulteary.com,demo.soulteary.io"
      - "traefik.frontend.entryPoints=https,http"
      - "traefik.frontend.headers.customResponseHeaders=Access-Control-Allow-Origin:*"
    networks:
      - traefik
    expose:
      - 80
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./mime.types:/etc/nginx/mime.types
      - ./public:/app/public
      - ./autoindex:/app/.autoindex
    extra_hosts:
      - "demo.soulteary.com:127.0.0.1"
      - "demo.soulteary.io:127.0.0.1"

networks:
  traefik:
    external: true
```

当然，既然提到服务，最低要求是能够自动负载均衡、并且提供多节点存活，比如下面这样。

```yaml
docker-compose up --scale nginx=2
```

## 最后

可能你会觉得这么一顿折腾，相比 Nginx 默认配置性能会有很大降低，然而事实是并没有，有兴趣的同学可以进行性能压测。

- 源码地址:  [https://github.com/soulteary/autoindex](https://github.com/soulteary/autoindex)

先写到这里。

—EOF
