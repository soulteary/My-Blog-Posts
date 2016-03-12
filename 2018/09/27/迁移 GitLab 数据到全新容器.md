# 迁移 GitLab 数据到全新容器

本篇文章可以看做是全新搭建 GitLab 的教程使用。

我个人使用方式是使用虚拟机软件强制将 `4核心4GB` 的运行环境划分给 `GitLab` 等同购买使用等配置的云主机，这样做可以避免和主机上其他软件进行资源争抢，影响运行效率。

从 13 年到现在，这已经是第3次迁移 GitLab 了，之前的迁移主要原因：

- 将低版本跨大版本升级至高版本
- 将裸机运行的软件迁移至容器中

而这次的迁移主要原因：

- 更换了性能更好的新硬件
- 测试备份包是否能够正常工作
- 减少磁盘碎片对于宿主机的资源浪费（实际使用20G以内，磁盘占用已经100多G）

在开始搭建应用和迁移数据之前，我们需要做一些准备工作。

## 准备工作

虽然我在对外的网站上使用了最新的 `Ubuntu 18.04` 发行版，但是考虑最大程度的稳定性，我还是选择了 `16.04` ，毕竟两年多的使用，几乎没有遇到问题，而且和 `docker-ce` 兼容性的问题，也解决了大半，省心省力。

### 配置操作系统

虚拟机默认安装完毕的操作系统，是无法使用 `SSH` 进行远程访问的。需要先安装 `openssh-server` 以便开启访问服务。

```bash
apt update && apt install -y openssh-server
```

程序安装完毕之后，就可以直接使用 `SSH` 连接并管理服务器了。

我这里为了后面的操作简单，将新机器在本地 `SSH CONFIG` 中配置为了 `gitlab-2018.lab.com` ，然后做了 `RSA KEY` 授信免登陆。

```bash
ssh-copy-id -i SOME—KEY.pub soulteary@gitlab-2018.lab.com
```

清理一下无用的软件源和注释。

```bash
cat /etc/apt/sources.list | grep -v '#' | grep -v 'cdrom:' | sudo tee /etc/apt/sources.list
```

如果在国内，希望加速软件下载，可以使用下面的命令。

```bash
cat /etc/apt/sources.list | sed -e "s/us\.archive\.ubuntu\.com/mirrors\.aliyun\.com/" | sed -e "s/security\.ubuntu\.com/mirrors\.aliyun\.com/" | sudo tee /etc/apt/sources.list
```

然后更新软件包，下载一些基础依赖和工具。

```bash
apt install -y apt-transport-https  ca-certificates curl software-properties-common
apt upgrade -y
```

