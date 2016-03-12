# 迁移 Nexus 软件仓库拾遗

国庆前，我把之前老设备上面运行的服务进行了完整的迁移。但是在迁移代码仓库的过程中，发现有一些小细节挺有意思的。

## 启动新的仓库服务实例

因为我的所有服务配置都保存在了 CVS 系统中，所以在新的机器上启动服务，只需要 `Git Clone` 、 `docker-compose up` 两步走。

然后在 `DNSMASQ` 中将解析从老设备指向到新设备即可，比如：

```bash
address=/docker.lab.com/10.11.12.232
address=/npm.lab.com/10.11.12.232
address=/nexus.lab.com/10.11.12.232
```

## 进行仓库数据迁移

之前我保存持久化数据使用的是 `Mount` 方式，这样做的好处是几乎不存在执行 `docker prune` 等操作误伤有用数据的情况。

但是缺点也明显，简单的打压缩包，进行数据迁移对于这种场景不好使。

这里有一个简单的方案：将之前保存在老仓库的软件包进行批量下载，然后重新推送到新的仓库。

### 转移 Docker 镜像

使用 `docker pull` 把要转移的镜像下载完毕后，使用 `docker images` 命令，配合 `grep` 以及 `awk` 可以快速的筛选出需要进行转移的镜像名称和版本。

```bash
$ docker images | grep 'docker.lab.com' | grep -v '<none>' | awk '{printf("%s:%s\n", $1, $2)}'
docker.lab.com/hugo-post-adapter:1.0.0-convert
docker.lab.com/aria2:1.0.0
docker.lab.com/pushover.lab.com:0.0.1
docker.lab.com/sonar.lab.com:7.3
...
```

然后使用 `xargs` 将这些镜像逐个推送到新仓库即可。

```bash
$ docker images | grep 'docker.lab.com' | grep -v '<none>' | awk '{printf("%s:%s\n", $1, $2)}' | xargs -I {} docker push {}
The push refers to repository [docker.lab.com/hugo-post-adapter]
d48cccfd71ab: Pushed 
d04f8a556ca9: Pushed 
e58056ca81cc: Pushed 
984705602673: Pushed 
6a2901185647: Pushed 
2d4ed44f6fa7: Pushed 
35e23a957234: Pushed 
894a6015dedf: Pushed 
2d790067a9f7: Pushed 
32b5d1364b1c: Pushed 
fbebd655f9aa: Pushed 
90e156c66608: Pushed 
a5a2fa193409: Pushed 
bc912d07a289: Pushed 
905f2907ec29: Pushed 
73046094a9b8: Pushed 
1.0.0-convert: digest: sha256:fe391b4bcd2f2af0373b969f68f3f56a0ca1ac097257160c4abf135d31d39a7d size: 3668
The push refers to repository [docker.lab.com/aria2]
165e36b92662: Pushed 
a1da261fbc00: Pushed 
5e8644249a98: Pushed 
```

当然，如果你要全面进行备份，而不是简单的重新推送镜像到新仓库的话，可以用下面的脚本：

```bash
docker images | tail -n +2 | grep -v "<none>" | awk '{printf("%s:%s\n", $1, $2)}' | while read IMAGE; do
    echo "find image: $IMAGE"
    filename="$(echo $IMAGE| tr ':' '-' | tr '/' '-').tar"
    echo "save as $filename"
    docker save ${IMAGE} -o $filename
done
```

### 转移 NPM 包

这个就更简单了，使用 `git checkout releaseTag` ，然后重新 `npm pub` 就完事了。

## Nexus 3.3.x 镜像权限问题

如果你使用了 Nexus 3.3.x 版本的镜像，并且将数据映射到了外部文件系统，大概率会遇到下面的报错。

```bash
mkdir: cannot create directory '../sonatype-work/nexus3/log': Permission denied

mkdir: cannot create directory '../sonatype-work/nexus3/tmp': Permission denied

Java HotSpot(TM) 64-Bit Server VM warning: Cannot open file ../sonatype-work/nexus3/log/jvm.log due to No such file or directory
```

解决方法很简单，在你映射的外部文件系统中执行下面的命令，将目录权限归属设定为 `200`。

```bash
sudo chown -R 200 ~/dockerVolume/nexus
```

## 最后

Nexus 相比一些后起之秀而言，管理界面可能相对简单了一些，但是不论是资源占用，还是软件整体的稳定性来说，都十分优秀。

在我使用了快一年的时间里，在上面积累了接近 5G 的代码包(hosted+proxy)和镜像文件(hosted)，迁移时发现产生的磁盘碎片仅有100M不到，而且支持集群模式，以及市面上几乎所有的语言仓库协议，具备定时任务…

如果你或者你的团队在使用 CI/CD 进行敏捷开发，但是又缺乏一个稳定的内部仓库，可以试试这款不错的软件。

如果你有意向了解或者学习 Nexus 的搭建和使用，或许我可以写一篇详细的从搭建到使用。

—EOF