# 如何搭配 CI 系统使用 Composer

上一篇文章讲了如何搭建[高性能的 Composer](https://soulteary.com/2019/08/23/build-a-high-performance-private-composer-image-service.html) 服务，本篇来聊聊如何搭配 CI 进行使用，让研发效率有一定的保障。

## 写在前面

本文以 GitLab Runner 中最简单通用的 `shell`模式为例，其他 CI 系统类似，酌情修改脚本即可。考虑到持续集成过程中需要进行资源隔离，我们使用工具容器作为持续集成环境。

## 定义阶段

在使用 CI 前，我们需要先拆分阶段，一般来说，基于 Composer 的项目存在三个阶段：

- 更新代码：`composer install` 阶段
- 部署代码：`sync release` 阶段
- 重启服务：`reload/restart service` 阶段

如果写成 yml 配置，可以这样描述：

```yaml
stages:
  - update
  - deploy
  - restart
```

定义完毕阶段后，需要详细描述每阶段做的事情。

先从 `update` 阶段开始说起。

## 更新代码

为了方便描述，项目结构简单定义为下面这样。

```TeXT
.
├── composer.lock
└── composer.json
```

更新代码最简单的方案便是进入项目目录，执行 `composer i` ，等待项目安装完毕了。然而这样会导致两个问题：

- CI 构建机需要安装并维护 composer，构建机器越多，管理成本越高
- 不同项目必须使用同一份配置，构建机的缓存不能够独立管理

所以如果使用固定配置构建的工具镜像，搭配“即用即丢”模式来用做构建执行环境，上面的问题就迎刃而解了。

```bash
docker run \
	--add-host composer.lab.com:192.168.123.234 \
	--volume $PWD:/app \
	composer /bin/sh -c "composer install && ls -al vendor"
```

使用上面的方案，搭配 `部署令牌` ，除了解决常规依赖的获取外，也不难解决下面这种类型的软件包的获取。

```bash
"repositories": [
    {
      "type": "vcs",
      "url": "https://gitlab+deploy-token-2:ZeYhk_F7oqtdxXuKsvvr@gitlab.lab.com/forum/session.git"
    },
    ...
]
```

一切似乎很美好，但是如果涉及到下面这类仓库，上面的方案就失灵了。

```bash
"repositories": [
    {
      "type": "vcs",
      "url": "ssh://git@gitlab.lab.com/forum/session.git"
    },
    ...
]
```

使用 Git 获取 SSH 协议的仓库数据，需要配置 SSH KEY。解决的方案也不难，为上面的 docker 命令加一些额外的参数：

```bash
docker run \
	--add-host composer.lab.com:192.168.123.234 \
	--volume $PWD:/app \
	--volume /etc/passwd:/etc/passwd:ro \
	--volume /etc/group:/etc/group:ro \
	--user $(id -u):$(id -g) \
	composer /bin/sh -c "composer install && ls -al vendor"
```

为了追求简单优雅，我们可以将上面的代码进一步优化，改为用 `docker-compose` ，像是下面这样。

```yaml
version: '3.6'

services:

  composer:
    image: composer
    volumes:
      - ./:/app
      - ./known_hosts:/root/.ssh/known_hosts
      - $HOME/.ssh/id_rsa:/root/.ssh/id_rsa
    command: install
```

你会发现这里多映射了一个 `known_hosts` 文件。由于 SSH 设计上防止中间人攻击，需要验证服务端的指纹，所以，我们需要将服务端指纹保存下来，否则当 `composer` 容器访问代码仓库服务器的时候，会因为下面的错误而中止仓库 Clone 。

```TeXT
Could not open a connection to your authentication agent.
```

这里通过 `ssh-keyscan` 来生成签名文件：`ssh-keyscan gitlab.lab.com > known_hosts` ，文件内容会类似下面：

```bash
ssh-keyscan gitlab.lab.com
# gitlab.lab.com:22 SSH-2.0-OpenSSH_7.2p2 Ubuntu-4ubuntu2.8
gitlab.lab.com ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCyC3qB2M68KeK79op1vdoYYbX+Y3G768DVV+nki0ujUSHsedCo77sfkTIU4ZHAojEn4w/oqAjDcz3bDJNEI/CIO3k1NeiM3tZbkzm8VKTGhXkvxbTYkAud3f/UbXij3LAhsskn8ykoxgl7qqQ4dH2y0v9oBARNh2fRNyENaj4Tvu3Ao8MIh4JWD69u3AcNvQwJWtqphkY7xbvFDb2GNzNrTW+X1R2jHsqCxq8Nuq6tCC9m+JNGtUR6IsZusm5B2/CtamxUSk+1YEtNsWxEJ2qikm7Ud+Ikkak9N+8xZ/Ck6lmjoAxlKwgoP7Tq6cDDfZrQt8drwLE1/j1hzrE7TfoR
# gitlab.lab.com:22 SSH-2.0-OpenSSH_7.2p2 Ubuntu-4ubuntu2.8
gitlab.lab.com ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBFzJ/O/QQQ0RsEBALP/UeiZa58IDpI+c4RRjS9Vd1nMFY//xFuS1UF6ml8mpRpaYwSjxgQG9lNbXlL9E9AoAslo=
# gitlab.lab.com:22 SSH-2.0-OpenSSH_7.2p2 Ubuntu-4ubuntu2.8
gitlab.lab.com ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIGkm3tji72xkD5ySTJo6NIeaE/zqhedlgkw5Y59quy6X
```

使用 `compose` 来描述内容，最后的 CI 脚本可以简化为：

```yaml
更新代码:
  stage: update
  script:
    - docker-compose up && ls -al
    - chown -R $(id -u):$(id -g) ./vendor
```

## 部署代码

部署代码可以做的简单些，比如直接用 rsync ，可以开启 `- I` 选项保证一致性，或者老老实实 ansible ，如果追求一步到位，把代码打包镜像分发也没有问题。先以 rsync 为例，ansible 足够再写一篇啦。

生成一枚 SSH 密钥用于部署，然后项目仓库/代码主机中配置信任该密钥，rsync 使用的话，手段就灵活多了：

- 使用 CI 变量储存 / 使用配置服务API获取
- 预先分配 KEY 到 CI 程序用户 `.ssh` 目录中

和上面一样同样考虑将工具打包为镜像使用：

```bash
FROM alpine:3.9

RUN echo '' > /etc/apk/repositories && \
    echo "https://mirror.tuna.tsinghua.edu.cn/alpine/v3.9/main"         >> /etc/apk/repositories && \
    echo "https://mirror.tuna.tsinghua.edu.cn/alpine/v3.9/community"    >> /etc/apk/repositories && \
    echo "Asia/Shanghai" > /etc/timezone

RUN apk update && \
    apk upgrade && \
    apk add --no-cache rsync openssh-client && \
    rm -rf /var/cache/apk/*
```

上面的 Dockerfile 演示了如何构建一个完全独立于系统，大小只有 `10MB` 的 Rsync 工具包。

使用命令也很简单，和传统的 Rsync 别无二致，只是前面加上了 `docker run` 命令：

```bash
docker run \
	--volume $DEPLOY_DIR/ssh.key:/ssh.key \
	--volume $PWD:/app \
	$RSYNC_TOOL rsync -az --list-only --include='vendor' --exclude='.git*' --timeout=3600 -P --partial --delete -e 'ssh -i /ssh.key  -p 22 -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null' /app/ $DEPLOY_TARGET_01:$PROJECT_WORKDIR
```

同样的，如果你愿意的话，这个命令也可以重写为更优雅一些的 `compose` 脚本。

此外，如果你想部署多台服务器，想将上一步的“过程产物”共享，需要定义缓存目录，本例中可以这样配置：

```yaml
cache:
  paths:
    - vendor
```

## 重启服务

重启/重载服务其实没有什么难的，如果业务需要平滑重启，可以将脚本配置成串行任务：

- 重启服务第一批
- 重启服务第二批
- …

前端不论是 Nginx 还是负载均衡服务，会根据服务可用情况帮助你平滑更新线上应用。

此外，如果你的服务部署更新后，需要触发其他项目的构建过程，可以使用 GitLab API：`pipeline trigger`，调用手段很多，最简单的莫过于使用 `curl` 进行调用，放在当前项目 CI 脚本的合适位置即可：

```bash
curl -X POST -F token=$DEP_PROJECT_TOKEN -F ref=$DEP_PROJECT_REF_NAME https://gitlab.lab.com/api/v4/projects/$DEP_PROJECT_ID/trigger/pipeline
```

上面的变量可以在 GitLab CI 帮助文档和项目配置中获得，在此就不赘述了。

## 最后

这里给出一套简单的参考配置，里面演示了如何使用不同的命令来进行项目部署：

```yaml
variables:
  RSYNC_TOOL:           "docker.lab.com/rsync-tool:1.0.0"
  CACHE_DIR:            "/data/caches/$CI_PROJECT_ID"
  SOURCE_DIR:           $CI_PROJECT_DIR
  PROJECT_WORKDIR:      /data/forum/$CI_PROJECT_NAME
  DEPLOY_TARGET_01:     $DEPLOY_HOST_01
  DEP_PROJECT_TOKEN:    cb0550eeaa6c6a696e6ada8b041234
  DEP_PROJECT_REF_NAME: master
  DEP_PROJECT_ID:       11

stages:
  - update
  - deploy
  - restart

cache:
  paths:
    - vendor

下载依赖:
  stage: update
  script:
    - docker-compose up && ls -al
    - chown -R $(id -u):$(id -g) ./vendor

分发依赖01:
  stage: deploy
  before_script:
    - DEPLOY_DIR=$CACHE_DIR && mkdir -p $DEPLOY_DIR && echo "$DEPLOY_KEY" | tr -d '\r' >$DEPLOY_DIR/ssh.key && chmod 600 $DEPLOY_DIR/ssh.key
  script:
    - docker run --volume $DEPLOY_DIR/ssh.key:/ssh.key --volume $PWD:/app $RSYNC_TOOL rsync -azi --list-only --include='vendor' --exclude='.git*' --timeout=3600 -P --partial --delete -e 'ssh -i /ssh.key  -p 22 -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null' /app/ $DEPLOY_TARGET_01:$PROJECT_WORKDIR

重启服务:
  stage: restart
  script:
    - "curl -X POST -F token=$DEP_PROJECT_TOKEN -F ref=$DEP_PROJECT_REF_NAME https://gitlab.lab.com/api/v4/projects/$DEP_PROJECT_ID/trigger/pipeline"
```

先写到这里吧。

—EOF
