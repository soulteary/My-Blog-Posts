# 群晖系统上的 Docker 使用拾遗

这是一篇在草稿箱里积灰许久的文章，没有发布的原因是之前一直在犹豫要不要配一些例子。前一阵基于 traefik 折腾了一套新的最佳实践，跑了一个多季度，看起来一切都还不错，所以本篇内容就可以拆分成几个小点单独进行发布了。

说起群晖的 Docker ，其实轻度使用的时候，跑个带端口映射的容器还真不错，但是在多个应用进行编排使用的时候，使用的时候还是有一些问题。

## Docker 和 compose 版本落后

我们可以从 `https://www.synology.cn/zh-cn/releaseNote/Docker` 查看到当前支持的 Docker 版本，以及更新记录。

但是一般来说群晖的版本会比 Docker 官方社区低起码一个大版本，有一段时间，DSM 上跑的容器都只能是 `version 2` 的 docker-compose 配置，缺少一堆指令的支持，直到升级到了 1.1.4 版本的 compose。

查看官方的版本发布记录，不难看出 1.1.4 也是去年的老版本了，不过好在支持到 3.0+ 了，多数指令也能使用了，只要**不涉及复杂组网**。

```plain
1.14.0 (2017-06-19). New features. Compose file version 3.3. Introduced version 3.3 of the docker-compose.yml specification. This version requires to be used ...
```

这里在实际使用的时候，设置为 3 即可，让编排功能随着编排工具滚动升级。

## 手动升降 Docker 版本

可能出于对资源占用的敏感，你会想手动选择一个 Docker 版本。

通过翻看群晖应用的管理代码，不难看到群晖软件包的真实下载地址：

```plain
https://usdl.synology.com/download/Package/spk/Docker/
```

将你想安装的版本下载下来之后，配合软件仓库的手动安装功能，就能够实现软件的升降级了（推荐对之前的软件先进行卸载处理）。

另外，如果你想试试交叉编译出最新版本的 Docker ，可以试试下面的工具：

```plain
https://github.com/SynoCommunity/spksrc
```

## 被替换的日志驱动

在不指定 `log` 使用 `docker-compose` 启动程序的时候，你会发现新版本 `stdout` 原本应该存在的输出内容是空白的，取而代之的是一行警告信息：

```plain
WARNING: no logs are available with the 'db' log driver
```

而这里提到的 `db` 日志驱动，是为了让我们能够在群晖 Web 管理界面中看到日志而存在的一个程序，如果我们要将日志还原为原本正确的输出行为中，该怎么做呢？

修改 `/usr/syno/etc/packages/Docker/dockerd.json` 文件，指定日志驱动的类型为 `json-file` ，然后重启 Docker 即可。


```json
{
   "log-driver": "json-file"
}
```

## 加速容器获取

同上面的操作一样，还是修改 `dockerd.json` 文件，不过这里要添加的是 `registry-mirrors` 这个 key，如果我们既要修改日志输出方式，又要使用三方的仓库镜像，可以这样配置。

```json
{
   "registry-mirrors" : [ "http://your-3th-party-mirror" ],
   "log-driver": "json-file"
}
```

如果你信任了你的自签证书的话，那么这里是可以直接配置成你的个人仓库镜像的。

## 其他

或许应该写个小脚本，提供一键修改群晖应用配置的功能。

--EOF
