# 使用 Docker 和 Nginx 打造高性能的二维码服务

本文将演示如何使用 `Docker` 完整打造一个基于  `Nginx` 的高性能二维码服务，以及对整个服务镜像进行优化的方法。如果你的网络状况良好，完整操作和体验时间应不超过 `15` 分钟。

## 动手前的脑洞

最近有一个小需求，需要在页面中快速生成一些二维码。

说到生成二维码，方法很多，比如按照 `QRCode` 算法进行计算之后：

- 使用各种服务端语言，然后调用 `GD` 绘图库在语言中的 `API` 进行绘制，并生成图片，然后配合能够提供 `HTTP` 服务的软件对用户提供图片访问地址。
- 使用服务端语言，然后使用 `CSS` 和 `HTML` 生成可以识别的页面图案，然后配合能够提供 `HTTP` 服务的软件对用户提供图片访问地址。
- 使用客户端脚本，使用 `Canvas` 生成二维码图片，或者和上一个方案一样，生成  `DOM` 图案。
- …

但是只是为了一个功能，就去配置一套完整的语言环境，引入一堆三方依赖，总有一种杀鸡用牛刀的感觉，并且在资源利用效率上来说，也不是最优解。

而使用客户端进行生成，现在虽然不存在太多的兼容问题，但是需要额外引入脚本资源，图片生成效率也相对较慢。

那么有没有什么环保高效的方案呢？

自然是有的，还是选择服务端生成，但是扔掉语言运行时，直接使用 `Nginx` 提供服务。

## 使用 Nginx 进行二维码生成

