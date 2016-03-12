# 远程管理 Mac OSX 设备

开发设备一般开放 `SSH` 端口和访问权限就能够满足几乎所有需求，但是如果设备换成了 Windows 或者 OSX ，这个事情就没有那么简单了，偶尔还是需要登录图形界面做一些事情。对于我这种家里局域网有完整开发环境的折腾控，远程管理 OSX 设备的需求就出现了。

如果你使用类似 AnyConnect 的方式去连接专有网络进行开发，这个事情可以相对容易的解决，但是会额外带来一个问题：你的所有流量都会走目标设备。

而如果使用本地端口转发、系统PROXY的方案来做，有些客户端又支持的不是很好，比如我一直在使用的 Remotix ，我购买这款软件有好几年了，它能够让我在笔记本、平板、甚至是手机上解决一些问题，但是一旦我离开局域网环境，它的“假期就开始了”。

本文将花费十五分钟的时间，解决这个问题。

## 反向代理指定应用端口

相比较全部流量都走目标设备，这个方案明显更合理。当然，你也可以选择配置路由表解决上面的问题，不过如果目标设备有多台，分布于多个位置，难道我们要不停切换网络连接状态吗。

这里用到的技术方案主要是：流量反向代理。

我选择的工具是 [https://github.com/fatedier/frp](https://github.com/fatedier/frp)，选择它的原因很简单，它的代码开源。

这个方案需要一台位于公网的服务器，前一阵清理服务，正好空闲了两台，拿来做这个事情再适合不过了。

### 配置服务端

这里的服务端是具备公网 IP 地址的云服务器，用来反向代理你在局域网中需要被访问的设备。

以 `Ubuntu 18.04` 为例，我们进行服务端的配置。访问 [https://github.com/fatedier/frp/releases](https://github.com/fatedier/frp/releases) 获取最新的软件包。

```bash
cd /tmp
wget https://github.com/fatedier/frp/releases/download/v0.24.1/frp_0.24.1_linux_amd64.tar.gz
tar -zxvf frp_0.24.1_linux_amd64.tar.gz
```

解压缩之后，你会看到一堆文件，其中名称为 `frps` 的文件是和服务器相关的，而命名为 `frpc` 则是客户端相关的文件。

我们先对应用目录进行规划，理想的目录环境可以是这样：

```bash
.
├── conf # 存放配置文件
│   └── supervisor.ini
├── frps # 服务端应用
├── frps.ini
└── log # 存放日志文件
    ├── frps.log
    └── supervisor.log
```

创建必要的目录，并将之前下载好的应用执行文件放到这个目录中。

```bash
mkdir -p /data/frp/{conf,log}
mv /tmp/frp_0.24.1_linux_amd64/frps /data/frp
```

接着我们来创建一个服务端配置文件，可以参考下面的配置，并适当修改内容。

```bash
[common]
bind_port = 8000
bind_addr = 0.0.0.0

dashboard_addr = 0.0.0.0
dashboard_addr = 8080
dashboard_user = homelab_user
dashboard_pwd  = homelab_pass

token = token_used_by_server_and_client

max_pool_count = 5
max_ports_per_client = 0
tcp_mux = true
```

如果你将上面的配置保存为 `frps.ini`，接下来就可以执行命令验证配置是否正确。

```bash
/data/frp/frps -c /data/frp/frps.ini
```

不出意外，你将看到下面的提示：

```TeXT
2019/03/11 12:30:45 [I] [service.go:124] frps tcp listen on 0.0.0.0:8000
2019/03/11 12:30:45 [I] [root.go:204] Start frps success
```

这里需要额外注意一件事，如果你使用的云服务开启了`安全组`功能，需要额外配置安全组的规则，对上述配置文件的端口进行放行，如果开启了 `ufw` 这类防火墙，也是同理。

为了服务的稳定运行，这里我们还是使用 `supervisor` 进行进程管理和运行守护，如果有不会配置的同学可以翻看以往的文章或者自行搜索。

这里给出一个基础的配置参考：

```TeXT
[program:frp]
; 启动目录
directory = /data/frp/
; 执行命令
command = /data/frp/frps -c /data/frp/frps.ini
; 随 supervisord 启动
autostart = true
; 程序启动 5s 内没有异常则认为是正常运行
startsecs = 10
; 程序异常退出后重新启动
autorestart = true
; 重试启动程序多少次
startretries = 1000
; 默认使用当前用户执行应用
;user = soulteary
; 需要手动创建目录
stdout_logfile = /data/frp/log/supervisor.log
```

当然，别忘记重启 `supervisor` 让配置生效，具体操作上一篇文章中有讲。

### 配置客户端

这里的客户端是指你需要被访问的设备，或者使用端口转发规则能够访问到目标设备的路由器、交换机设备，本文中的客户端指的是一台 Mac Book Pro 笔记本。

和配置服务端类似，我们先下载软件包，并准备应用执行目录。

```TeXT
wget https://github.com/fatedier/frp/releases/download/v0.24.1/frp_0.24.1_darwin_amd64.tar.gz
tar -zxvf frp_0.24.1_darwin_amd64.tar.gz

mkdir -p ~/Service/frp/{conf,log}
mv frp_0.24.1_darwin_amd64/frpc ~/Service/frp/
```

接着创建客户端的配置文件，这里暂定命名为 `frpc.ini`，具体内容请根据具体情况修改。

```TeXT
[common]
server_addr = 123.123.123.123
server_port = 8000
token = token_used_by_server_and_client

admin_port = 8080
admin_user = homelab_client
admin_pwd  = homelab_client

pool_count = 5
tcp_mux = true

[vnc]
type = tcp
local_ip = 127.0.0.1
local_port = 5900
use_encryption = true
use_compression = trye
remote_port = 5900
health_check_type = tcp
health_check_timeout_s = 3
health_check_max_failed = 3
health_check_interval_s = 10
```

这里需要额外注意的是：**token** 字段，内容务必和服务端保持一致，否则无法创建连接。

另外，你希望能够被远程访问到的服务，它的 `remote_port` 同样需要服务端开放此端口的访问规则。

接着，我们运行下面的命令，进行测试，看看服务是否联通。

```bash
~/Service/frp/frpc -c ~/Service/frp/frpc.ini
```

如果一切正常，你将能够看到类似下面的日志：

```TeXT
2019/03/11 12:36:00 [I] [service.go:214] login to server success, get run id [4a62ac3f5f635268], server udp port [0]
2019/03/11 12:36:00 [I] [proxy_manager.go:137] [2a62ac3f5f635268] proxy added: [rdp]
2019/03/11 12:36:00 [I] [health.go:115] [2a62ac3f5f635268] [rdp] health check status change to success
2019/03/11 12:36:00 [I] [proxy_wrapper.go:206] [2a62ac3f5f635268] [rdp] health check success
2019/03/11 12:36:00 [I] [control.go:143] [vnc] start proxy success
```

这时，打开客户端软件，创建一个 VNC 链接，服务器地址填写云主机的IP，端口填写客户端服务 `remote_port` ，如果上述配置都正常，那么你将会看到远程桌面的登录提示。

![Mac OSX 登录提示](https://attachment.soulteary.com/2019/03/11/mac-osx-vnc-prompt.png)

输入正确的用户名和密码之后，熟悉的桌面就呈现在你的眼前了。

![Mac OSX 远程桌面](https://attachment.soulteary.com/2019/03/11/mac-osx-vnc-desktop.png)

Mac OSX 系统上的进程管理，上一篇文章已经提到过，这里不做赘述，简单提供一个 `supervisor` 配置。

```TeXT
rogram:frp]
; 启动目录
directory = /Users/soulteary/Service/frp/
; 执行命令
command = /Users/soulteary/Service/frp/frpc -c /Users/soulteary/Service/frp/frpc.ini
; 随 supervisord 启动
autostart = true
; 程序启动 5s 内没有异常则认为是正常运行
startsecs = 10
; 程序异常退出后重新启动
autorestart = true
; 重试启动程序多少次
startretries = 1000
; 默认使用当前用户执行应用
;user = soulteary
; 需要手动创建目录
stdout_logfile = /Users/soulteary/Service/frp/log/supervisor.log
```

## 最后

长时间这样暴露端口，其实并不是理智 & 安全的选择，为了更加安全的使用这个服务，我们还需要做一些额外的策略，后续我会再更新一篇，详细介绍如何处理。
