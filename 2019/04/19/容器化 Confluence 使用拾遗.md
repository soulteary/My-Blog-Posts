# 容器化 Confluence 使用拾遗

之前介绍过使用容器搭建 [Confluence](https://soulteary.com/2019/03/30/construct-confluence-with-docker.html) 作为团队协同工具使用。在使用过程中，我们遇到了一些问题，比如文章时间展示不正确、中文内容无法显示、运行资源不足。

下面就来讲讲在容器场景下，怎么解决简单快速的这些问题。

## 解决文章时间戳不正确

默认 `Confluence` 使用的是东一区（零时区）的时间制式，想解决时区问题，需要先在 `environment` 字段内指定 `CATALINA_OPTS` 参数内容。

```yaml
environment:
    - 'CATALINA_OPTS= -Duser.timezone=GMT+08'
```

另外，为了避免容器和宿主机时间不一致，可以将本机的 `localtime` 挂载到容器中。

```yaml
volumes:
  - /etc/localtime:/etc/localtime:ro
```

## 解决应用卡顿

之前的完整配置将会使用 `Confluence` 默认资源运行服务，程序最大使用内存是 `1GB`，当团队人数和内容多了之后，由于资源不足，会让服务运行变慢，最简单的解决方案就是增加资源。只需要在 `environment` 字段内声明下面内容即可，举个例子，我们可以提高他使用的内存资源为 `4~8 GB`。

```yaml
environment:
    - 'JVM_MINIMUM_MEMORY=4096m'
    - 'JVM_MAXIMUM_MEMORY=8192m'
```

## 解决中文文档不能预览

由于默认容器镜像不包含中文字体，当我们想预览一个中文文档的时候，得到的结果会是一堆“口口口”方块。

解决这个问题的第一步是为镜像系统安装中文字体，下载一些中文字体 ( ttf/ttc )，比如宋体、楷体，将文件命名为：`simsun.ttf`、`simkai.ttc`，然后保存在 `fonts` 文件夹中，然后挂载到容器系统中。

```yaml
volumes:
  - ./fonts:/usr/local/share/fonts
```

接着在参数中添加转换参数：

```yaml
environment:
  - 'CATALINA_OPTS= -Duser.timezone=GMT+08 -Dconfluence.document.conversion.fontpath=/usr/local/share/fonts/ '
```

如果你之前没有预览过中文文档，现在重启应用，问题就解决了。

如果你之前已经预览过中文文档，发现重启应用，预览问题依然如故，那么可以通过清除预览缓存来解决问题。

在之前的配置中，我们将应用数据挂载到了本地。

```yaml
volumes:
  - ./data:/var/atlassian/application-data/confluence
```

通过清空下列目录中的缓存内容，可以即时解决问题。

```yaml
rm -rf ./data/shared-home/dcl-document/*
rm -rf ./data/shared-home/dcl-document_hd/*
rm -rf ./data/shared-home/dcl-thumbnail/*
```

## 最后

先聊到这里吧。

--EOF