这里可以使用一个现成的开源模块 [ngx\_http\_qrcode\_module](https://github.com/dcshi/ngx_http_qrcode_module) 。

它通过将用户请求参数进行转换，并调用使用 `C` 实现的二维码快速生成库 [libqrencode](https://github.com/fukuchi/libqrencode) 的 `QRcode_encodeString` 实现二维码快速生成，在未开启缓存的情况下，测试平均生成图片在 `10ms` 左右。

为了方便大家理解全部的安装配置过程，我先提供一个“啰嗦”版本的 `Dockerfile`：

```bash
FROM ubuntu:18.04

RUN cat /etc/apt/sources.list | sed -e "s/archive\.ubuntu\.com/mirrors\.aliyun\.com/" | sed -e "s/security\.ubuntu\.com/mirrors\.aliyun\.com/" | tee /etc/apt/sources.list
RUN apt update && \
    apt install -y unzip wget

WORKDIR /data

# https://github.com/fukuchi/libqrencode
RUN apt install -y autoconf automake autotools-dev libtool pkg-config libpng-dev && \
    cd /data && wget https://github.com/fukuchi/libqrencode/archive/master.zip && unzip master.zip && rm -rf master.zip && \
    cd libqrencode-master && ./autogen.sh && ./configure && make && make install && ldconfig && \
    cd .. && rm -rf libqrencode-master

RUN apt install -y libgd-dev

ADD ngx_http_qrcode /data/ngx_http_qrcode
ADD nginx-1.15.5.tar.gz /data
ADD nginx.conf /data

RUN apt install -y libpcre3 libpcre3-dev && \
    cd nginx-1.15.5 && ./configure --add-module=../ngx_http_qrcode/ && \
    make && make install && mv /data/nginx.conf /usr/local/nginx/conf/nginx.conf && \
    cd .. && rm -rf ngx_http_qrcode
```

将上面的文件保存完毕。接下来我们来配置 `Nginx`。

```bash
worker_processes  1;

events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;

    keepalive_timeout  65;

    server {
        listen       80;
        server_name  localhost;

		location / {

		    set $fg_color 000000;
		    set $bg_color FFFFFF;
		    set $level 0;
		    set $hint 2;
		    set $size 300;
		    set $margin 80;
		    set $version 2;
		    set $case 0;
		    set $txt "https://soulteary.com";

		    if ( $arg_fg_color ){
                set $fg_color $arg_fg_color;
		    }
		    if ( $arg_bg_color ){
                set $bg_color $arg_bg_color;
		    }
		    if ( $arg_level ){
                set $level $arg_level;
		    }
		    if ( $arg_hint ){
                set $hint $arg_hint;
            }
            if ( $arg_size ){
                set $size $arg_size;
            }
            if ( $arg_margin ){
                set $margin $arg_margin;
            }
            if ( $arg_ver ){
                set $version $arg_ver;
            }
            if ( $arg_case ){
                set $case $arg_case;
            }
            if ( $arg_txt ){
                set $txt $arg_txt;
            }


			qrcode_fg_color $fg_color;
			qrcode_bg_color $bg_color;

			qrcode_level    $level;
			qrcode_hint     $hint;
			qrcode_size     $size;
			qrcode_margin   $margin;
			qrcode_version  $version;
			qrcode_casesensitive $case;
			qrcode_urlencode_txt $txt;

			qrcode_gen;
		}

    }
}

```

将上面的配置保存为 `nginx.conf`，然后使用下面的命令进行镜像构建。

```bash
docker build -t docker.lab.com/qrcode.lab.com .
```

如果你的网络通畅，5分钟之内，这个镜像就构建完毕了。接下来，我们对它进行一下可用性验证。

将下面的配置文件保存为 `docker-compose.yml`，然后使用 `docker-compose up` 命令启动，一个支持 `HTTP/HTTPS`，域名为 `qrcode.lab.com` 的网站就准备就绪了。

这里我使用了 `Traefik` 进行服务发现，感兴趣的童鞋可以参考我以前写的文章：[ 使用服务发现改善开发体验 ](https://soulteary.com/2018/06/11/use-server-side-discovery-improve-development.html)、[ 更完善的 Docker + Traefik 使用方案 ](https://soulteary.com/2018/08/28/better-use-of-docker-and-traefik.html)、[使用 Traefik 的一些补充细节](https://soulteary.com/2018/09/07/some-additional-details-using-traefik.html)），一旦你开始使用并掌握了它，你会发现搭建高可扩展的 `Web` 服务变的更简单了。

```yaml
version: '3'

services:

  qrcode:
    image: docker.lab.com/qrcode.lab.com:0.0.2
    expose:
      - 80
    networks:
      - traefik
    labels:
      - "traefik.enable=true"
      - "traefik.port=80"
      - "traefik.frontend.rule=Host:qrcode.lab.com"
      - "traefik.frontend.entryPoints=http,https"

networks:
  traefik:
    external: true
```

然后我们在浏览器中分别访问，来验证二维码服务是否就绪：

- `https://qrcode.lab.com`
- `https://qrcode.lab.com/?size=150&margin=20&txt=https%3A%2F%2Fsoulteary.com`

![QR Code 预览页面](https://attachment.soulteary.com/2018/10/19/qrcode-page.png)

看来服务是正常运行的，本文的基础需求到这里就解决了，并且，为了这个服务能够更好的被使用，我们可以在书签中输入下面的脚本代码：

```js
javascript:(function(){document.location.href='https://qrcode.lab.com/?size=150&txt='+encodeURIComponent(document.location.href);})()
```

当你点击书签的时候，会将当前页面自动转换为一个可以扫描的二维码。

## 通过整合语句优化容器镜像

虽然上面的内容已经满足了我们的基础需求，但是作为一个有追求的开发者，我们不光是要追求执行效率，还要追求储存效率。

虽然 `Nginx` 的运行资源占用不多。

```plain
top - 09:50:29 up 21 days, 19 min,  0 users,  load average: 0.03, 0.05, 0.05
Tasks:   4 total,   1 running,   3 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.3 us,  0.3 sy,  0.0 ni, 99.4 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  6101684 total,   332268 free,  3649484 used,  2119932 buff/cache
KiB Swap:   998396 total,   936632 free,    61764 used.  2122020 avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND                                                                                                             
    8 nobody    20   0   72352   4792   3124 S   0.7  0.1   0:00.02 nginx                                                                                                               
    1 root      20   0   70800   4996   4240 S   0.0  0.1   0:00.01 nginx 
```

但是使用 `docker images` 命令查看镜像详情，我们可以看到这个镜像还是挺大的，有 `400+MB`。

```plain
REPOSITORY                            TAG                   IMAGE ID            CREATED              SIZE
docker.lab.com/qrcode.lab.com         0.0.1                 d98376b43ae9        About a minute ago   454MB
```

这里我们修改一下上面的镜像 `Dockerfile`，尝试重新进行镜像构建。

```plain
FROM ubuntu:18.04

RUN cat /etc/apt/sources.list | sed -e "s/archive\.ubuntu\.com/mirrors\.aliyun\.com/" | sed -e "s/security\.ubuntu\.com/mirrors\.aliyun\.com/" | tee /etc/apt/sources.list

WORKDIR /tmp

RUN apt update && apt install -y unzip wget autoconf automake autotools-dev libtool pkg-config libpng-dev libgd-dev libpcre3 libpcre3-dev && \
    wget https://nginx.org/download/nginx-1.15.5.tar.gz && tar -zxvf nginx-1.15.5.tar.gz && rm -rf nginx-1.15.5.tar.gz && \
    wget https://github.com/dcshi/ngx_http_qrcode_module/archive/master.zip && unzip master.zip && mv ngx_http_qrcode_module-master ngx_http_qrcode_module && rm -rf master.zip && \
    wget https://github.com/fukuchi/libqrencode/archive/master.zip && unzip master.zip && mv libqrencode-master libqrencode && \
    cd libqrencode && ./autogen.sh && ./configure && make && make install && ldconfig && \
    cd /tmp/nginx-1.15.5 && ./configure --add-module=../ngx_http_qrcode_module/ && make && make install && \
    apt remove -y unzip wget autoconf automake autotools-dev libtool pkg-config && \
    rm -rf /tmp/* && rm -rf /var/cache/

ADD nginx.conf /usr/local/nginx/conf/nginx.conf

EXPOSE 80

ENTRYPOINT [ "/usr/local/nginx/sbin/nginx", "-g", "daemon off;" ]
```

再次构建完毕，我们会发现镜像只是减少了 `35MB`，相比较 `400MB` 多的整体体积，优化部分杯水车薪。

```plain
REPOSITORY                            TAG                   IMAGE ID            CREATED             SIZE
docker.lab.com/qrcode.lab.com         0.0.1                 a24ffc73121a        1 minutes ago      420MB
```

那么优化就到此为止了么？显然不是。

## 通过优化基础镜像来优化容器镜像

这里我们选择使用体积更小的 `Linux` 镜像，`Alpine`来进行同样功能的二维码服务的容器镜像。

因为 `Alpine` 和 `Ubuntu` 不是一个社区进行维护，所以软件包很多名称是不同的，这里我直接提供我已经查找修改完毕的镜像文件。

如果你也有类似的需求，需要将不同系统的软件进行迁移安装，可以在 `https://pkgs.alpinelinux.org/packages` 查找你所需要的软件包的名称。

```bash
FROM alpine:3.8

RUN cat /etc/apk/repositories | sed -e "s/dl-cdn.alpinelinux.org/mirrors.aliyun.com/" | tee /etc/apk/repositories && \
    apk --update add openssl-dev pcre-dev zlib-dev wget build-base autoconf automake libtool libpng-dev libgd pcre pcre-dev pkgconfig gd-dev && \
    cd /tmp && \
    wget https://nginx.org/download/nginx-1.15.5.tar.gz && tar -zxvf nginx-1.15.5.tar.gz && rm -rf nginx-1.15.5.tar.gz && \
    wget https://github.com/dcshi/ngx_http_qrcode_module/archive/master.zip && unzip master.zip && mv ngx_http_qrcode_module-master ngx_http_qrcode_module && rm -rf master.zip && \
    wget https://github.com/fukuchi/libqrencode/archive/master.zip && unzip master.zip && mv libqrencode-master libqrencode && \
    cd libqrencode && ./autogen.sh && ./configure && make && make install && ldconfig || true && \
    cd /tmp/nginx-1.15.5 && ./configure --add-module=../ngx_http_qrcode_module/ && make && make install && \
    apk del build-base autoconf automake pkgconfig && \
    rm -rf /tmp/* && rm -rf /var/cache/apk/*

ADD nginx.conf /usr/local/nginx/conf/nginx.conf

EXPOSE 80

ENTRYPOINT [ "/usr/local/nginx/sbin/nginx", "-g", "daemon off;" ]

```

当镜像打包完毕，我们再次查看镜像体积，可以看到体积有了明显的优化效果。

```plain
REPOSITORY                            TAG                   IMAGE ID            CREATED             SIZE
docker.lab.com/qrcode.lab.com         0.0.2                 d236b96c8950        1 minutes ago   79.1MB
```

## 最后

还记得本文标题中的关键词“高性能”嘛，虽说我个人测试单实例的响应时间都在 `10ms` 左右，但是如果你真的考虑使用它做对外服务的话，可以使用下面的命令，根据自己情况对节点进行动态扩容，成倍提高服务响应能力。

```bash
docker-compose scale qrcode=4
```

或者使用

```bash
docker-compose up --scale qrcode=2 -d
```

如果你也是 `Traefik` 用户，你将会看到你的实例被成功进行挂载以及流量负载均衡。

![Traefik 控制台](https://attachment.soulteary.com/2018/10/19/traefik-preview.png)

另外，为了避免被恶意利用，还需要考虑使用 `Nginx` / `iptable` 的 `req_limit` 等模块限制访问频率，以及适当修改 `ngx_http_qrcode_module` 生成内容和图片尺寸的判断。

—EOF