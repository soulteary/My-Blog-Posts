# 源码编译 GitLab Runner

从源码安装 GitLab 你或许听说过，但是从源码安装 GitLab Runner ，或许这将是你听到的第一篇相关博客。

最近遇到一个问题，需要手动编译构建 GitLab Runner，而官方文档陈旧、命令过时，如果按照官方错误的指引搞下去，难免会浪费很多时间，而且得不到你想要的结果。

如果你也有类似需求，跟随本篇文章，大概十分钟左右就能折腾出一个属于你自己的 GitLab Runner。

## 前置准备

这次的前置准备真的不多，大概就需要两样：

- 一台虚拟机，我选择的操作系统是 `Ubuntu 18.04 64位`
- 一颗耐心

在编译 GitLab Runner 之前，我们需要制作编译工具，而编译工具依赖定制的系统环境，所以第一步就是简单折腾一个系统环境。

## 准备系统环境

安装基础环境的第一步，无非是升级系统环境、安装必要依赖。

 这里推荐安装 `git` 依赖，而非官方文档中注明的 `git-core` ，详细原因见[这篇文章](https://askubuntu.com/questions/5930/what-are-the-differences-between-the-git-and-git-core-packages)。

```bash
apt update && apt upgrade -y

apt install software-properties-common git mercurial unzip vim binfmt-support qemu-user-static -y
```

与其他软件不同的是，这里必须安装 Docker ，因为后面的构建过程中会使用到 Docker （哪怕你编译的不是 Runner 的 Docker 镜像）。

截止文章发布，使用 `Docker version 19.03.1` 并无任何异常。如果你使用的是阿里云，可以使用下面的命令进行安装，更详细的内容，可以参考[之前的文章](https://soulteary.com/2019/04/06/configure-ubuntu-18-04.html)。

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
add-apt-repository "deb [arch=amd64] https://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
apt install -y docker-ce
```

由于软件是使用 `Go` 语言编写，所以安装 `Go` 语言运行时必不可少。但可惜的是，我们无法使用最新的 `Go 1.12`，必须老老实实使用 2017 年推出的 `1.8.7`。

```bash
wget https://storage.googleapis.com/golang/go1.8.7.linux-amd64.tar.gz

sudo tar -C /usr/local -xzf go*-*.tar.gz

echo "export GOPATH=$HOME/Go" >> ~/.profile
echo "export PATH=$PATH:$GOPATH/bin:/usr/local/go/bin" >> ~/.profile
source ~/.profile
```

接下来需要做的事情是获取软件包源码、以及开发需要使用的依赖包。

```bash
go get gitlab.com/gitlab-org/gitlab-runner
cd $GOPATH/src/gitlab.com/gitlab-org/gitlab-runner/

go get -u github.com/jteeuwen/go-bindata/...
```

## 准备编译工具

在上述命令都执行完毕，且没有报错的情况下，继续执行下面的语句，就能获得可用的编译工具啦。

```bash
make helper-build helper-docker
```

如果你比较顺利，将会看到类似下面的日志：

```bash
# make helper-build helper-docker

go get github.com/mitchellh/gox
gox -osarch=windows/amd64 -ldflags "-X gitlab.com/gitlab-org/gitlab-runner/common.NAME=gitlab-runner -X gitlab.com/gitlab-org/gitlab-runner/common.VERSION=12.2.0~beta.1803.g41d5c6ad -X gitlab.com/gitlab-org/gitlab-runner/common.REVISION=41d5c6ad -X gitlab.com/gitlab-org/gitlab-runner/common.BUILT=2019-08-01T16:59:49+0000 -X gitlab.com/gitlab-org/gitlab-runner/common.BRANCH=master -s -w" -output=dockerfiles/build/binaries/gitlab-runner-helper.x86_64-windows gitlab.com/gitlab-org/gitlab-runner/apps/gitlab-runner-helper
Number of parallel builds: 7

-->   windows/amd64: gitlab.com/gitlab-org/gitlab-runner/apps/gitlab-runner-helper
gox -osarch=linux/amd64 -ldflags "-X gitlab.com/gitlab-org/gitlab-runner/common.NAME=gitlab-runner -X gitlab.com/gitlab-org/gitlab-runner/common.VERSION=12.2.0~beta.1803.g41d5c6ad -X gitlab.com/gitlab-org/gitlab-runner/common.REVISION=41d5c6ad -X gitlab.com/gitlab-org/gitlab-runner/common.BUILT=2019-08-01T16:59:49+0000 -X gitlab.com/gitlab-org/gitlab-runner/common.BRANCH=master -s -w" -output=dockerfiles/build/binaries/gitlab-runner-helper.x86_64 gitlab.com/gitlab-org/gitlab-runner/apps/gitlab-runner-helper
Number of parallel builds: 7

-->     linux/amd64: gitlab.com/gitlab-org/gitlab-runner/apps/gitlab-runner-helper
gox -osarch=linux/arm -ldflags "-X gitlab.com/gitlab-org/gitlab-runner/common.NAME=gitlab-runner -X gitlab.com/gitlab-org/gitlab-runner/common.VERSION=12.2.0~beta.1803.g41d5c6ad -X gitlab.com/gitlab-org/gitlab-runner/common.REVISION=41d5c6ad -X gitlab.com/gitlab-org/gitlab-runner/common.BUILT=2019-08-01T16:59:49+0000 -X gitlab.com/gitlab-org/gitlab-runner/common.BRANCH=master -s -w" -output=dockerfiles/build/binaries/gitlab-runner-helper.arm gitlab.com/gitlab-org/gitlab-runner/apps/gitlab-runner-helper
Number of parallel builds: 7

-->       linux/arm: gitlab.com/gitlab-org/gitlab-runner/apps/gitlab-runner-helper
docker build -t gitlab/gitlab-runner-helper:x86_64-41d5c6ad -f dockerfiles/build/Dockerfile.x86_64 dockerfiles/build
Sending build context to Docker daemon  40.32MB
Step 1/6 : FROM alpine:3.9
3.9: Pulling from library/alpine
e7c96db7181b: Pull complete
Digest: sha256:7746df395af22f04212cd25a92c1d6dbc5a06a0ca9579a229ef43008d4d1302a
Status: Downloaded newer image for alpine:3.9
 ---> 055936d39205
Step 2/6 : RUN apk add --no-cache bash ca-certificates git git-lfs miniperl 	&& ln -s miniperl /usr/bin/perl
 ---> Running in 93e1c5a12b93
fetch http://dl-cdn.alpinelinux.org/alpine/v3.9/main/x86_64/APKINDEX.tar.gz
fetch http://dl-cdn.alpinelinux.org/alpine/v3.9/community/x86_64/APKINDEX.tar.gz
(1/14) Installing ncurses-terminfo-base (6.1_p20190105-r0)
(2/14) Installing ncurses-terminfo (6.1_p20190105-r0)
(3/14) Installing ncurses-libs (6.1_p20190105-r0)
(4/14) Installing readline (7.0.003-r1)
(5/14) Installing bash (4.4.19-r1)
Executing bash-4.4.19-r1.post-install
(6/14) Installing ca-certificates (20190108-r0)
(7/14) Installing nghttp2-libs (1.35.1-r0)
(8/14) Installing libssh2 (1.8.2-r0)
(9/14) Installing libcurl (7.64.0-r2)
(10/14) Installing expat (2.2.7-r0)
(11/14) Installing pcre2 (10.32-r1)
(12/14) Installing git (2.20.1-r0)
(13/14) Installing git-lfs (2.5.1-r2)
Executing git-lfs-2.5.1-r2.post-install
Git LFS initialized.
(14/14) Installing miniperl (5.26.3-r0)
Executing busybox-1.29.3-r10.trigger
Executing ca-certificates-20190108-r0.trigger
OK: 42 MiB in 28 packages
Removing intermediate container 93e1c5a12b93
 ---> cf8c23304de7
Step 3/6 : RUN git lfs install --skip-repo
 ---> Running in cdd487902490
Git LFS initialized.
Removing intermediate container cdd487902490
 ---> 995fbdc82d24
Step 4/6 : COPY ./scripts/ /usr/bin
 ---> d1a8aabbcd3c
Step 5/6 : COPY ./binaries/gitlab-runner-helper.x86_64 /usr/bin/gitlab-runner-helper
 ---> fcb3bbe621aa
Step 6/6 : RUN echo 'hosts: files dns' >> /etc/nsswitch.conf
 ---> Running in d5a9a71bbe2f
Removing intermediate container d5a9a71bbe2f
 ---> 73b181e7c05a
Successfully built 73b181e7c05a
Successfully tagged gitlab/gitlab-runner-helper:x86_64-41d5c6ad
docker rm -f gitlab-runner-prebuilt-x86_64-41d5c6ad
Error: No such container: gitlab-runner-prebuilt-x86_64-41d5c6ad
Makefile.runner_helper.mk:55: recipe for target 'out/helper-images/prebuilt-x86_64.tar' failed
make: [out/helper-images/prebuilt-x86_64.tar] Error 1 (ignored)
docker create --name=gitlab-runner-prebuilt-x86_64-41d5c6ad gitlab/gitlab-runner-helper:x86_64-41d5c6ad /bin/sh
be14d18aa9153578b35e7c58fce754093b89d6b50228c02baeaa440c753d723a
docker export -o out/helper-images/prebuilt-x86_64.tar gitlab-runner-prebuilt-x86_64-41d5c6ad
docker rm -f gitlab-runner-prebuilt-x86_64-41d5c6ad
gitlab-runner-prebuilt-x86_64-41d5c6ad
xz -f -9 out/helper-images/prebuilt-x86_64.tar
docker build -t gitlab/gitlab-runner-helper:arm-41d5c6ad -f dockerfiles/build/Dockerfile.arm dockerfiles/build
Sending build context to Docker daemon  40.32MB
Step 1/6 : FROM multiarch/alpine:armhf-v3.9
armhf-v3.9: Pulling from multiarch/alpine
d2a76f42393b: Pull complete
0e48aa23384e: Pull complete
Digest: sha256:138eab1a3a8b31f4d0df75a752b5fdda95612505522de9e9f8d2f96d3d97abd9
Status: Downloaded newer image for multiarch/alpine:armhf-v3.9
 ---> daef464a9e14
Step 2/6 : RUN apk add --no-cache bash ca-certificates git git-lfs miniperl 	&& ln -s miniperl /usr/bin/perl
 ---> Running in a87dfb239af0
fetch https://uk.alpinelinux.org/alpine/v3.9/main/armhf/APKINDEX.tar.gz
fetch https://uk.alpinelinux.org/alpine/v3.9/community/armhf/APKINDEX.tar.gz
(1/15) Installing ncurses-terminfo-base (6.1_p20190105-r0)
(2/15) Installing ncurses-terminfo (6.1_p20190105-r0)
(3/15) Installing ncurses-libs (6.1_p20190105-r0)
(4/15) Installing readline (7.0.003-r1)
(5/15) Installing bash (4.4.19-r1)
Executing bash-4.4.19-r1.post-install
(6/15) Installing ca-certificates (20190108-r0)
(7/15) Installing nghttp2-libs (1.35.1-r0)
(8/15) Installing libssh2 (1.8.2-r0)
(9/15) Installing libcurl (7.64.0-r2)
(10/15) Installing libgcc (8.3.0-r0)
(11/15) Installing expat (2.2.7-r0)
(12/15) Installing pcre2 (10.32-r1)
(13/15) Installing git (2.20.1-r0)
(14/15) Installing git-lfs (2.5.1-r2)
Executing git-lfs-2.5.1-r2.post-install
Git LFS initialized.
(15/15) Installing miniperl (5.26.3-r0)
Executing busybox-1.29.3-r10.trigger
Executing ca-certificates-20190108-r0.trigger
OK: 40 MiB in 34 packages
Removing intermediate container a87dfb239af0
 ---> 919719cb8063
Step 3/6 : RUN git lfs install --skip-repo
 ---> Running in 292da8a82fff
Git LFS initialized.
Removing intermediate container 292da8a82fff
 ---> da40dc15d497
Step 4/6 : COPY ./scripts/ /usr/bin
 ---> 65df18912961
Step 5/6 : COPY ./binaries/gitlab-runner-helper.arm /usr/bin/gitlab-runner-helper
 ---> eb4e1e5c8f94
Step 6/6 : RUN echo 'hosts: files dns' >> /etc/nsswitch.conf
 ---> Running in 6478fa57668a
Removing intermediate container 6478fa57668a
 ---> 9dc701e1cac4
Successfully built 9dc701e1cac4
Successfully tagged gitlab/gitlab-runner-helper:arm-41d5c6ad
docker rm -f gitlab-runner-prebuilt-arm-41d5c6ad
Error: No such container: gitlab-runner-prebuilt-arm-41d5c6ad
Makefile.runner_helper.mk:55: recipe for target 'out/helper-images/prebuilt-arm.tar' failed
make: [out/helper-images/prebuilt-arm.tar] Error 1 (ignored)
docker create --name=gitlab-runner-prebuilt-arm-41d5c6ad gitlab/gitlab-runner-helper:arm-41d5c6ad /bin/sh
d6f6155e534039619d616944528205ec3e388c9eda51787cbae329fcec10ec03
docker export -o out/helper-images/prebuilt-arm.tar gitlab-runner-prebuilt-arm-41d5c6ad
docker rm -f gitlab-runner-prebuilt-arm-41d5c6ad
gitlab-runner-prebuilt-arm-41d5c6ad
xz -f -9 out/helper-images/prebuilt-arm.tar
```

## 开始构建软件

当上面的步骤都就绪后，就可以正式开始编译构建 GitLab Runner 啦。

```bash
source ci/touch_make_dependencies
make build BUILD_PLATFORMS="-osarch='linux/amd64'"
```

`-osarch` 参数支持同时填写多个平台，来达到输出多个平台的可执行文件，比如填写 `linux/amd64 darwin/amd64`，将输出64位的Linux、OSx 系统的应用软件。

目前支持的平台有：

- `darwin/386` / `darwin/amd64`
-  `freebsd/386` / `freebsd/amd64` / `freebsd/arm`
-  `linux/386` / `linux/amd64` / `linux/arm`
-  `windows/386` /  `windows/amd64`

如果我们只构建 Linux 64 位系统下的应用，将看到类似下面的日志输出：

```bash
# make build BUILD_PLATFORMS="-osarch='linux/amd64'"

go get github.com/mitchellh/gox
# Building gitlab-runner in version 12.2.0~beta.1803.g41d5c6ad for -osarch='linux/amd64'
gox -osarch='linux/amd64' \
	-ldflags "-X gitlab.com/gitlab-org/gitlab-runner/common.NAME=gitlab-runner -X gitlab.com/gitlab-org/gitlab-runner/common.VERSION=12.2.0~beta.1803.g41d5c6ad -X gitlab.com/gitlab-org/gitlab-runner/common.REVISION=41d5c6ad -X gitlab.com/gitlab-org/gitlab-runner/common.BUILT=2019-08-01T17:04:48+0000 -X gitlab.com/gitlab-org/gitlab-runner/common.BRANCH=master -s -w" \
	-output="out/binaries/gitlab-runner-{{.OS}}-{{.Arch}}" \
	gitlab.com/gitlab-org/gitlab-runner
Number of parallel builds: 7

-->     linux/amd64: gitlab.com/gitlab-org/gitlab-runner
```

命令执行完毕后，我们可以在 `./out/binaries/` 目录内看到我们想要的二进制程序。试着运行一下吧：

```bash
# ./out/binaries/gitlab-runner-linux-amd64 -v
Version:      12.2.0~beta.1803.g41d5c6ad
Git revision: 41d5c6ad
Git branch:   master
GO version:   go1.8.7
Built:        2019-08-01T17:04:48+0000
OS/Arch:      linux/amd64
```

如果你也能看到类似的输出，那么源码编译 GitLab Runner 的任务，就这么愉快的结束啦。

上述问题解决方案来自项目 [.gitlab-ci.yml](https://gitlab.com/gitlab-org/gitlab-runner/blob/master/.gitlab-ci.yml) 持续集成配置文件，感兴趣的同学可以了解下。

## 最后

《编程匠艺》曾提过不应把过时错误的信息提供给你的伙伴，要维护良好的文档。然而现实中充满了过时错误的信息，就像本例中一样，作为一款开源软件，这些错误的信息难免会浇灭外部贡献者的热情。

下一篇文章，我将讲讲我为什么要编译 GitLab。

—EOF
