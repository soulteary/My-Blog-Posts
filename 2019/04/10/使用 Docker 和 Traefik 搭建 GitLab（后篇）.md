# 使用 Docker 和 Traefik 搭建 GitLab（后篇）

[前篇文章](https://soulteary.com/2019/04/10/gitlab-was-built-with-docker-and-traefik-part-1.html)提到了要介绍一些 GitLab 安全配置上的问题，本篇文章就来简单聊聊如何加固你部署在公网上的 GitLab 代码仓库。

<!-- more -->

## 问题背景

一般来说，我们建议在内网环境下使用 GitLab 这类**包含大量敏感信息**，相对**大型**、**复杂**的软件，因为网络的隔离，天然可以减少许多“可攻击面”。

如果需要外部使用，则推荐使用 “专用隧道” 等方式提供定向的流量访问途径。

前文提到过，这次遇到的诉求恰恰是：

- 不能使用专有的流量隧道
- 不能搭建在内网，要提供公网访问方式

没关系，见招拆招即可。 

## 分析攻击基本面

在暂时不考虑内鬼作祟的情况下，对应用进行环境加固。可以先对可以受到攻击的场景进行简单的分类：

- 系统层（宿主机环境）
- 网络层（具备流量交互的场景）
- 应用层（软件运行环境、相关配置）
- 用户层（软件使用日常场景）

考虑篇幅、文章针对性两个原因，本文重点介绍“应用层”安全加固。

## 系统加固

网上关于系统加固的资料有不少，我之前的文章中也有零碎的提过，感兴趣可以自行翻看。为了方便你进行搜索，这里简单提一下常规手段：

- 针对敏感端口进行日志审计。
- 修改系统敏感端口，比如 SSH 端口，并考虑按需开启。
- 限制登录用户和用户登录方式（比如堡垒机、或者秘钥认证），并创建普通用户取代 `root` 进行日常操作。
- 购买云安全服务，并对接监控告警。

## 网络加固

网络加固的内容比较杂，我们从下面五个点展开。

### 加密流量传输

网络加固这里有一个简单原则，除了本机流量外，但凡可以使用 SSL 加密的流量，一律使用 SSL 加密模式进行传输，包括：

- 跨主机之间的系统调用
- 应用和数据库之前的调用

虽然不进行 `SSL` 化配置，部署和编码成本会有所提高，如果机器资源紧张，还可能影响一些性能，并且还可能带来额外的费用问题：

- 企业使用的SSL证书按年付费，价格十分昂贵。
- 不过如果你觉得购买证书费用太过高昂，也可以使用自签名证书来解决问题。

实际上，部署 SSL 所带来的各种成本放到长期来看，都是可以忽略不计的一次性投入，但是**安全风险问题是基础和底线，不值得为此冒险**。

### 避免公开的 DNS 解析

提到网络服务，其中有一点经常被忽略：**DNS 解析**。

如果你的应用只针对少数人提供服务，不妨考虑**不在公网 DNS 上进行解析**，仅通过绑定 `Hosts` 提供服务。

配合 Traefik 的服务发现功能，如果对方不知道你的服务域名，即使通过 IP 扫描到你的站点，请求后得到的结果也只有 `404 Not Found`。

### 添加网络请求验证

上一条措施，不进行公网域名暴露，已经可以解决一大部分扫描器的嗅探。但是面对有针对性的攻击，这招就不灵光了。

此时，建议在我们的 Web 系统前添加一层基础的用户验证：`Baisc Auth`。

使用 Traefik 添加这层验证很容易，只需要下面两行简单的声明：

```yaml
- "traefik.gitlab.frontend.auth.basic=${BASIC_AUTH}"
- "traefik.gitlab.frontend.auth.basic.removeHeader=true"
```

这两行配置的作用是：

- 第一行告诉程序，我们要使用 Basic 认证，认证的用户名密码是什么。
- 第二行配置则告诉程序，这个认证仅仅在 Traefik 流量进入的时候使用，不要继续传递给应用程序，避免带来其他麻烦（比如 Confluence 这类应用会将 HTTP 请求头中的 `authorization` 用作系统登录凭据）。

当然，这里同样需要创建一个 `.env` 环境配置文件，比如：

```yaml
BASIC_AUTH=soulteary:$apr1$rgGAffTk$vDZ1tL03og0nZ8XlCfdv80
```

如果你好奇这段代码是如何生成的，可以在[使用 Docker 搭建 Confluence](https://soulteary.com/2019/03/30/construct-confluence-with-docker.html) 这篇文章中找到答案。

下面给出一个相对完整的配置参考：

```yaml
labels:
      - "traefik.enable=true"
      # GitLab Web 服务
      - "traefik.gitlab.frontend.auth.basic.removeHeader=true"
      - "traefik.gitlab.frontend.auth.basic=${BASIC_AUTH}"
      - "traefik.gitlab.port=80"
      - "traefik.gitlab.frontend.rule=Host:gitlab.${BASEHOST}"
      - "traefik.gitlab.frontend.entryPoints=http,https"
      - "traefik.gitlab.frontend.headers.SSLProxyHeaders=X-Forwarded-For:https"
      - "traefik.gitlab.frontend.headers.STSSeconds=315360000"
      - "traefik.gitlab.frontend.headers.browserXSSFilter=true"
      - "traefik.gitlab.frontend.headers.contentTypeNosniff=true"
      - "traefik.gitlab.frontend.headers.customrequestheaders=X-Forwarded-Ssl:on"
      - "traefik.gitlab.frontend.passHostHeader=true"
      - "traefik.gitlab.frontend.passTLSCert=false"
```

### 使用浮动 IP

如果对方不光使用侵入的方式进行攻击，还想让你暂时无法正常使用系统，比如对你进行令人发指的 DDoS 攻击。

作为被攻击方，可以使用 `浮动IP` 的方式，在遭遇攻击的时刻，降低切换 IP 的成本，快速金蝉脱壳，这里配合支持动态加速的 CDN 服务效果更好。

## 应用层

应用层做的事情也比较杂，我们来慢慢说起。

### 用户侧流量加密

建议系统不提供任何 `HTTP` 流量，防止用户侧流量被劫持利用。

所有出公网流量一律走 `HTTPS`，如果你也使用前文提到的 Traefik ，那么这个事情默认就是做好了的（参考刚刚的配置）。

### 对接 Prometheus 性能监控

如果你对可用性有很高的要求，可以参考官方文档，对接[ Prometheus 性能监控](https://docs.gitlab.com/ee/administration/monitoring/prometheus/)，如果你对Prometheus没有概念，可以先浏览一下官方的[在线示例](https://dashboards.gitlab.com/d/RZmbBr7mk/gitlab-triage?refresh=30s&orgId=1)，这部分展开聊可以写好几篇，先略过。

### 处理 CI Runner

CI 虽然作为呼之即来、挥之即去的“附加部分”，但是实际上也可以因为“频繁调用”而拒绝服务，或者因为不恰当的 CI 配置，而泄露敏感信息，或者作为攻击跳板，伤害到线上业务代码。

对于 GitLab CI Runner 运行监控，推荐使用 [ timoschwarzer/gitlab-monitor ](https://github.com/timoschwarzer/gitlab-monitor)，不过如果你在系统中配置好了推送消息，项目数量比较少的时候，一个手机Push过来，或许更方便迅捷。

对于 CI  Runner ，要确定尽可能少的提供 `SHELL` 模式的 Runner，多提供容器模式的 Runner，减少 Runner 攻击到宿主机的可能。

另外 Runner 可被触发的分支和仓库要做额外的限制，尽可能避免过度频繁的 Runner 执行，让宿主机器“过劳死”。

最后，Runner 中使用的环境变量和配置信息，需要使用加密环境变量的方式进行获取，而非明文写死在配置文件代码中。GitLab 这部分做的很好，有兴趣的小伙伴可以了解一下。

### 监控 GitLab SSH 端口

因为我们对用户提供了 `SSH` 的方式去 `Clone` 和 `Push` 代码，所以作为开放访问的 `SSH` 端口就面临被攻击的可能。

下面是一台长期运行在公网的代码仓库的端口日志（`cat logs/sshd/current`），我节选了比较有代表性的一部分日志，并隐去了具体时间：

```yaml
Invalid user admin from 179.53.182.234
Invalid user user from 183.89.94.13
Invalid user admin from 36.236.233.142
Invalid user admin from 143.255.154.219
Invalid user admin from 85.57.5.107
Invalid user ubnt from 85.57.5.107
Invalid user admin from 85.57.5.107
Invalid user admin from 156.223.73.14
Invalid user support from 200.145.6.88
Invalid user admin from 152.231.118.191
Invalid user user from 171.228.172.27
Invalid user admin from 27.66.79.45
Invalid user Admin from 117.0.57.69
Invalid user admin from 14.207.231.218
Invalid user gitlab from 121.71.20.66
Invalid user admin from 183.157.173.121
Invalid user guest from 37.214.104.206
Invalid user admin from 197.32.190.120
Invalid user Administrator from 125.34.196.43
Invalid user Administrator from 125.34.196.43
Invalid user \243\254git from 112.87.206.54
Invalid user \243\254git from 112.87.206.54
Invalid user admin from 116.118.104.96
Invalid user admin from 14.186.202.33
Invalid user admin from 113.172.217.15
....
```

可以看到有大量扫描器在默默的替你关注者你的系统安全，毫不夸张的说，一旦你漏出破绽，你的机器、你的应用就不归你使用了，这类扫描器的拥有者便能光明正大的随意用你的机器、玩你的系统、欺负你的用户…

如何避免这类恶意的扫描器呢？其实写一段简单的日志检测脚本就能解决很大一部分问题。

比如下面这段脚本，在参考 [这篇文章](http://www.cszhi.com/20120413/block-abuse-ssh-ip.html) 后，我结合实际情况，更新了它，让脚本能够处理 GitLab 的日志格式。

```bash
#!/bin/bash

# 允许错误尝试的最大次数
LIMIT=3
# 要进行分析的日志文件
SCAN_LOG="/data/gitlab/logs/sshd/current"
# 封禁IP记录
LOGFILE="/data/gitlab/logs/bad_gay_22_port.log"
# 要匹配的日志格式: 2019-04-10_12
TIME=$(date '+%Y-%m-%d_%H')

# 扫描当前 GitLab 日志，找出所有的错误登录行为，并进行计数，筛选出超过允许次数的 IP
BLOCK_IP=$(grep "$TIME" "$SCAN_LOG"|grep "Invalid user"|awk '{print $(NF-3)}'|sort|uniq -c|awk '$1>"$LIMIT"{print $1":"$2}')
for i in $BLOCK_IP
do
    IP=$(echo $i|awk -F: '{print $2}')
	# 验证该IP是否已经被封禁
    iptables-save|grep INPUT|grep DROP|grep $IP>/dev/null
	# 如果未被封禁，则进行封禁操作
    if [ $? -gt 0 ];then
        iptables -A INPUT -s $IP -p tcp --dport 22 -j DROP
        NOW=$(date '+%Y-%m-%d %H:%M')
        echo -e "$NOW : $IP">>${LOGFILE}
    fi
done
```

将上面的内容保存为 `gitlab_ssh.sh` ，然后赋予脚本可执行权限。

```bash
chmod 755 gitlab_ssh.sh && chmod +x gitlab_ssh.sh
```

接着将脚本放到 GitLab 应用目录中（或者任意你方便管理的地方），举个例子： `/data/gitlab/gitlab_ssh.sh`。

最后将脚本添加到 `crontab` 中，以10分钟为粒度执行 (结合自己情况进行调整)。

```bash
echo "*/10 * * * * root /data/gitlab/gitlab_ssh.sh" >>/etc/crontab
```

不出意外，往后如果还有这类扫描器，他们最多只能扑腾个10分钟左右。

至于这个脚本的战绩，可以通过查看 `/data/gitlab/logs/bad_gay_22_port.log` 日志文件来进行了解：

```bash
2019-04-10 19:33 : 113.172.217.15
2019-04-10 19:33 : 14.186.202.33
```

### 监控界面登录

前面已经在网络层添加了访问授权，但是如果授权密码泄露，被针对性攻击，比如在界面/应用接口层面进行弱口令扫描，那么又该如何处理呢。

配合 `fail2ban` 可以减少这类事情的影响，下面给出一段[参考脚本](https://gist.github.com/pawilon/238c278d3c6c4669771eb81b03264acd)。

```bash
[Init]
maxlines = 6

[Definition]

# The relevant log file is in /var/log/gitlab/gitlab-rails/production.log
# Note that a single failure can appear in the logs up to 3 times with just one login attempt. Adjust your maxfails accordingly.

## Example fail - clone repo via https
#Started GET "/" for 10.0.0.91 at 2016-10-25 00:01:24 +0200
#Processing by RootController#index as HTML
#Completed 401 Unauthorized in 69ms (ActiveRecord: 23.7ms)

## Example fail - login via GUI
#Started GET "//chmielu/test.git/info/refs?service=git-upload-pack" for 10.0.0.91 at 2016-10-25 00:01:09 +0200
#Processing by Projects::GitHttpController#info_refs as */*
#  Parameters: {"service"=>"git-upload-pack", "namespace_id"=>"chmielu", "project_id"=>"test.git"}
#Filter chain halted as :authenticate_user rendered or redirected
#Completed 401 Unauthorized in 50ms (Views: 0.8ms | ActiveRecord: 8.1ms)


failregex = ^Started .* for <HOST> at .*<SKIPLINES>Completed 401 Unauthorized

ignoreregex =
```

## 用户层

用户层面其实问题不多，如果你能确定你可以坚持使用以下措施的话：

- 关注官方版本更新和 `changelog`，及时更新应用版本，减少 `XSS` 、`CVE` 漏洞问题。
- 进行最小权限授予，减少错误授权带来的风险。
	- 在系统设置中设置所有项目都是 `private` 的，避免某云平台的事故重演。
	- 避免添加过多的全局 `Admin` 角色，针对项目群组和项目进行管理员设置。
- 仅允许使用 `SSH` 方式进行代码 `Clone` 和 `Push`，推荐使用秘钥认证的方式进行系统交互。
- 尽可能减少与外部系统的交互，比如导入外部仓库，仅支持你觉得必要的来源；比如服务调用，仅调用你觉得安全可靠的。
- 关闭默认注册方式，使用邀请制度，或者使用 SSO/LDAP 方式进行注册。
- 根据实际情况进行用户频率限制（系统功能）。
- **要求你的用户使用随机生成的强密码，并定期更换。**

## 最后

使用容器在公网环境搭建 GitLab 就先介绍到这里，性能监控部分，等把 WordPress 的坑填完，再细聊吧。