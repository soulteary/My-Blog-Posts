# Mac OSX 开机启动应用

Mac OSX 的稳定性一向不错，不论是用于跑基于 XCode 的 CI，还是用于做简单的 `HomeLab` 使用，都能满足需求。

但是在日程使用中，难免出现系统维护需要重启、应用遇到问题崩溃退出运行，这个时候，就不得不引用进程管理工具来帮助我们进行应用的开机启动或者进程退出重启了。

本篇文章将花十分钟左右介绍如何简单优雅的对 Mac 进行程序开机启动、运行保护，先从开机启动聊起。

## 关于 Mac OSX 系统的开机启动

说起 Mac 的开机启动，通常绕不开 `LaunchDaemons`、`LaunchAgent`、`Login Item`、`Startup Items`、`AppHelper` 等关键词。

凡是折腾过应用开机启动的同学，一定对上面的关键词很清楚，手写 `plst` 是基础要求，如果想用户不登录自动启动应用，可能还需要小写一段代码，编译成应用的 `Helper`。

有没有既好用，又不必这么**硬核**的应用启动方案呢？

## 使用 Supervisor

这里依旧推荐 [Supervisor](http://supervisord.org/)，早些时候，我曾不止一次的推荐过这款开源应用，它不光活跃在各大互联网公司的生产环境中，还能够运行在各种开发、线下环境中，并且会主动适配各种环境。

### 快速安装 Supervisor

在 Mac OSX 环境中，如果想快捷安装它，需要先安装 `brew` （一句话安装）。

```bash
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

安装完毕之后，执行下面的命令，便可以安装好 `supervisor` 以及配套的 `services` 应用。

```bash
brew install supervisor
brew install services
```

安装完毕之后，只需要执行下面的命令便可以启动 `supervisor`。

```bash
brew services start supervisor
```

如果进程已经执行，你将会受到下面的提示，可以忽略掉。

```TeXT
Service `supervisor` already started, use `brew services restart supervisor` to restart.
```

如果你想关闭 `supervisor`，只需要将上面的参数替换为 `stop` 即可。

```TeXT
brew services stop supervisor
```

### 配置 Supervisor

安装完毕之后，我们的默认配置会保存在下面的位置。

```TeXT
/usr/local/etc/supervisord.ini
```

虽然我们最常使用命令行的方式对 `supervisor` 进行操作，但是它同样提供了一个更简单的方式进行进程的管理和查看：Web 界面。

浏览上面的配置文件，你将会看到这么一段被注释掉的配置。

```Toml
;[inet_http_server]         ; inet (TCP) server disabled by default
;port=127.0.0.1:9001        ; ip_address:port specifier, *:port for all iface
;username=user              ; default is no username (open server)
;password=123               ; default is no password (open server)
```

如果不在公网开放，仅供本机进行调试维护，可以将头两行配置前的注释符号 `;` 去掉，变成下面的样子：

```Toml
[inet_http_server]         ; inet (TCP) server disabled by default
port=127.0.0.1:9001        ; ip_address:port specifier, *:port for all iface
;username=user              ; default is no username (open server)
;password=123               ; default is no password (open server)
```

接着执行下面的命令，重启 `supervisor` ，让配置生效。

```TeXT
brew services reload supervisor
```

验证 HTTP 管理界面的方式很简单，打开浏览器，访问 `http://127.0.0.1:9001` 即可，或者使用 `lsof` 命令也可以快速验证服务是否正常，如果你也看到类似下面的命令行输出，那么服务就很轻松愉快的跑起来了。

```TeXT
soulteary in ~ λ lsof -i:9001
COMMAND    PID      USER   FD   TYPE             DEVICE SIZE/OFF NODE NAME
Google    5388 soulteary   34u  IPv4 0x2fed8630edccdf51      0t0  TCP localhost:49785->localhost:etlservicemgr (ESTABLISHED)
Google    5388 soulteary   45u  IPv4 0x2fed8630ee723c51      0t0  TCP localhost:49786->localhost:etlservicemgr (ESTABLISHED)
Google    5388 soulteary   50u  IPv4 0x2fed8630ee5165d1      0t0  TCP localhost:49787->localhost:etlservicemgr (ESTABLISHED)
python2.7 7593 soulteary    5u  IPv4 0x2fed8630edccf251      0t0  TCP localhost:etlservicemgr (LISTEN)
python2.7 7593 soulteary    8u  IPv4 0x2fed8630edccd5d1      0t0  TCP localhost:etlservicemgr->localhost:49785 (ESTABLISHED)
python2.7 7593 soulteary    9u  IPv4 0x2fed8630ec024c51      0t0  TCP localhost:etlservicemgr->localhost:49786 (ESTABLISHED)
python2.7 7593 soulteary   10u  IPv4 0x2fed8630ec027251      0t0  TCP localhost:etlservicemgr->localhost:49787 (ESTABLISHED)
```

如果此刻你重启设备，等待设备重启完成，除了使用 `ps -ef` 检查 `supervisor` 是否运行之外，使用浏览器打开 `127.0.0.1:9001` 也是一样的。

![Web Dashboard 界面](https://attachment.soulteary.com/2019/03/11/web-dashboard.png)

日志是应用执行过程中宝贵的财富，所以要将 `supervisor` 的日志好好保存，默认配置中，日志会保存在下面的地方，根据自己的情况将日志挪到便于管理的地方吧，如果做了修改，也别忘记要重启进程，方法上面交代过了。

```Toml
[supervisord]
logfile=/usr/local/var/log/supervisord.log ; main log file; default $CWD/supervisord.log
```

继续浏览配置文件，不难发现 `Supervisor` 默认会自动读取 `/usr/local/etc/supervisor.d/` 目录下所有的 `ini` 文件。

```Toml
[include]
files = /usr/local/etc/supervisor.d/*.ini
```

考虑到可维护性，我们要对它进行一些修改，将启动配置“就近保存”在需要被启动的应用的目录内，比如 `~/Service/example` 中，比如这样。

```Toml
[include]
files = /Users/soulteary/Service/*/conf/supervisor.ini
```

### 进行应用验证

接下来我们将编写一个简单的脚本来对 `supervisor` 进行进程启动和守护的能力验证。

先准备一个项目目录。

```TeXT
mkdir -p ~/Service/example
```

在目录中创建一个名为 `test.sh` 的脚本，内容如下：

```bash
#!/usr/bin/env bash

python -m SimpleHTTPServer 8080
```

赋予脚本执行权限之后，我们可以运行这个脚本。

```bash
chmod +x test.sh
./test.sh
```

当你打开浏览器，访问 `127.0.0.1:8080`，你将会看到一个使用 Python 运行的 Web Server。

![Web Server 界面](https://attachment.soulteary.com/2019/03/11/py-dashboard.png)

在项目目录中创建一个 `conf` 文件夹，用来存放 `supervisor` 程序的配置文件。

```bash
mkdir conf
touch conf/supervisor.ini
```

配置文件内容示例：

```bash
[program:test]
; 启动目录
directory = /Users/soulteary/Service/example/
; 执行命令
command = /Users/soulteary/Service/example/test.sh
; 随 supervisord 启动
autostart = true
; 程序启动 5s 内没有异常则认为是正常运行
startsecs = 5
; 程序异常退出后重新启动
autorestart = true
; 重试启动程序多少次
startretries = 3
; 默认使用当前用户执行应用
user = soulteary
; 需要手动创建目录
stdout_logfile = /Users/soulteary/Service/log/supervisor.log
```

重启 `supervisor` 后，使用浏览器或者 `ps` 验证，Web Server 运行了起来，说明 `supervisor` 是能够启动进程的。

在验证 `supervisor` 是否能够守护进程，需要关闭当前 Web Server 进程，先使用命令行获取当前进程 PID 。

```bash
ps -ef | grep test.sh | grep -v grep | awk '{print $2}'
```

举个例子，此处 PID 显示 1529 ，接下来我们执行下面的命令，进行进程杀除，此时浏览器中的站点应该已经打不开了。

```bash
ps -ef | grep test.sh | grep -v grep | awk '{print $2}' | xargs -I {} kill {}
```

稍等片刻，刷新浏览器，Web 服务又被启动了起来，再次执行第一条命令，可以看到进程 PID 已经变化，说明 `supervisor` 已经配置完毕。

当然，`supervisor` 也提供了一套简单好用的命令给用户使用，比如可以使用下面的命令查看当前需要被守护的应用的运行状态：

```bash
supervisorctl status
test                             RUNNING   pid 1629, uptime 0:04:21
```

更多命令，请自行查询文档，这里就不赘述了。当然，你也可以从 Web 界面查看当前应用是否成功运行。

![使用 Web Status 查看进程状态](https://attachment.soulteary.com/2019/03/11/web-status.png)


## 最后

如果你对本文描述内容不是很熟悉，可以翻阅以往的文章，补全对上述技术的认识，希望我的文章对你有所帮助。

— EOF