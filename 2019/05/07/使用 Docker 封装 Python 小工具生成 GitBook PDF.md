# 使用 Docker 封装 Python 小工具生成 GitBook PDF

众所周知 GitBook 新版本生成的 PDF 是调用 `calibre` 的 `ebook-convert` 模块进行电子书生成的，而它默认生成的 PDF 尺寸比较大，而且不支持压缩，非常不利于传播。

经过简单的寻找，我看到 fuergaosi233 同学用 Python 基于 weastprint 编写了一个简单的 GitBook PDF 生成工具，使用下来感觉还不错，于是就封装了这个容器镜像，希望能够帮助到有同样需求的你。

本文将聊聊如何封装简单的 Python 应用为 Docker 工具镜像，并使用它生成 PDF 文件，操作时间在十分钟内。

完整的项目代码，我已经上传到：[https://github.com/soulteary/docker-gitbook-pdf-generator](https://github.com/soulteary/docker-gitbook-pdf-generator)，有定制需求的同学可以自取。

## 前置准备

在开始使用之前，你需要准备两个东西。

- Docker
- 你喜欢的字体文件（如果需要传播生成的电子书，注意版权风险哦）
	- 比如：苹方、思源、…

安装好容器环境，准备好字体之后，我们就可以进行容器封装了，如果你不关注封装细节，只是想使用，可以自行跳转“使用方法”小节。

## 封装容器

因为我们使用的电子书生成工具是由 Python 编写，为了更快的封装（不折腾 pip 这些基础工具），所以我使用了相对小巧的 `python:3.7-alpine3.9` 基础镜像，封装命令很简单，只需要十行左右。

```bash
FROM python:3.7-alpine3.9
LABEL  MAINTAINER="soulteary <soulteary@gmail.com>"

ENV LIBRARY_PATH /lib:/usr/lib

RUN wget https://github.com/soulteary/gitbook2pdf/archive/master.zip -O /tmp/app.zip && \
    cd /tmp && unzip app.zip && mv /tmp/gitbook2pdf-master /app

RUN apk add build-base python3-dev gcc musl-dev jpeg-dev zlib-dev libffi-dev cairo-dev pango-dev gdk-pixbuf-dev libxslt-dev && \
    cd /app && pip install -r /app/requirements.txt && \
    apk del build-base && rm -rf /var/cache/apk/*

VOLUME [ "/app/output" ]
VOLUME [ "/usr/share/fonts/" ]

WORKDIR /app

ENTRYPOINT [ "python", "/app/gitbook.py" ]
```

从上面可以看出，封装逻辑也十分简单：

- 下载代码（为了防后续有break change，我fork了原作者的仓库）
- 安装编译依赖、项目执行依赖后，下载项目依赖包，并执行编译，然后清理掉不再使用的编译依赖
- 声明可以挂载的文件位置，切换工作目录，声明容器入口点（默认执行命令）

如果我们在服务端构建，因为多数服务器具备良好的网络条件，能够快速的得到结果。但如果我们选择在本地构建，网络条件没有那么好的时候，我们访问 `alpine`、`python pip` 软件源速度不佳，构建镜像的速度将极其缓慢。

这个时候，可以使用 Mirror 来对构建进行加速，上面的构建命令可以改为下面这样：

```bash
FROM python:3.7-alpine3.9
LABEL  MAINTAINER="soulteary <soulteary@gmail.com>"

ENV LIBRARY_PATH /lib:/usr/lib

RUN wget https://github.com/soulteary/gitbook2pdf/archive/master.zip -O /tmp/app.zip && \
    cd /tmp && unzip app.zip && mv /tmp/gitbook2pdf-master /app

RUN cat /etc/apk/repositories | sed -e "s/dl-cdn.alpinelinux.org/mirrors.aliyun.com/" | tee /etc/apk/repositories && \
    apk add build-base python3-dev gcc musl-dev jpeg-dev zlib-dev libffi-dev cairo-dev pango-dev gdk-pixbuf-dev libxslt-dev && \
    cd /app && pip install -i https://mirrors.aliyun.com/pypi/simple/ -r /app/requirements.txt && \
    apk del build-base && rm -rf /var/cache/apk/*

VOLUME [ "/app/output" ]
VOLUME [ "/usr/share/fonts/" ]

WORKDIR /app

ENTRYPOINT [ "python", "/app/gitbook.py" ]
```

当然，你也可以根据自己的实际状况，将上面的阿里云的软件源替换为清华源、或者自己的源，获取更快的构建体验。

将上面的内容保存为 **Dockerfile**，然后执行 `docker build -t gitbook .`，喝口水、刷刷网站，不一会这个工具镜像就构建完成啦。

接下来，我们来聊聊使用。

## 使用方法

我们在当前目录创建一个名为 `fonts` 的文件夹，然后把早已准备好的字体内容放进去，如果不这样做的话，我们生成的电子书将会因为字体缺失而展示一堆“口口口”。

接着你可以选择使用我们上文自己构建好的镜像，或者我为你准备好的镜像开始电子书的生成操作了。

比如我们要将 `http://self-publishing.ebookchain.org` 的网页内容转换为电子书，只需要执行下面的命令：

```bash
docker run --rm -v `pwd`/fonts:/usr/share/fonts \
                -v `pwd`/output:/app/output \
                soulteary/docker-gitbook-pdf-generator "http://self-publishing.ebookchain.org"
```

如果你在上一步自己构建了容器镜像，命令中的 `soulteary/docker-gitbook-pdf-generator` 可以替换为 `gitbook` 。

稍等片刻，你将会看到日志提示：

```bash
crawl : all done!
Generating pdf,please wait patiently
Generated
```

与此同时，你当前目录会自动多出一个名为 output 的新目录，而我们想生成的电子书已经安静的躺在里面啦。

如果你觉得上面这条命令太过复杂，更喜欢使用 `docker-compose` 来简化操作，可以使用下面的配置：

```yaml
version: '2'

services:

  pdf-generator:
    image: soulteary/docker-gitbook-pdf-generator
    volumes:
      - ./output:/app/output:rw
      - ./fonts/:/usr/share/fonts:ro
    # 下面的URL替换为你想生成电子书的地址即可
    command: "http://self-publishing.ebookchain.org"
```

将上面的内容保存为 `docker-compose.yml`，然后执行 `docker-compose up` 等待电子书生成完毕即可。

## 其他

如果你对生成电子书的样式有额外定制需求，可以使用文件挂载的方式修改 `/app/gitbook.css` 样式文件。

感谢 fuergaosi233 同学的开源项目，他的项目还有几个 todo 没有完成，如果你感兴趣，可以给他提 PR ，让工具变的更好用。

先写到这里啦。

—EOF