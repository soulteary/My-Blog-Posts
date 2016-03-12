# 加速Ghost静态资源

Ghost最近势头不错，连续几个月，基本每个月都是release 2个版本，不过也正是因为如此，ghost每次发布之后，API都会有些许变动。

故，不建议在官方merge任何i18N功能之前，使用中文版本，毕竟后台没几个英文，前台主题可以依赖自己的主题模板去实现i18N，没有使用影响。

下面分享一个简单修改的细节，可以使用七牛一类的CDN服务商无痛加速网站资源。

七牛官方分享的加速方案是网友修改storage接口，将内容不存自己的服务器，直接使用CDN，个人觉得不妥，不利于迁移维护，以及临时灾备，况且把AK,SK都存下来，对于后续升级也不利。

而且CDN基本都提供反代功能，可能大家不叫这个词，业界唤作“一键加速”。

0.5.8版本的Ghost默认暂时不支持CDN，不过简单修改模板helpers可以完成我们的需求。

首先，我们需要把通用配置添加到config.js中。这里我们使用一个对象来储存cdn信息，以方便扩展和维护。

```js
// https://github.com/soulteary/Ghost/blob/master/config.example.js
var path = require('path'),
    config;

config = {
    // ### Production
    // When running Ghost in the wild, use the production environment
    // Configure your URL and mail settings here
    production: {
        url: 'http://my-ghost-blog.com',
        mail: {},
        database: {
            client: 'sqlite3',
            connection: {
                filename: path.join(__dirname, '/content/data/ghost.db')
            },
            debug: false
        },

        server: {
            // Host to be passed to node's `net.Server#listen()`
            host: '127.0.0.1',
            // Port to be passed to node's `net.Server#listen()`, for iisnode set this to `process.env.PORT`
            port: '2368'
        },
        cdn: { assets: "//your-cdn-path"}
    },
    ...
```

如果你想关闭CDN，请设置`cdn:false`，或者删除这个对象。 然后我们需要修改模板的assets helper，这个helper主要是针对模板使用assets方法引入的资源进行地址修正。 关键代码如下：

```js
// https://github.com/soulteary/Ghost/blob/master/core/server/helpers/asset.js
...
    // Get rid of any leading slash on the context
    context = context.replace(/^\//, '');
    output += context;

    // cdn assets option
    // 如果设置了CDN，那么使用CDN配置项目中的路径进行资源替换
    if (!isAdmin && config['cdn'] && config['cdn']['assets']) {
        output = utils.assetTemplate({
            source : config['cdn']['assets'] + output,
            version: config.assetHash
        });
    } else {
        if (!context.match(/^favicon\.ico$/)) {
            output = utils.assetTemplate({
                source : output,
                version: config.assetHash
            });
        }
    }
...
```

image helper会修改markdown中的image标签的路径，如果你直接有使用低版本，会插入完整的路径，需要自己修改，或者在此基础上，进一步修改，没几篇内容的话，自己手动改下吧。 关键代码如下：

```js
// https://github.com/soulteary/Ghost/blob/master/core/server/helpers/image.js
image = function (options) {
    var absolute = options && options.hash.absolute,
        imgPath = this.image;
    if (this.image) {
        // cdn assets
        // 如果设定了cdn，那么对内容添加cdn
        if (config['cdn'] && config['cdn']['assets']) {
            imgPath = config['cdn']['assets'] + this.image;
        }
        return config.urlFor('image', {image: imgPath}, absolute);
    }
};
```

最后修改url helper对遗漏的内容进行补刀，在这个helper里，你可以拿到最后的render结果。 关键代码如下：

```js
// https://github.com/soulteary/Ghost/blob/master/core/server/helpers/url.js
if (schema.isUser(this)) {
    return config.urlFor('author', {author: this}, absolute);
}
// compressing page content can improve the response speed
// 较稳妥的压缩页面输出内容，提高响应速度，markdown转换之后，效果明显。
this.body = this.body.replace(/>(\n*|\r*|\s*)</gm, '><').replace(/>(\s*|\n*|\r*)/gm, '>');
// enable cdn for post content
if (config['cdn'] && config['cdn']['assets']) {
    this.body = this.body.replace(/<img.*?src=\"(.*?)\".*?\/?>/g, function (src, uri) {
        if (uri.indexOf(config['cdn']['assets']) !== 0) {
            if (uri.indexOf(config['url']) === 0) {
                return src.replace(config['url'], config['url'].split('://')[0] + ':' + config['cdn']['assets']);
            } else {
                if (uri.indexOf('://') === -1) {
                    return src.replace(uri, config['cdn']['assets'] + uri);
                }
            }
        }
    });
}
return config.urlFor(this, absolute);
```

另外，英文摘要输出对于中文摘要来说不太友好，简单的优化如下：

```js
// https://github.com/soulteary/Ghost/blob/master/core/server/helpers/excerpt.js
// Strip inline and bottom footnotes
excerpt = excerpt.replace(/<a href="#fn.*?rel="footnote">.*?<\/a>/gi, '');
excerpt = excerpt.replace(/<div class="footnotes"><ol>.*?<\/ol><\/div>/, '');

// truncate for GBK when use character mode
// 中文内容截断，根据字符的方式
if (truncateOptions.characters) {
    // remove heading tags with content and other html tags
    excerpt = excerpt.replace(/<h\d.*?\/h\d>/gi, '');
    excerpt = excerpt.replace(/<\/?[^>]+>/gi, '');
    var arr = excerpt.split("\n"), tmp = "";
    for (var i = 0, j = arr.length; i < j; i++) {
        // remove \n && \r
        arr[i] = arr[i].replace(/(\r\n|\n|\r)+/gm, '');
        if (arr[i]) {
            if (tmp.length < truncateOptions.characters) {
                var len = tmp.length;
                if (len + arr[i].length > truncateOptions.characters) {
                    tmp += arr[i];
                    tmp = tmp.substr(0, truncateOptions.characters - 3);
                    tmp = tmp.substr(0, len) + tmp.substr(len).replace(/(，|\,|\:|\-|\"|、|“|”)/gm, '') + '...';
                    break;
                } else {
                    // less than 10 chars is unnecessary
                    if (truncateOptions.characters - len < 10 && arr[i].length<10) {
                        break;
                    } else {
                        tmp += arr[i];
                    }
                }
            }
        }
    }
    excerpt = tmp.replace(/(\r\n|\n|\r)+/gm, ' ');
} else {
    // Strip other html
    excerpt = excerpt.replace(/<\/?[^>]+>/gi, '');
    excerpt = excerpt.replace(/(\r\n|\n|\r)+/gm, ' ');
}

/*jslint regexp:false */

if (!truncateOptions.words && !truncateOptions.characters) {
    truncateOptions.words = 50;
}
```

如果你觉得修改麻烦，也可以直接到相关PR页面下载文件：https://github.com/TryGhost/Ghost/pull/4839/files

--EOF--

