# 清理 OSX 系统中的 Docker 容器、镜像与数据卷

说实话一直以来都很信任Docker的客户端，毕竟使用这么久只遇到过三次问题：

1. 低版本几年前安装在服务器上出现过碎文件占用完所有的iNodes，`df -i` 发现是 100% IUse。
2. 早先从 `boot2docker` 升级到 `Docker For Mac` 出现过程序进程死掉，然后所有储存数据都消失，但是磁盘占用不减少的问题。
3. 这次在 `v18.0X` 发现存在清理各种数据之后，磁盘占用不减少的问题。

## 解决方法

导出所有需要的自建镜像，并重置 `Docker` 保存数据，可以适当更新客户端到最新的稳定版本。

## 解决过程

下面来聊聊解决过程。

**由于我已经全部重置，所以只演示示意流程。**

## 望闻问切

在 OSX 系统可以使用 `du` 和 `df` 确认磁盘（包含子文件）占用和 iNodes 占用。

```bash
df -ih ~/Library/Containers/com.docker.docker/

Filesystem     Size   Used  Avail Capacity iused               ifree %iused  Mounted on
/dev/disk1s1  466Gi  426Gi   36Gi    93% 3286807 9223372036851489000    0%   /

du -d 1 -h ~/Library/Containers/com.docker.docker/ 
4.4G	/Users/soulteary/Library/Containers/com.docker.docker//Data
4.4G	/Users/soulteary/Library/Containers/com.docker.docker/
```

如果发现你明明干掉了不必要的数据卷和无用镜像以及废弃容器，磁盘占用空间还是很大（我之前是几十G）；或者明明硬盘还没用满，但是由于 `iNodes` 用尽导致无法写入文件了，那么应该给 `Docker` 看病的时机就到了。

### 头疼医头 - 高阶命令

表象是磁盘资源被过度占用，所以治理表象需要把资源释放掉。

下面给出几种不同的方法，具体哪种可用，因版本而异，请自取辨别。

`Docker` 1.13之后，其实官方支持了更高级的打扫命令：`docker system prune`。

默认使用会告知你它的工作模式是删除所有停止运行的容器，所有单容器使用的虚拟网卡，所有无名称的镜像，所有构建过程的缓存内容。

```bash
docker system prune
WARNING! This will remove:
        - all stopped containers
        - all networks not used by at least one container
        - all dangling images
        - all build cache
Are you sure you want to continue? [y/N] 
```

但是如果你想清理数据卷的话，可以这样用：

```bash
docker system prune --volume
```

如果你想清理掉没有使用的镜像，而并非那些讨厌的 `<None>` 镜像的话，可以这样用：

```bash
docker system prune --all
```

### 头疼医头 - 基础命令

如果你担心上面的方法搞不定或者无法使用 `system` 命令解决问题，那么可以试试直接调用它的实现方法。

清理不用的容器：

```bash
docker container prune
# 或者可以带参数清理停止运行一天以上
docker container prune until=24h
```

使用 `volume`、`network`、`container`、`image` 对数据卷、虚拟网卡、容器、镜像进行清理，注意这里这些命令都支持 `filter` 参数，支持使用以下表达式进行高效过滤：

**label=<key>, label=<key>=<value>, label!=<key>, or label!=<key>=<value>**

例如这样使用：

- 'exited=0'
- 'until=24h'
- 'statue!=running'

为了表示简洁，下面以 `$COMMAND` 代替上面的具体命令，实际使用替换回来。

```bash
docker $COMMAND prune
# 清理所有内容
docker $COMMAND prune --all
# 或者给定条件进行清除
docker $COMMAND prune --filter <exp>
```

### 头疼医头 - 低阶命令

如果上面的命令你还是使用起来有问题，没关系，可以使用更低阶的版本，我个人使用的alias中，常年存放了两条命令：

```bash
cat ~/.alias/docker.sh 
#!/usr/bin/env bash

# 清除无用的镜像
alias give-me-remove-docker-unused-images='docker rmi $(docker images -f "dangling=true" -q)'
# 清理无用容器
alias give-me-remove-docker-unused-container='docker rm $(docker ps --filter status=exited -q)'
```

没错，使用 `docker rm` 和 `docker rmi` 命令可以直接对容器和镜像进行清理，不过对于虚拟网卡和数据卷就不适用了，不过你可以配合系统的 `rm` 删除掉其他数据：

```bash
rm -rf ~/Library/Containers/com.docker.docker/Data
```

### 头疼医头 - 界面重置

![界面大法](
https://attachment.soulteary.com/2018/07/19/reset-docker-for-mac.png)

当然，我试验过最近两个版本的 `Docker For Mac` 使用界面按钮 `Reset to factory default` 重置数据也是极其有效的。

### 治标治本


尽己所能的升级你的客户端版本，新版本会包含老版本的 Bugfix，并且关注以下两个问题反馈官方的处理结果，毕竟这个实际是文件储存系统的Bug，暂时真的是彻底解决不能，不然我也不会给出三个指标不治本的方法了，你说是不。

```bash
https://github.com/docker/for-mac/issues/371
https://github.com/moby/moby/issues/21925
```

## 最后

欢迎讨论斧正本文内容，当然，如果官方解决了这个问题，我也会同时更新文章修正内容。

--EOF

