# 从定制 Ghost 镜像聊聊优化 Dockerfile

在[《修理 Ghost 中文输入法的 BUG》](https://soulteary.com/2020/01/19/bugfix-for-ghost-editor-cjk-input.html)一文中，通过给源码打补丁，并进行编译的方式，我们解决了 Ghost 的“陈年固疾”：不能正常输入中文。

两个月过去了，Ghost 开启了鸡血模式，不讲道理的更新了若干版本，从当时的 3.3.0 飙升至 3.9.0，考虑到项目中有依赖 Ghost，需要持续的更新维护，那么就在这里分享一下，如何更好的折腾它。

## 写在前面

在[GitHub](https://github.com/soulteary/youling) 的仓库中，我们可以看到，解决这个 Bug 需要两步走：

- 对管理后台的前端实现代码进行补丁，并重新构建
- 对管理后台的服务器端渲染模版进行更新

而在使用和维护上，必须考虑以下几点：

- 补丁内容是否会影响现有逻辑
- 是否可以不干扰用户使用官方镜像
- 是否可以尽可能少/不编码，实现镜像的维护更新
- 用于构建修正过前端功能的工具镜像性能能否更高

由于 Ghost 服务端脚本/模版不需要构建使用，我们以修改处理比较“麻烦”的 Ghost 前端资源为例，讲讲如何优化 Dockerfile。

## 优化构建镜像

在代码仓库中，我们可以看到 `Dockerfile` 的内容是这样编写的：

```bash
FROM node:12-alpine
LABEL maintainer="soulteary@gmail.com"
 
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US.UTF-8
ENV LC_ALL=en_US.UTF-8

RUN echo '' > /etc/apk/repositories && \
    echo "https://mirror.tuna.tsinghua.edu.cn/alpine/v3.10/main"         >> /etc/apk/repositories && \
    echo "https://mirror.tuna.tsinghua.edu.cn/alpine/v3.10/community"    >> /etc/apk/repositories && \
    echo "Asia/Shanghai" > /etc/timezone

RUN apk update && apk add git && \
    yarn global add knex-migrator grunt-cli ember-cli bower

COPY patches/mobiledoc-kit/event-manager.js /patches/mobiledoc-kit/event-manager.js

RUN git clone https://github.com/TryGhost/mobiledoc-kit.git /mobiledoc-kit && \
    cd /mobiledoc-kit && \
    git checkout 3b0f375d32f7183a4eee9cce5373ebabeb249165 && \
    cp /patches/mobiledoc-kit/event-manager.js /mobiledoc-kit/src/js/editor/event-manager.js && \
    yarn && \
    cp -r /mobiledoc-kit/dist /patches/mobiledoc-kit/dist && \
    rm -rf /mobiledoc-kit

RUN git clone --recurse-submodules https://github.com/TryGhost/Ghost.git /Ghost && \
    cd /Ghost && \
    git checkout 3.3.0 && \
    yarn setup

RUN rm -rf /Ghost/core/client/node_modules/\@tryghost/mobiledoc-kit/dist && \
    cp -r /patches/mobiledoc-kit/dist /Ghost/core/client/node_modules/\@tryghost/mobiledoc-kit/

WORKDIR /Ghost

RUN grunt prod

EXPOSE 2368

CMD ["npm", "start"]
```

这里存在几个问题：

1. “代码版本”被硬编码到了 Dockerfile 中，不利于`mobiledoc-kit` 和 `Ghost` 代码升级管理。
2. 对 Ghost 进行完整数据获取，是没有必要的。
3. 我们不确定是否能够继续对目标文件打补丁。

明确需要解决的问题之后，解决问题就容易多了。

### 解决硬编码的问题

我们首先需要将“版本”定义为变量，然后抽象出来，考虑到不希望未来每次代码升级都需要修改 Dockerfile，我们可以使用 它的 `ARG` 指令，对于原始内容进行优化，例如：

```bash
# FOR GHOST 3.9.0
ARG MOBILEDOC_KIT_VERSION=v0.11.1-ghost.4
RUN git clone https://github.com/TryGhost/mobiledoc-kit.git /mobiledoc-kit && \
    cd /mobiledoc-kit && \
    git checkout $MOBILEDOC_KIT_VERSION && \
    cp /patches/mobiledoc-kit/event-manager.js /mobiledoc-kit/src/js/editor/event-manager.js && \
    yarn && \
    cp -r /mobiledoc-kit/dist /patches/mobiledoc-kit/dist && \
    rm -rf /mobiledoc-kit
```

未来如果 Ghost 发布 `4.0.0`，这个依赖的组件也有了版本变化，那么在构建的时候只需要添加构建参数，即可完成新版本镜像的构建，而不用在修改 Dockerfile，像是这样：

```bash
docker build --build-arg MOBILEDOC_KIT_VERSION=v0.11.1-ghost.5
```

### 只获取必要的代码

原始的 Dockerfile 中，我们获取 Ghost 源码将其整个仓库都下载下来，在网络条件不好的时候，非常影响构建。

所以可以通过限定 `depth` 克隆深度，以及 `branch` 下载分支，限定要获取的代码量，只下有用的内容。

```bash
ARG GHOST_RELEASE_VERSION=3.9.0
RUN git clone --recurse-submodules https://github.com/TryGhost/Ghost.git --depth=1 --branch=$GHOST_RELEASE_VERSION /Ghost && \
    cd /Ghost && \
    yarn setup
```

### 判断是否能够对当前文件打补丁

可以判断的方法很多，但是最简单的自动化方案莫过于判断代码文件是否被修改过。

先使用 `shasum` 或者任何你用的顺手的计算工具，对目标要进行补丁的文件进行校验值计算，如果你使用的镜像的基础系统是 Ubuntu 可以使用下面的方式进行校验：

```bash
# 计算校验值
shasum -a 256 path_to_project/event-manager.js

0faeb4eb1cd177f3d7972fda6c52a0089a4814171cedd5c43a7302f027b26723  path_to_project/event-manager.js

# 进行校验
echo "0faeb4eb1cd177f3d7972fda6c52a0089a4814171cedd5c43a7302f027b26723  path_to_project/event-manager.js" | shasum -a 256 -c
```

这里使用 Alpine 发行版作为系统的容器，在不安装额外软件的情况下，可以换为 `md5sum` 来进行计算，原来的 Dockerfile 可以更新成下面这样：

```bash
# FOR GHOST 3.9.0
ARG MOBILEDOC_KIT_VERSION=v0.11.1-ghost.4
ARG EVENT_MANAGER_HASH=9a0456060f1c816a0a66bdcf3363e928
RUN git clone https://github.com/TryGhost/mobiledoc-kit.git /mobiledoc-kit && \
    cd /mobiledoc-kit && \
    git checkout $MOBILEDOC_KIT_VERSION && \
    (echo "$EVENT_MANAGER_HASH  /mobiledoc-kit/src/js/editor/event-manager.js" | md5sum -c -s -) && \
    cp /patches/mobiledoc-kit/event-manager.js /mobiledoc-kit/src/js/editor/event-manager.js && \
    yarn && \
    cp -r /mobiledoc-kit/dist /patches/mobiledoc-kit/dist && \
    rm -rf /mobiledoc-kit
```

如果校验值和我们传递的不一致，构建会自动中断，如果发生这个状况，那么理论来说我们需要调整补丁逻辑，并计算出新的文件的校验值。

## 完整的镜像文件

为了方便有相同需求的同学，这里给出完整的镜像文件，相关代码也已经上传 [GitHub](https://github.com/soulteary/youling)。

```bash
FROM node:12-alpine
LABEL maintainer="soulteary@gmail.com"

ENV LANG en_US.UTF-8
ENV LANGUAGE en_US.UTF-8
ENV LC_ALL=en_US.UTF-8

RUN echo '' > /etc/apk/repositories && \
    echo "https://mirror.tuna.tsinghua.edu.cn/alpine/v3.10/main"         >> /etc/apk/repositories && \
    echo "https://mirror.tuna.tsinghua.edu.cn/alpine/v3.10/community"    >> /etc/apk/repositories && \
    echo "Asia/Shanghai" > /etc/timezone

RUN apk update && apk add git && \
    yarn global add knex-migrator grunt-cli ember-cli bower

COPY patches/mobiledoc-kit/event-manager.js /patches/mobiledoc-kit/event-manager.js

ARG GHOST_RELEASE_VERSION=3.9.0
RUN git clone --recurse-submodules https://github.com/TryGhost/Ghost.git --depth=1 --branch=$GHOST_RELEASE_VERSION /Ghost && \
    cd /Ghost && \
    yarn setup

# FOR GHOST 3.9.0
ARG MOBILEDOC_KIT_VERSION=v0.11.1-ghost.4
ARG EVENT_MANAGER_HASH=9a0456060f1c816a0a66bdcf3363e928
RUN git clone https://github.com/TryGhost/mobiledoc-kit.git /mobiledoc-kit && \
    cd /mobiledoc-kit && \
    git checkout $MOBILEDOC_KIT_VERSION && \
    (echo "$EVENT_MANAGER_HASH  /mobiledoc-kit/src/js/editor/event-manager.js" | md5sum -c -s -) && \
    cp /patches/mobiledoc-kit/event-manager.js /mobiledoc-kit/src/js/editor/event-manager.js && \
    yarn && \
    cp -r /mobiledoc-kit/dist /patches/mobiledoc-kit/dist && \
    rm -rf /mobiledoc-kit

RUN rm -rf /Ghost/core/client/node_modules/\@tryghost/mobiledoc-kit/dist && \
    cp -r /patches/mobiledoc-kit/dist /Ghost/core/client/node_modules/\@tryghost/mobiledoc-kit/

WORKDIR /Ghost

RUN grunt prod

EXPOSE 2368

CMD ["npm", "start"]
```

## 其他

处理软件的版本更新，多多少少需要做一些简单的分析，下面这些在线链接或许可以帮到你（以这次处理 3.9.0 为例）：

- 确认新旧版本差异：[https://github.com/TryGhost/Ghost/compare/3.3.0...3.9.0](https://github.com/TryGhost/Ghost/compare/3.3.0...3.9.0)
	- 老版本需要放在前面，不然按照时间线，是一定对比不出内容的。
- 确认新版本引用子模块版本：[https://github.com/TryGhost/Ghost/tree/3.9.0/core](https://github.com/TryGhost/Ghost/tree/3.9.0/core)
	- Ghost 的管理后台、主题使用子模块方式引入，需要单独检查并确认引用。
	- 也可以 Clone 到本地，检查 `.submodule` 文件，不过需要两个步骤操作，略显麻烦。
- 确认新版本的子模块依赖内容：[https://github.com/TryGhost/Ghost-Admin/blob/3.9.0/package.json](https://github.com/TryGhost/Ghost-Admin/blob/3.9.0/package.json)
	- 检查是否还存在 `@tryghost/mobiledoc-kit` 依赖，并确认其版本。
- 确认编辑器组件模版：[https://github.com/TryGhost/mobiledoc-kit/compare/v0.11.1-ghost.4...3b0f375d32f7183a4eee9cce5373ebabeb249165](https://github.com/TryGhost/mobiledoc-kit/compare/v0.11.1-ghost.4...3b0f375d32f7183a4eee9cce5373ebabeb249165)
	- 确认模块是有效的，并且查看是否有比较大的逻辑变更。

## 最后

下一篇 Ghost 相关的内容，或许会聊聊怎么在容器中使用阿里云（oss）/腾讯云（cos）对象储存，以及如何搭配 SSO 单点登录使用 Ghost。

--EOF
