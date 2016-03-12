# Hugo 跨版本升级

使用 Hugo 一年半了，终于有了升级的动力。趁着最近事情不多，着手搞定了这个事情，记录下来分享给需要的同学。

## 背景

在这一年半里，我一直使用着的是老版本：`v0.20.7` ，运行非常稳定，写完文章 `Git Push` 后，GitLab Runner 自动更新预览地址，浏览没问题就可以一键发布了。

但是下面两个问题让我有了升级的想法。

### 构建速度随着内容增多变慢

去年十月，在[网站架构简化](https://soulteary.com/2018/10/14/site-optimization-record.html)之后，我的完整发布编译时间从 **1分钟** 进入了 **40s** 的阶段，但是随着内容的膨胀、编译时间越来越慢了，可以看到不少发布时间变长。

![类似这种编译时间超过45s不在少数](https://attachment.soulteary.com/2019/02/01/more-and-more-slow.png)

看着 Hugo 升级 Golang 版本，**重构优化速度变的越来越快，不免心动。**

### 需要维护两套不同版本的站点

一月初将博客主题重构之后，分享给了公司技术团队博客使用，考虑可维护性，新站点使用的是最新版本 `0.53` ，因为技术团队站点功能更简单，所以升级过程中不兼容的部分其实并不多。

但是这个站点，因为自定义了“年月日”格式的归档，以及使用的是老版本的模板查找逻辑，生成页面链接也不完全兼容，所以直接升级是不行的。

可以预期的是，随着使用时间越来越长，这两个站点的差异会越来越大，**为了可维护性，必须将这两个站点使用的 Hugo 版本统一**。

## 梳理主要问题

1. 官方支持 RSS 文件直接输出，是否还需要自定义站点 RSS 文件？
2. 官方直接提供压缩能力，是否足够替换 Pipeline 中定制的压缩服务？
3. 页面模板查找逻辑、模板语法、站点配置文件变更，现有模板无法直接使用。
4. 分类标签系统扁平化，不再支持树形层级嵌套，链接兼容如何处理？

下面我来逐个击破。

## Hugo RSS 解决方案

官方支持了 RSS 格式的输出，只要在 `layouts` 根目录创建一个文件即可，`index.rss.xml`，模板可以自定义，参考官方文档 。

另外官方生成文档，默认会输出正确的 XML Version，所以可以检查并删除己配置的文档模板中下面的内容。

```xml
<?xml version="1.0" encoding="utf-8" standalone="yes" ?>
```

但是这样会有两个额外的问题，第一个是额外生成了许多不需要生成的“订阅源”，像是下面这样。

```bash
find public/**/*.xml

public/tags/标签A/index.xml
public/tags/标签A/index.xml
public/tags/标签C/index.xml
public/topics/code/index.xml
public/topics/funny/index.xml
public/topics/index.xml
public/topics/life/index.xml
public/topics/share/index.xml
public/topics/website/index.xml
```

公司的技术团队博客可以保留这个功能，但是我个人一来更新频率没有那么高，二来我希望订阅源唯一可控，所以这些多余的内容我是要干掉的。

第二个问题是官方 RSS 输出内容不支持自定义路径，你的订阅地址就只能是下面这样：

```plain
网站地址/index.xml
```

使用老版本的 RSS 方案，创建一个 `/feed` ，然后放置自定义的 RSS 模板，你会发现生成内容，仅支持该目录之下的文章… ORZ

如果你有类似的需求，这里更好的方案是“禁用官方RSS生成能力”、“自定义RSS模板”，可以做到按照你的需求在你期望的路径生成你期望数量的 RSS 内容。

首先是禁用官方RSS生成能力，在站点 `config.toml` 配置文件中添加下面的内容：

```Toml
disableKinds= ["RSS"]
```

如果你有定义 `output` 格式，并包含 RSS 定义，也需要删除该内容。

```diff
 [outputs]
-page = [ "HTML", "RSS" ]
+page = [ "HTML" ]
```

接着分别创建 `layouts/feed/index.html` 和 `content/feed/index.md` 两个文件。

模板文件内容可以参考：

```xml
{{ `<?xml version="1.0" encoding="utf-8" standalone="yes" ?>` | safeHTML }}
<rss version="2.0" xmlns:content="http://purl.org/rss/1.0/modules/content/" xmlns:wfw="http://wellformedweb.org/CommentAPI/" xmlns:dc="http://purl.org/dc/elements/1.1/" xmlns:atom="http://www.w3.org/2005/Atom" xmlns:sy="http://purl.org/rss/1.0/modules/syndication/" xmlns:slash="http://purl.org/rss/1.0/modules/slash/">
    <channel>
        <title>{{ if eq .Title .Site.Title }}{{ .Site.Title }}{{ else }}{{ with .Title }}{{.}} - {{ end }}{{ .Site.Title }}{{ end }}</title>
        <link>{{ .Site.BaseURL }}feed/</link>
        <description>{{ .Site.Title }}最近更新内容。</description>{{ with .Site.LanguageCode }}
        <language>{{.}}</language>{{end}}{{ with .Site.Author.email }}
        <managingEditor>{{.}}{{ with $.Site.Author.name }} ({{.}}){{end}}</managingEditor>{{end}}{{ with .Site.Author.email }}
        <webMaster>{{.}}{{ with $.Site.Author.name }} ({{.}}){{end}}</webMaster>{{end}}{{ with .Site.Copyright }}
        <copyright>{{.}}</copyright>{{end}}{{ if not .Date.IsZero }}
        <lastBuildDate>{{ .Date.Format "Mon, 02 Jan 2006 15:04:05 -0700" | safeHTML }}</lastBuildDate>{{ end }}
        <sy:updatePeriod>hourly</sy:updatePeriod>
        <sy:updateFrequency>1</sy:updateFrequency>
        <generator>hugo</generator>
        <atom:link href="{{ .Site.BaseURL }}feed/" rel="self" type="application/rss+xml"/>
        {{ range first 10 (where .Site.Pages "Type" "post") }}
        <item>
            <title>{{ .Title }}</title>
            <link>{{ .Permalink }}</link>
            <pubDate>{{ .Date.Format "Mon, 02 Jan 2006 15:04:05 -0700" | safeHTML }}</pubDate>
            <dc:creator>{{`<![CDATA[`|safeHTML}}苏洋(soulteary){{`]]>`|safeHTML}}</dc:creator>
            <author>{{`<![CDATA[`|safeHTML}}苏洋(soulteary){{`]]>`|safeHTML}}</author>
            <guid>{{ .Permalink }}</guid>
            <description>{{`<![CDATA[`|safeHTML}}{{ .Description | safeHTML }}{{`]]>`|safeHTML}}</description>
            <content:encoded>{{`<![CDATA[`|safeHTML}}{{ .Description | safeHTML }}{{`]]>`|safeHTML}}</content:encoded>
        </item>{{ end }}
    </channel>
</rss>
```

页面数据文件，示例文件：

```plain
---
title: "Rss Feed"
author: "soulteary"
date: "2019-01-24"
type: feed
draft: false
isCJKLanguage: true
outputs: [ "HTML" ]
---
```

再次执行 `hugo` 进行站点生成，会发现生成的页面内容已经大幅减少：

```plain
自定义之前：
                   |  EN   
+------------------+------+
  Pages            | 4419  
  Paginator pages  |  594  
  Non-page files   |    0  
  Static files     |  197  
  Processed images |    0  
  Aliases          | 1796  
  Sitemaps         |    1  
  Cleaned          |    0  

自定义之后：
                   |  EN   
+------------------+------+
  Pages            | 3285  
  Paginator pages  |  594  
  Non-page files   |    0  
  Static files     |  197  
  Processed images |    0  
  Aliases          | 1796  
  Sitemaps         |    1  
  Cleaned          |    0  
```

最后别忘记通过构建脚本，将生成文件进行重命名。

```bash
mv feed/index.html feed/index.xml
```

## 更好的Hugo页面压缩能力

在使用 Hugo 版本和之前的压缩模式进行对比，发现 Hugo 压缩确实效率高不少，添加压缩参数 `--minify` 执行 Hugo ，生成时间几乎没有变化，还能省下 GitLab Pipeline 调用 Job 过程中的时间损耗，真的是太赞了。

但是压缩结果完全使用了 HTML 宽松模式，所有的 HTML Tag 属性都失去了引号，像是下面这样。

```html
<span class=crayon-o>/</span>
```

个人倾向使用相对严格的模式进行页面结构编写，即使是程序生成的代码也是如此，因为一旦要进行调试，相对严格的标准，可以消除很多歧义，减少不必要的调试时间。

由于老版本不支持 `--minify` 参数，所以我使用 `Node.js` 简单写了一个脚本，用于替换页面内的空白内容，程序处理内容比较多，贴一下主要替换逻辑供参考。

```js
const trimTags = (s) => s.replace(/>\s+</gm, '><').replace(/>(\s+\n|\r)/g, '>');
const content = trimTags(content);
```

如果你和我一样，定制了代码高亮，有更复杂的 HTML 结构，那么还需要额外处理一下代码，避免出现 `ReDoS` 问题。

## 模板配置相关处理

首先贴出我的站点主题的 `layouts` 目录结构（部分），它也代表了网站的整个抽象逻辑。

```plain
layouts
├── 404.html
├── _default
│   ├── baseof.html
│   ├── list.html
│   ├── section.html
│   ├── summary.html
│   ├── tag.html
│   ├── topics.html
│   └── topics.terms.html
├── about
│   └── single.html
├── about-site
│   └── single.html
├── archives
│   ├── list.html
│   └── single.html
├── booklist
│   └── single.html
├── contact
│   └── single.html
├── feed
│   └── single.html
├── index.html
├── index.redir
├── links
│   └── single.html
├── partials
│   ├── bloc
│   │   ├── content
│   │   │   ├── badges.html
│   │   │   ├── comments.html
│   │   │   ├── content.html
│   │   │   ├── navigation.html
│   │   │   ├── pagination.html
│   │   │   ├── sidebar.html
│   │   │   └── summary.html
│   ├── loop.html
│   ├── modules
│   │   ├── footer
│   │   │   ├── index.html
│   │   │   └── script.html
│   │   ├── header
│   │   │   ├── custom.html
│   │   │   ├── index.html
│   │   │   ├── link-style.html
│   │   │   ├── meta-robots.html
│   │   │   ├── meta.html
│   │   │   └── script.html
│   │   ├── meta
│   │   │   ├── page-home-meta.html
│   │   │   ├── page-topic-meta.html
│   │   │   ├── post-meta-bottom.html
│   │   │   └── post-meta-top.html
│   │   ├── sidebar
│   │   │   └── index.html
│   │   └── site
│   │       │   ├── social
│   │       │   │   └── rss.html
│   │       │   └── social.html
│   │       ├── pagination.html
│   │       └── topline.html
│   ├── pages
│   │   ├── about
│   │   │   └── index.html
│   │   ├── about-site
│   │   │   └── index.html
│   │   ├── archives
│   │   │   ├── all.html
│   │   │   ├── index.html
│   │   │   ├── month.html
│   │   │   └── year.html
│   │   ├── booklist
│   │   │   └── index.html
│   │   ├── contact
│   │   │   └── index.html
│   │   ├── error-page
│   │   │   └── index.html
│   │   ├── home
│   │   │   ├── index.html
│   │   │   ├── list.html
│   │   │   └── summary.html
│   │   ├── links
│   │   │   └── index.html
│   │   ├── post
│   │   │   └── index.html
│   │   ├── subject
│   │   │   ├── index.html
│   │   │   └── item.html
│   │   ├── tag
│   │   │   ├── index.html
│   │   │   ├── list.html
│   │   │   └── summary.html
│   │   └── topic
│   │       ├── index.html
│   │       ├── list.html
│   │       └── summary.html
│   └── static
│       ├── error-track.js.html
│       ├── global-config.js.html
│       ├── init.js.html
│       ├── page-debug.js.html
│       └── web-pref.js.html
├── post
│   ├── single.html
│   ├── single.md
├── robots.txt
├── shortcodes
│   └── crayonCode.html
├── sitemap.xml
└── subject
    └── single.html
```

现在的查找逻辑不是十分合理，为了避免构建时的警告信息，我使用 `layouts/_default` 接管了标签和分类的模板入口，其余的入口页面依旧放在 `layouts` 子目录的各同名目录下，比如 `layouts/post/single.html` 。

所有页面的真实处理逻辑放置在 `layouts/partials/pages` 中，可以最大限度保障页面主题的兼容性，比如这次，我就只是修改了入口页面的位置，而页面处理逻辑没有大动。

额外说明一点，`_default` 目录下的文件，只有 `tag` 和 `topics` 相关三个文件存在内容，其余文件保持为空即可。

新版本 Hugo 针对 config.toml  也有升级策略，直接执行 `hugo` ，配置文件中的问题，它会进行报错提示，并示例你如何更改，比如这样：

```diff
@@ -61,30 +61,13 @@ 
 [mediaTypes]
 
 [mediaTypes."text/plain"]
-suffix = "md"
-
-[mediaTypes."application/rss"]
-suffix = "xml"
+suffixes = ["md"]
 
 [outputFormats.MD]
 Path = "/"
 mediaType = "text/plain"
 isPlainText = true
 
-[outputFormats.FEED]
-mediatype = "application/rss"
-baseName = "feed"
-
 [outputs]
-page = [ "HTML" ,"MD", "FEED" ]
+page = [ "HTML" ,"MD" ]
```

## 分类和标签扁平化以及其他兼容处理

在 Hugo 升级之前，我使用的是这样的分类结构：

```plain
topics: [ "知识点滴/容器化" ]
```

老版本的 Hugo 会自动生成两级分类目录，并且两个目录都支持索引，像是下面这样。

```plain
/知识点滴
/知识点滴/index.html
/知识点滴/page/2.html
/知识点滴/容器化
/知识点滴/容器化/index.html
/知识点滴/容器化/page/2.html
```

而新版本会生成唯一的分类，并且使用自己的策略**转义链接地址中的空格和斜杠为连字符**。

```plain
/知识点滴-容器化
/知识点滴-容器化/index.html
/知识点滴-容器化/page/2.html
```

在思考之后，我发现除了做接口需要表明资源从属关系之外，除非写书似乎文章还真的不太需要那么明确的层级隔离，一级目录外加标签就能提供良好的阅读和解决简单检索需求。

所以我进行了简化，去掉了所有的文章分类从属关系。但是我还有一个按照年月日进行日期归档的路径，这里还是希望能够保持从属关系的，该怎么解决呢？

首先归档内容，暂时还是需要自己手动生成并维护的，下面是我使用另外一个脚本在每次文章发布时生成的目录结构（部分）。

```plain
./content/archives
├── 2018
│   ├── 11
│   │   └── index.md
│   ├── 12
│   │   └── index.md
│   └── _index.md
├── 2019
│   ├── 01
│   │   └── index.md
│   └── _index.md
└── _index.md
```

这里使用了一个取巧的方案，使用 `_index.md` 会枚举目录中所有内容的特性，可以轻松生成下面的结构。

```plain
./public/archives
├── 2018
│   ├── 12
│   │   └── index.html
│   └── index.html
├── 2018.html
├── 2019
│   ├── 01
│   │   └── index.html
│   └── index.html
├── 2019.html
└── index.html
```

接下来只需要修正原本模板中引用地址 `/archives/YEAR/` 为 `/archives/YEAR.html` 即可

上文提到过 Hugo 新版本对于链接的一些额外转义处理，除了分类会被影响外，标签也被波及到了。

举个例子，我原本有一个标签叫做 ： `Linux/Mac` ，在旧版本的 Hugo 中的输出结果是这样：

```plain
/public/tags/linux/mac/index.html
```

但是在新版本变成了这样：

```plain
/public/tags/linux-mac/index.html
```

因为我禁用了 RSS ，暂时不提供标签的订阅，文章内直接引用标签目前也比较少，访问地址变了就变了，但是模板中如果直接使用老版本的语法，标签地址生成的还是老样子（生成链接策略和渲染逻辑不一致），结果就是含有空格和斜杠的标签页面是无法正常浏览的！

举个例子，老版本语法：

```plain
{{ $tagLink | urlize }}
```

解决方式比较 trick，需要手动在模板中进行转义，并补全 `.html` 后缀：

```plain
{ replace (replace (lower $tag) "/" "-") " " "-"}}.html"
```

至此，升级过程中的主要问题就都讲完了，我们接下来聊聊性能提升和其他的话题。

## 性能和一些其他的事情

这次重构之后，完整发布时间缩短了 `10s` ，构建时间减少了 `3s` ，后续计划将预览和生产的流水线进行分割，应该还能进一步提升效率。

![完整发布时间35s左右](https://attachment.soulteary.com/2019/02/01/finial.png)

在做这次升级重构之前，我首先考虑了最低成本升级到的完全兼容的可用版本。

> v0.20.7
> @bep bep released this on 3 May 2017 · 1666 commits to master since this release

在试验之后，我发现唯一和我当前版本兼容的是 `v0.21`，也是一个古董版本，没有实质的变化。

> v0.47.1
> Hugo Static Site Generator v0.47.1 darwin/amd64 BuildDate: 2018-08-20T08:16:52Z

稍微做一些重构，能够接触到的最新版本则是上面的 0.47.1 ，也还是低版本的 Golang，并且和主流版本差异还是不小。

我使用 `hugo benchmark` 比较了 `0.20.x` 和 `0.40.x` 发现性能不升反降，期初我也很犹豫是否值得进行升级。

**v0.20.x**

```plain
Average time per operation: 3955ms
Average memory allocated per operation: 1651040kB
Average allocations per operation: 22417100
```

**v0.40.x**

```plain
Average time per operation: 4056ms
Average memory allocated per operation: 1650683kB
Average allocations per operation: 22396305
```

于是我认真浏览了这一年半以来的[版本发布记录](https://github.com/gohugoio/hugo/releases?after=v0.25.1)，觉得还是要与时俱进，如果不升级，将一直锁定在低版本的 Golang 运行时，所有的问题也都只能自己定制解决，完全不能使用社区新功能、也不把折腾内容贡献社区。

在使用两个简单站点分别使用 v0.20  和 v0.50 进行测试的时候，我发现提升还是很明显的，于是便着手进行了升级，附加调试完整花费了6个小时。

很可惜在 `v0.50.3` 版本之后，官方废弃了 `hugo benchmark` 这个命令，所以我们不能够和以往一样输出性能报告，不过直接使用站点生成时间来进行对比，也是一样的（站点实际构建时间）。

```plain
Total in 3452 ms
Total in 3547 ms
Total in 3730 ms
```

可以看到效果还是不错的，另外我也终于可以使用 `brew install hugo` 安装的 Hugo 更方便的进行本地预览了。

## 最后

下篇内容，我们继续聊聊 Wiki 系统的搭建。

—EOF