当然，如果要安装 `docker-ce` ，推荐使用官方源，可以参考 [之前的文章](https://soulteary.com/2018/08/28/better-use-of-docker-and-traefik.html) 。我这里为了安装快速，就先使用国内阿里云的源了。

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
apt-key fingerprint 0EBFCD88

add-apt-repository "deb [arch=amd64] https://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
apt install -y docker-ce
```

这里可以根据自己情况，考虑是否要锁定当前的 `GitLab` 软件版本。

```bash
apt-mark docker-ce
```

编排工具，使用官方出品的 `compose` 即可，简单够用。

```bash
curl -L https://github.com/docker/compose/releases/download/1.22.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

考虑到 `GitLab` 会占用 `22` 端口，我们先行将系统的端口进行修改。

```bash
cat /etc/ssh/sshd_config | sed -e 's/#Port 22/Port 55555/' | sed -e 's/Port 22/Port 55555/' | sudo tee /etc/ssh/sshd_config 

service ssh restart
```

如果你也配置了 `SSH CONFIG`， 需要一同修改里面的端口记录。

有指定 SHELL 习惯的童鞋，可以顺手配置。

```yaml
apt install -y zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

### 修改 Docker 配置

如果觉得国内下载 `Docker` 镜像包比较慢，可以编辑 `/etc/docker/daemon.json` 加速资源下载。

如果你进行了配置修改，想要让配置修改生效，必须重启 `docker` 服务，可以参考下面的命令。

```bash
mkdir -p /etc/docker && touch /etc/docker/daemon.json

cat <<EOF > /etc/docker/daemon.json
{
    "registry-mirrors": [
        "http://你的镜像地址"
    ]
}
EOF

service docker restart
```

### 搭建新的 GitLab 运行容器

下面提供一个简单的文件配置，各位看官可以按自己需求修改。

因为没有用到太多特殊功能，我这里将 `compose` 配置声明为 `v2` 。

```yaml
version: '2'

services:

  gitlab:
    restart: always
    image: 'gitlab/gitlab-ce:11.2.3-ce.0'
    hostname: 'gitlab.lab.com'
    ports:
      - "80:80"
      - "443:443"
      - "22:22"
      - "9005:9999" # Nginx
    # 解决搜索引擎搜索不出小于3字符问题
    # https://gitlab.com/gitlab-org/gitlab-ce/issues/40379
    entrypoint: |
      bash -c 'sed -i "s/MIN_CHARS_FOR_PARTIAL_MATCHING = 3/MIN_CHARS_FOR_PARTIAL_MATCHING = 1/g" /opt/gitlab/embedded/service/gitlab-rails/lib/gitlab/sql/pattern.rb && /assets/wrapper'
    volumes:
      - './config:/etc/gitlab'
      - './logs:/var/log/gitlab'
      - './data:/var/opt/gitlab'
      - './cert/lab.com.crt:/etc/gitlab/ssl/lab.com.crt:ro'
      - './cert/lab.com.key:/etc/gitlab/ssl/lab.com.key:ro'
      - './cert/lab.com.crt:/etc/gitlab/ssl/registry.gitlab.lab.com.crt:ro'
      - './cert/lab.com.key:/etc/gitlab/ssl/registry.gitlab.lab.com.key:ro'
      - './cert/lab.com.crt:/etc/gitlab/ssl/page.lab.com.crt:ro'
      - './cert/lab.com.key:/etc/gitlab/ssl/page.lab.com.key:ro'
      - './cert/lab.com.crt:/etc/gitlab/ssl/pages-nginx.crt:ro'
      - './cert/lab.com.key:/etc/gitlab/ssl/pages-nginx.key:ro'
      - './cert/lab.com.crt:/var/opt/gitlab/registry/certificate.crt:ro'
      - './embedded-logs:/opt/gitlab/embedded/logs/'
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'https://gitlab.lab.com'
        gitlab_rails['time_zone'] = 'Asia/Shanghai'
        gitlab_rails['gitlab_default_projects_features_issues'] = true
        gitlab_rails['gitlab_default_projects_features_merge_requests'] = true
        gitlab_rails['gitlab_default_projects_features_wiki'] = true
        gitlab_rails['gitlab_default_projects_features_snippets'] = true
        gitlab_rails['gitlab_default_projects_features_builds'] = true
        gitlab_rails['gitlab_default_projects_features_container_registry'] = false
        gitlab_rails['lfs_enabled'] = true
        gitlab_rails['registry_enabled'] = false
        prometheus_monitoring['enable'] = false
        node_exporter['enable'] = false
        redis_exporter['enable'] = false
        postgres_exporter['enable'] = false
        gitlab_monitor['enable'] = false
        registry['enable'] = false
        nginx['enable'] = true
        nginx['client_max_body_size'] = '250m'
        nginx['redirect_http_to_https'] = true
        nginx['redirect_http_to_https_port'] = 80
        nginx['ssl_certificate'] = "/etc/gitlab/ssl/lab.com.crt"
        nginx['ssl_certificate_key'] = "/etc/gitlab/ssl/lab.com.key"
        nginx['ssl_ciphers'] = "ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-GCM-SHA256"
        nginx['ssl_prefer_server_ciphers'] = "on"
        nginx['ssl_protocols'] = "TLSv1.2"
        pages_external_url "https://page.lab.com/"
        pages_nginx['ssl_certificate'] = "/etc/gitlab/ssl/pages-nginx.crt"
        pages_nginx['ssl_certificate_key'] = "/etc/gitlab/ssl/pages-nginx.key"
        gitlab_pages['enable'] = true
        gitlab_pages['inplace_chroot'] = true
        gitlab_pages['redirect_http'] = false
        gitlab_pages['use_http2'] = true
        gitlab_pages['dir'] = "/var/opt/gitlab/gitlab-pages"
        gitlab_pages['log_directory'] = "/var/log/gitlab/gitlab-pages"
        gitlab_pages['artifacts_server'] = true
        gitlab_pages['artifacts_server_timeout'] = 10
        nginx['http2_enabled'] = true
        nginx['proxy_set_headers'] = {
          "X-Forwarded-Proto" => "http",
          "CUSTOM_HEADER" => "VALUE"
        }
        nginx['status'] = {
          "listen_addresses" => ["127.0.0.1"],
          "fqdn" => "gitlab.lab.com",
          "port" => 9999,
          "options" => {
            "stub_status" => "on", # Turn on stats
            "access_log" => "on", # Disable logs for stats
            "allow" => "127.0.0.1", # Only allow access from localhost
            "deny" => "all" # Deny access to anyone else
          }
        }

```

将配置文件保存到新的服务器上，执行 `docker-compose up` 后，程序便会自动下载镜像文件，然后自动启动应用。

```bash
Creating network "gitlablabcom_default" with the default driver
Pulling gitlab (gitlab/gitlab-ce:11.2.3-ce.0)...
11.2.3-ce.0: Pulling from gitlab/gitlab-ce
3b37166ec614: Pull complete
3b37166ec614: Pull complete
ba077e1ddb3a: Pull complete
34c83d2bc656: Pull complete
84b69b6e4743: Pull complete
0f72e97e1f61: Pull complete
2d84d0050f56: Pull complete
3988829a97fa: Pull complete
9388ce60a1b5: Pull complete
3f38070b922c: Pull complete
2484a73c1218: Pull complete
be305d229b7a: Pull complete
Digest: sha256:b5927ed7ac79524251c3b717195c239023a33d169709c9f1d74f0a898975938f
Status: Downloaded newer image for gitlab/gitlab-ce:11.2.3-ce.0
Creating gitlablabcom_gitlab_1 ... done
Attaching to gitlablabcom_gitlab_1
gitlab_1  | Thank you for using GitLab Docker Image!
gitlab_1  | Current version: gitlab-ce=11.2.3-ce.0
gitlab_1  | 
gitlab_1  | Configure GitLab for your system by editing /etc/gitlab/gitlab.rb file
gitlab_1  | And restart this container to reload settings.
gitlab_1  | To do it use docker exec:
```

当你看到下面的请求记录日志后，你的应用便启动完成了。

```bash
gitlab_1  | ==> /var/log/gitlab/sidekiq/current <==
gitlab_1  | 2018-09-27_03:15:25.08874 2018-09-27T03:15:25.088Z 1013 TID-osuacsvzh PagesDomainVerificationCronWorker JID-f6d295ba87e7ea6c13f788d4 INFO: start
gitlab_1  | 2018-09-27_03:15:25.09873 2018-09-27T03:15:25.098Z 1013 TID-osuacsw65 StuckImportJobsWorker JID-465eb7f9283897dd1d622dc1 INFO: start
gitlab_1  | 2018-09-27_03:15:25.13306 2018-09-27T03:15:25.132Z 1013 TID-osuacsvzh PagesDomainVerificationCronWorker JID-f6d295ba87e7ea6c13f788d4 INFO: done: 0.044 sec
gitlab_1  | 2018-09-27_03:15:25.14188 2018-09-27T03:15:25.141Z 1013 TID-osuacsw65 StuckImportJobsWorker JID-465eb7f9283897dd1d622dc1 INFO: done: 0.043 sec
gitlab_1  | 
gitlab_1  | ==> /var/log/gitlab/gitlab-rails/production.log <==
gitlab_1  | Started GET "/help" for 127.0.0.1 at 2018-09-27 03:15:50 +0000
gitlab_1  | Processing by HelpController#index as */*
gitlab_1  | Completed 200 OK in 304ms (Views: 296.9ms | ActiveRecord: 3.0ms)
gitlab_1  | 
```

如果你是新用户，那么现在便可以结束文章浏览，访问你的 `GitLab` 进行体验了。

接下来我们开始老数据的备份，以及将数据迁移到新的 `GitLab` 应用中。

### 备份之前的 GitLab 数据

如果你也是使用容器化的方式运行 `GitLab` ，备份命令需要依赖 `Docker` 作为代理执行命令( `gitlab_gitlab_1` 为容器运行名称)。

```bash
docker exec -t gitlab_gitlab_1 gitlab-rake gitlab:backup:create
```

如果是裸机运行，那么直接运行下面的命令。

```bash
gitlab-rake gitlab:backup:create
```

命令顺利执行完毕后，然后可以在应用的 `/data/backups` 中找到备份的数据文件。 

将数据文件保存/转移到新的机器上之后，我们便可以开始正式的迁移工作。

### 迁移（还原）数据

先将备份文件放置回新的应用的 `/data/backups` 文件夹中。

然后使用 `docker ps` 获取运行容器的名称。

```bash
CONTAINER ID        IMAGE                          COMMAND                   CREATED             STATUS                    PORTS                                                                                  NAMES
3fba53e6c021        gitlab/gitlab-ce:11.2.3-ce.0   "bash -c 'sed -i \"s/…"   10 minutes ago      Up 24 minutes (healthy)   0.0.0.0:22->22/tcp, 0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp, 0.0.0.0:9005->9999/tcp   gitlablabcom_gitlab_1
```

然后执行 `docker exec -t gitlablabcom_gitlab_1 gitlab-rake gitlab:backup:restore RAILS_ENV=production` 即可开始恢复数据。

在执行过程中，程序会询问你是否愿意放弃当前应用的所有数据，开始恢复操作，请回复 `yes` 。

```bash
docker exec -t gitlablabcom_gitlab_1 gitlab-rake gitlab:backup:restore
Unpacking backup ... done
Before restoring the database, we will remove all existing
tables to avoid future upgrade problems. Be aware that if you have
custom tables in the GitLab database these tables and all data will be
removed.

Do you want to continue (yes/no)? yes
```

经过漫长的等待（根据数据量而定）。

```bash
done
Restoring uploads ... 
done
Restoring builds ... 
done
Restoring artifacts ... 
done
Restoring pages ... 
done
Restoring lfs objects ... 
done
This will rebuild an authorized_keys file.
You will lose any data stored in authorized_keys file.
Do you want to continue (yes/no)? yes

.......
Deleting tmp directories ... done
done
done
done
done
done
done
done
```

最后重新启动你的 `GitLab` 应用，迁移就完毕了。

## 最后

喜欢折腾的小伙伴可以扫二维码添加好友（记得注明来源和目的，方便备注拉群）。

我计划拉个小群，把喜欢折腾的朋友聚一起，在不发广告的情况下，聊聊软件、HomeLab、编程上的一些问题，来扩展和积累一些写作素材。

![](https://attachment.soulteary.com/2018/09/27/qrcode-2018.09.png)

---EOF