# 使用私有CI构建GitHub仓库

最近使用 Drone 改造了网站的构建发布流程，感觉比较轻量，用起来还不错，记录下来留给后面感兴趣的同学参考。

## 起因

在[五月初](https://soulteary.com/2018/05/07/refactor-and-recent.html)进行网站简化的过程中，因为使用了自定义的[代码高亮](https://github.com/soulteary/crayon-syntax-highlight)服务，和[文档转换服务](https://github.com/soulteary/posts-adapter-pure-to-hugo)，对于构建机器的配置要求升高到了`1C2G`，对PHP Fpm脚本进行高并发调用的时候，资源消耗太大了，即使设定了并发数限制，还是偶尔会出现500状态。

于是把构建机器从`1C1G`升配到了`2C2G`，这样倒是解决了问题，但是构建服务使用频率不是很高，加上海外机器线路进行国内机器部署的时候，**偶尔使用也会遇到构建十几秒，部署十分钟**。

最开始构建服务是跑在家里服务器的，博客文章源码也是在服务器上的，更新、构建统一使用  `GitLab + GitLab Runner` ，倒也还算稳定，只是：

1. 更新必须使用私有云的私有仓库。
2. 构建环境需要整体打包成一个容器，不然分阶段执行，要贮存 `artifacts`。（家里没有上k8s）

因为生成工具我都容器固化了，所以不想贮存 `artifacts` ，即使能设置 `expires`，过期自动删除，但是也太不环保了。

另外，我希望文档是可以保存在GitHub上，而非私有仓库中的，同时也不想把文档推送多个`Git Origin`或者做mirror什么的，维护起来太麻烦了。

这一套架构使用体验贼差，于是考虑进行一番改进。

## 历史

说起 Drone，去年年底机器资源严重不足的时候，就考虑过找一个轻量的CI工具，暂时替换一下GitLab CI，等资源充沛之后，再替换回来。

当时看了一圈，感觉drone还不错，能够直观看到的不足只有：文档中有大量未解之谜、docker镜像缺乏明确的tags、最新版本未同步docker hub。

但是相比 `GitLab + GitLab Runner` 的组合拳，资源占用小，还能灵活的接入各种仓库，也是基于容器化分阶段进行执行操作，还是很有竞争力的。

当时临时使用过一阵 `gogs + drone`，在一台1C1G的机器上跑着就很欢腾，但是遇到几个小问题：

- 对于个别仓库支持mirror，但是对于mirror不支持自动触发CI构建，要维护一个脚本定时检查并触发，仓库源也需要维护至少两份（源+Mirror）。
- 出于洁癖gogs使用22作为Git Server Port，服务器经常收到爬虫探测，不胜其烦，故还是不考虑把仓库暴露在公网了，并且GitHub免费仓库+私有云仓库已经很好了，不想多维护一个仓库服务。

思来想去，在海外购置了一台1C2G的机器，临时的解决了这个问题，然后就遇到了文章开头的情况。

## 架构简化

GitHub 不可用时其实很少，所以可以考虑把本来就是要公开的内容直接托管GitHub，然后使用CI与其直接对接，省去部分维护仓库的成本。

这里没有使用GitLab、以及GitLab Runner直接与其对接，毕竟有的半成品还是不想公开，而且我对这个大块头的安全性很不放心，成天到晚组件有安全漏洞...

那么公网搭建一个drone便是一个不错的选择，小巧轻量，自带GitHub OAuth插件，还能直接签注SSL证书。

但是考虑到构建机器对资源配置有一定的要求，并且部署对于国内线路有一定的要求，于是考虑在内网启动一个Drone服务，对外暴露到公网机器上即可：外网机器只是充当一个反向代理，充分利用家里高配置机器资源和大带宽。

简单的架构图如下:

```uml

[外网机器]<->[内网机器]
[内网机器|
  [构建服务]->[发布国内云服务器]
]

```

数据透传，我选择了国人开源产品frp取代了ngrok。

## 数据透传

关于frp的配置其实很简单：


服务端配置， frps.ini

```Toml
[common]
# 如果你计划多添加一层nginx，这个端口需要修改掉，避免冲突
# 我这边懒得加一层nginx，直接使用frp暴露远程端口
vhost_http_port  = 80
vhost_https_port = 443

# 客户端和服务端沟通端口
bind_port        = 7000

token            = 需要和客户端设置一致

# 管理面板相关配置，如果不进行设置，默认使用 admin

# 对外服务使用0.0.0.0，不然可以使用127.0.0.1，配合代理进行访问
dashboard_addr   = 0.0.0.0
dashboard_port   = 你的端口
dashboard_user   = 你的账号
dashboard_pwd    = 你的密码
```

需要注意的是，你计划暴露的端口，记得在防火墙里打开：

```bash
ufw allow 你的端口号
```

下面是客户端配置，frpc.ini

```Toml
[common]
server_addr      = 你的服务器IP或者私有域名
# 客户端和服务端沟通端口
server_port      = 7000

token            = 需要和客户端设置一致
dns_server       = 你的DNS服务，不设置默认使用8.8.8.8

[web01]
type     = http
local_ip = 0.0.0.0
local_port     = 17071
custom_domains = 公网要暴露的域名

[web02]
type = https
local_ip   = 0.0.0.0
local_port = 17072
use_encryption  = false
use_compression = false
custom_domains  = 公网要暴露的域名

```

这里有两点要注意的：

1. 你的内网服务，往往会指定一些私有DNS解析，需要配置common节中的`dns_server`为你内网的DNS服务器地址，比如网关路由，或者DNS服务器地址，避免在内网解析内部服务的时候使用的是公网服务，找不到域名，无法提供服务。
2. 你的公网暴露域名可以是任意域名，只要外网DNS能够指向到你的`server_addr`即可。

## CI服务

drone配置起来很简单，就像下面的配置：

```yaml
version: '2'

services:
  drone-server:
    restart: always
    image: drone/drone:0.8.5
    # 根据自己情况进行端口映射，但是要配套修改上面数据透传里的客户端端口
    ports:
      - 17071:80
      - 17072:443
    volumes:
    # 这里如果是OSX系统，需要mkdir或者把映射目录进行修改
      - /var/lib/drone:/var/lib/drone/
    restart: always
    environment:
      # 这个变量务必设置，否则软件将会以调试模式执行
      - GIN_MODE=release
      - DRONE_OPEN=false
      - DRONE_SECRET=和drone-agent设置一致即可
      - DRONE_ADMIN=soulteary
      - DRONE_HOST=https://你的域名
      - DRONE_GITHUB=true
      - DRONE_GITHUB_URL=https://github.com
      - DRONE_GITHUB_CLIENT=到GitHub获取
      - DRONE_GITHUB_SECRET=到GitHub获取
      - DRONE_GITHUB_SCOPE=repo,repo:status,user:email,read:org
      # 是否自动注册SSL证书
      - DRONE_LETS_ENCRYPT=true

  drone-agent:
    restart: always
    image: drone/agent:0.8.5
    command: agent
    restart: always
    depends_on:
      - drone-server
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - GIN_MODE=release
      - DRONE_SERVER=drone-server:9000
      - DRONE_SECRET=和drone-server设置一致即可
```

网上很多教程使用的通讯端口都进行了暴露，这里其实大可不必，docker-compose编排的软件直接默认可访问。
另外，根据日志反馈，需要额外定义`GIN_MODE`，避免软件执行在调试状态下。

## 定义CI过程


在你的GitHub仓库中添加一个drone.yml，内容可以参考如下：

```yaml
pipeline:

  同步数据:
    image: appleboy/drone-scp
    secrets:
      - source: deploy_key
        target: ssh_key
      - source: deploy_host
        target: ssh_host
      - source: deploy_port
        target: ssh_port
      - source: deploy_user
        target: ssh_username
    target: ~/Code/www.soulteary.com/My-Blog-Posts/
    source: ./*


  转换文档:
    image: appleboy/drone-ssh
    secrets:
      - source: deploy_key
        target: ssh_key
      - source: deploy_host
        target: ssh_host
      - source: deploy_port
        target: ssh_port
      - source: deploy_user
        target: ssh_username
      - source: excute_timeout
        target: ssh_command_timeout
    script:
      - cd ~/Code/www.soulteary.com/blog/
      - ./1.convert.sh


  开始构建:
    image: appleboy/drone-ssh
    secrets:
      - source: deploy_key
        target: ssh_key
      - source: deploy_host
        target: ssh_host
      - source: deploy_port
        target: ssh_port
      - source: deploy_user
        target: ssh_username
      - source: excute_timeout
        target: ssh_command_timeout
    script:
      - cd ~/Code/www.soulteary.com/blog/
      - ./3.update.sh


  压缩页面:
    image: appleboy/drone-ssh
    secrets:
      - source: deploy_key
        target: ssh_key
      - source: deploy_host
        target: ssh_host
      - source: deploy_port
        target: ssh_port
      - source: deploy_user
        target: ssh_username
      - source: excute_timeout
        target: ssh_command_timeout
    script:
      - cd ~/Code/www.soulteary.com/blog/
      - ./4.minify.sh


  产物打包:
    image: appleboy/drone-ssh
    secrets:
      - source: deploy_key
        target: ssh_key
      - source: deploy_host
        target: ssh_host
      - source: deploy_port
        target: ssh_port
      - source: deploy_user
        target: ssh_username
      - source: excute_timeout
        target: ssh_command_timeout
    script:
      - cd ~/Code/www.soulteary.com/blog/
      - ./5.package.sh


  尝试发布:
    image: appleboy/drone-ssh
    secrets:
      - source: deploy_key
        target: ssh_key
      - source: deploy_host
        target: ssh_host
      - source: deploy_port
        target: ssh_port
      - source: deploy_user
        target: ssh_username
      - source: excute_timeout
        target: ssh_command_timeout
    script:
      - cd ~/Code/www.soulteary.com/blog/
      - ./6.deploy.sh
      - ./7.notice.sh
```

这里是有drone-scp和drone-ssh两个插件，通过drone依次在远程服务器上进行了一些脚本的执行。
如果你的构建服务简单，还可以直接写在scripts中，直接在docker容器中执行CI过程。

这里有几点需要注意：

1. 插件没有使用docker tag进行固化，尽量自行保存容器镜像。
2. 定义`deploy_*`变量的时候，`ssh_host`如果是内网服务，记得进行内网解析。
3. `excute_timeout`单位是秒。
4. 在OSX环境执行的话，会丢失PATH变量，需要在执行脚本中重新声明变量：

```bash
sysOS=`uname -s`
if [ $sysOS == "Darwin" ];then
    PATH=/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin
fi
```


## 启动服务

先在服务器启动frp的服务端：

```bash
frps -c frps.ini
```

你可以使用进程守护软件保障进行健康，这里不做展开，网上文章参考讲了太多。

接着在内网服务器启动frp的客户端：

```bash
frpc -c frpc.ini
```

和服务端一样，可以使用进程守护来进行可用性保障。

然后启动你的drone服务：

```bash
docker-compose -f docker-compose.yml up -d
```

然后访问你的域名，即可看到drone暴露在公网上，并且通过OAuth和GitHub产生了“联动”效果。

接下来向你的GitHub进行一次提交、合并操作，你会发现CI已经在进行自动的构建处理了。



---EOF

