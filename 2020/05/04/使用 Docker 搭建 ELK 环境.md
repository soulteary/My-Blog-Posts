# 使用 Docker 搭建 ELK 环境

本文将聊聊如何使用 Docker 搭建 ELK （Elasticsearch、Logstash、Kibana）。

文章将分两个部分对搭建进行介绍，用于开发测试以及一般分析需求的环境，以及弹性扩容后可以用于一般生产的环境。

因为借助于方便的 Docker，完整操作时间不超过 15 分钟，如果你对 Docker 还不熟悉，可以浏览之前的文章。

## 写在前面

为了方便搭建，我们使用 [https://github.com/deviantony/docker-elk](https://github.com/deviantony/docker-elk) 这个开源项目，这个项目维护了 ELK 技术栈最近的三个版本，也就是 7.x、6.x、5.x ，本文将使用最新版本。

用于开发测试的基础环境使用一台1c2g的虚拟机即可，当然机器资源越多我们的服务运行效率也会越高、相同时间内数据处理能力也就越大。而用于一般生产环境建议根据自己具体情况给予更多资源。

先聊聊测试环境搭建。

## 测试开发环境

使用 Git Clone 命令将项目下载到所需要的位置。

```bash
git clone https://github.com/deviantony/docker-elk.git /app/docker-elk

Cloning into '/app/docker-elk'...
remote: Enumerating objects: 1729, done.
remote: Total 1729 (delta 0), reused 0 (delta 0), pack-reused 1729
Receiving objects: 100% (1729/1729), 410.25 KiB | 11.00 KiB/s, done.
Resolving deltas: 100% (705/705), done.
```

在启动项目前，我们先了解下基础的目录结构。

```TeXT
├── docker-compose.yml
├── docker-stack.yml
├── elasticsearch
│   ├── config
│   │   └── elasticsearch.yml
│   └── Dockerfile
├── extensions
│   ├── apm-server
│   ├── app-search
│   ├── curator
│   ├── logspout
├── kibana
│   ├── config
│   │   └── kibana.yml
│   └── Dockerfile
├── LICENSE
├── logstash
│   ├── config
│   │   └── logstash.yml
│   ├── Dockerfile
│   └── pipeline
│       └── logstash.conf
└── README.md
```

可以清楚看到，项目主要使用根目录的 **docker-compose.yml** 作为启动配置，并在首次启动时，构建相关服务的容器镜像。

了解了项目工作方式后，可以使用  `docker-compose up` 来启动项目。

```TeXT
docker-compose up

Creating network "docker-elk_elk" with driver "bridge"
Creating volume "docker-elk_elasticsearch" with default driver
Building elasticsearch
Step 1/2 : ARG ELK_VERSION
Step 2/2 : FROM docker.elastic.co/elasticsearch/elasticsearch:${ELK_VERSION}
7.6.2: Pulling from elasticsearch/elasticsearch
c808caf183b6: Downloading [==>                                                ]  3.736MB/85.21MB
d6caf8e15a64: Downloading [===================>                               ]  13.69MB/34.71MB
b0ba5f324e82: Download complete
d7e8c1e99b9a: Downloading [=>                                                 ]  11.71MB/321.6MB
85c4d6c81438: Waiting
3119218fac98: Waiting
914accf214bb: Waiting
...

Creating docker-elk_elasticsearch_1 ... done
Creating docker-elk_logstash_1      ... done
Creating docker-elk_kibana_1        ... done
Attaching to docker-elk_elasticsearch_1, docker-elk_logstash_1, docker-elk_kibana_1
logstash_1       | OpenJDK 64-Bit Server VM warning: Option UseConcMarkSweepGC was deprecated in version 9.0 and will likely be removed in a future release.
elasticsearch_1  | Created elasticsearch keystore in /usr/share/elasticsearch/config
logstash_1       | WARNING: An illegal reflective access operation has occurred
logstash_1       | WARNING: Illegal reflective access by com.headius.backport9.modules.Modules (file:/usr/share/logstash/logstash-core/lib/jars/jruby-complete-9.2.9.0.jar) to method sun.nio.ch.NativeThread.signal(long)
logstash_1       | WARNING: Please consider reporting this to the maintainers of com.headius.backport9.modules.Modules
logstash_1       | WARNING: Use --illegal-access=warn to enable warnings of further illegal reflective access operations
logstash_1       | WARNING: All illegal access operations will be denied in a future release
elasticsearch_1  | OpenJDK 64-Bit Server VM warning: Option UseConcMarkSweepGC was deprecated in version 9.0 and will likely be removed in a future release.
elasticsearch_1  | {"type": "server", "timestamp": "2020-05-03T03:47:40,483Z", "level": "INFO", "component": "o.e.e.NodeEnvironment", "cluster.name": "docker-cluster", "node.name": "0d05db8360df", "message": "using [1] data paths, mounts [[/usr/share/elasticsearch/data (/dev/sda2)]], net usable_space [83.6gb], net total_space [97.9gb], types [ext4]" }
elasticsearch_1  | {"type": "server", "timestamp": "2020-05-03T03:47:40,493Z", "level": "INFO", "component": "o.e.e.NodeEnvironment", "cluster.name": "docker-cluster", "node.name": "0d05db8360df", "message": "heap size [247.5mb], compressed ordinary object pointers [true]" }
kibana_1         | {"type":"log","@timestamp":"2020-05-03T03:47:40Z","tags":["info","plugins-service"],"pid":6,"message":"Plugin \"case\" is disabled."}
```

启动过程中的日志会类似上面这样，因为首次启动需要从官网镜像仓库中下载相关镜像，所以会慢一些。当你看到终端输出类似上面的日志时，说明服务已经启动完毕。

为了验证，我们可以使用浏览器或者 curl 等工具访问机器地址加端口号 9200，并使用默认用户名 `elastic` 和默认密码 `changeme` 来访问 Elasticsearch HTTP 端口，如果一切正常，你将看到类似下面的结果。

```json
{
  "name" : "0d05db8360df",
  "cluster_name" : "docker-cluster",
  "cluster_uuid" : "Mq2EZX59TqW7ysGx7Y-jIw",
  "version" : {
    "number" : "7.6.2",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "ef48eb35cf30adf4db14086e8aabd07ef6fb113f",
    "build_date" : "2020-03-26T06:34:37.794943Z",
    "build_snapshot" : false,
    "lucene_version" : "8.4.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

现在不着急访问 Kibana ，我们继续进行配置调整。

### 重置内建用户密码

使用 `docker-compose exec -T elasticsearch bin/elasticsearch-setup-passwords auto --batch` 命令对服务默认的账户进行默认密码重置。

```TeXT
docker-compose exec -T elasticsearch bin/elasticsearch-setup-passwords auto --batch

Changed password for user apm_system
PASSWORD apm_system = YkELBJGOT6AxqsPqsi7I

Changed password for user kibana
PASSWORD kibana = FxRwjm5KRYvHhGEnYTM9

Changed password for user logstash_system
PASSWORD logstash_system = A4f5VOfjVWSdi0KAZWGu

Changed password for user beats_system
PASSWORD beats_system = QnW8xxhnn7LMlA7vuI7B

Changed password for user remote_monitoring_user
PASSWORD remote_monitoring_user = OvjEGR13wjkOkIbWqaEM

Changed password for user elastic
PASSWORD elastic = PGevNMuv7PhVnaYg7vJw
```

将密码妥善保存后，我们需要将 `docker-compose.yml` 配置文件中的 elasticsearch 服务的 `ELASTIC_PASSWORD` 去掉，这样可以确保服务启动只使用我们刚刚重置后的密码（keystore）。以及需要对 kibana 、 logstash 配置文件中的信息进行替换，将文件中的 **elastic** 用户的密码进行更新，相关文件我们在开篇的目录结构中有提过，暂时先修改下面三个文件就可以了：

- kibana/config/kibana.yml
- logstash/config/logstash.yml
- logstash/pipeline/logstash.conf

需要注意的是， logstash pipeline 需要一个高权限账号，当前测试开发过程中，可以使用上面重置后的 elastic 账号和密码，如果是生产使用可以参考官方文档 [Configuring Security in Logstash](https://www.elastic.co/guide/en/logstash/current/ls-security.html) 进行操作，分配一个新的专用账号。

在配置修改完毕后，我们执行 `docker-compose restart` 重启相关服务。

```TeXT
docker-compose restart

Restarting docker-elk_kibana_1        ... done
Restarting docker-elk_logstash_1      ... done
Restarting docker-elk_elasticsearch_1 ... done
```

如果日志中没有出现任何 401 或者访问拒绝的内容，说明服务一切正常。

### 使用 Kibana 控制台

启动之后，我们使用浏览器访问服务器IP+端口5601，打开 kibana 控制台。

![Kibana  默认登陆界面](https://attachment.soulteary.com/2020/05/04/kibana-login.png)

使用elastic 账号和密码登录后，就能够看到 Kibana 的界面了，如果第一次使用，会看到欢迎界面。

![Kibana  登陆后欢迎界面](https://attachment.soulteary.com/2020/05/04/kibana-logined.png)

在登陆之后，第一次使用，可以考虑导入示例数据，来帮助了解 Kibana 的基础功能。

![Kibana  导入示例数据](https://attachment.soulteary.com/2020/05/04/kibana-add-example-data.png)

接下来就是自由探索的过程了，: )

![Kibana  默认示例面板](https://attachment.soulteary.com/2020/05/04/kibana-example-dashboard.png)

### 关闭付费组件

打开设置界面，选择 Elasticsearch 模块中的 License Management ，可以看到默认软件会启动为期一个月的高级功能试用。

![ELK  新安装后默认授权](https://attachment.soulteary.com/2020/05/04/kibana-license.png)

在 [官方订阅](https://www.elastic.co/cn/subscriptions) 页面，我们可以看到官方支持的订阅类型，一般来说，如果没有特殊需求，使用基础版本就好。

![ELK 官方付费方案](https://attachment.soulteary.com/2020/05/04/elk-subscriptions.png)

如果你想要了解软件当前运行状态，除了登陆机器，查看宿主机和容器状态外，在监控界面，我们也可以方便快捷的看到单节点中各个服务的运行状态和资源占用。

![ELK  单节点版本健康状态](https://attachment.soulteary.com/2020/05/04/elk-single-node-monitoring.png)

选择“Revert to Basic license”，选择回退到基础版本，可以看到整个界面都简洁了不少，至此如果不付费的话，就可以安心合法的使用软件了。

![ELK  基础版本授权界面](https://attachment.soulteary.com/2020/05/04/elk-basic-license.png)

当然，你也可以在 elasticsearch.yml 配置文件中的 `xpack.license.self_generated.type` 修改为 `basic` 来禁用 X-Pack 相关功能。

```yaml
xpack.license.self_generated.type: basic
```

对接各种软件/系统进行日志收集，以及定制你的可视化界面或者 API 我们后面有机会再聊，接下来继续聊聊，如何进一步搭建配置 ELK 。

## 修改自官方示例的生产环境

生产环境的基础要求是高可用性，常规实现方案中见的比较多的是“多副本/实例”，多机器，多机架，甚至多区域部署。

本文我们先聊聊最简单的多实例。

### 前置准备

如果想让生产环境中使用 Docker 运行 ELK，有一些必备的系统设置必不可少。

首先调整 `vm.max_map_count` 的数值，至少调整到 262144 以上。在 `/etc/sysctl.conf` 添加下面的内容即可。

```bash
vm.max_map_count = 262144

sysctl -w vm.max_map_count=262144
```

然后调整 `ulimits` 以及 `nprocedit`，因为我们使用容器运行 ELK 相关应用，所以直接修改 compose 配置文件中的 ES 配置就可以了：

```yaml
ulimits:
  memlock:
    soft: -1
    hard: -1
```

我们继续调整，关闭内存交换，同样修改 compose 配置文件中的 ES 配置即可：

```yaml
environment:
  bootstrap.memory_lock: "true"
```

Java 堆大小同样需要调整，默认的数值如下，在生产环境中太小了，更详细的内容可以参考[这里](https://www.elastic.co/guide/en/elasticsearch/reference/current/heap-size.html)。

```yaml
environment:
  ES_JAVA_OPTS: "-Xmx1g -Xms1g"
```

如果你真实使用在生产环节，务必开启 TLS 和密码认证，此处为了不额外扩展篇幅（以及懒得申请通配符证书/配置自签名）先使用关闭安全配置的方式忽略这个设置 : )

### 修改配置支持多实例

官方多实例方案（[这篇官方指引](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html)），采取在 compose 中定义三个不同的服务，然后使用第一个服务作为 Master 对外暴露服务，我们先以该方案为基础，并进行一些调整。

首先创建服务所需要的数据目录，并赋予所需要的权限。

```yaml
mkdir -p data/{es01,es02,es03}
chmod g+rwx data/*
chgrp 0 data/*
```

之前在单节点中，我们挂载的数据使用的是容器的数据卷方案，在这里，我们可以考虑使用性能更好的文件映射替换之前的方案，当然也可以配合[储存插件](https://docs.docker.com/engine/extend/legacy_plugins/)使用：

```yaml
volumes:
  - ./elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml:ro
  - ./data:/usr/share/elasticsearch/data:rw
```

考虑到多实例之间配置几乎一致，并且如果要单独维护太过啰嗦，我们可以将 `elasticsearch.yml` 内的配置使用环境变量的方式写在 compose 配置文件中：

```yaml
environment:
  cluster.name: "docker-cluster"
  network.host: "0.0.0.0"
  bootstrap.memory_lock: "true"
  xpack.license.self_generated.type: "basic"
  xpack.security.enabled: "false"
  xpack.monitoring.collection.enabled: "true"
  ES_JAVA_OPTS: "-Xmx1g -Xms1g"
```

此外，因为涉及到多个节点组网，所以这里需要额外指定这些实例的节点名称等信息：

```yaml
version: "3.2"

services:
  elasticsearch01:
    environment:
      node.name: "es01"
      discovery.seed_hosts: "es02,es03"
      cluster.initial_master_nodes: "es01,es02,es03"

  elasticsearch02:
    environment:
      node.name: "es02"
      discovery.seed_hosts: "es01,es03"
      cluster.initial_master_nodes: "es01,es02,es03"

  elasticsearch03:
    environment:
      node.name: "es03"
      discovery.seed_hosts: "es01,es02"
      cluster.initial_master_nodes: "es01,es02,es03"
```

最后，按照官网的推荐的模式，让其中一台机器支持对外暴露端口，与外部进行数据交换即可。

```yaml
ports:
  - 9200:9200
  - 9300:9300
```

到这里多实例就配置完成了。

### 更新 Logstash  配置

logstash  需要更新的有两处，一处是要让服务在 刚刚定义的 elasticsearch 实例启动之后再启动，另外可以将配置以相同方式改写，方便后续维护。

```yaml
logstash:
  volumes:
    - ./logstash/config/logstash.yml:/usr/share/logstash/config/logstash.yml:ro
    - ./logstash/pipeline:/usr/share/logstash/pipeline:ro
  depends_on:
    - elasticsearch01
    - elasticsearch02
    - elasticsearch03
```

接着需要更新 `logstash/config/logstash.yml` 配置中的 `xpack.monitoring.elasticsearch.host` 信息，确保启动不会出现问题。

```yaml
xpack.monitoring.elasticsearch.hosts: [ "http://es01:9200" ]
```

以及 `logstash/pipeline/logstash.conf` 配置中的 `output. elasticsearch` 信息为 `hosts => "es01:9200"`。

### 更新 Kibana  配置

需要更新的地方有三处，两处同 Logstash ，额外还有一处是指定 `ELASTICSEARCH_URL` 地址为我们暴露服务的 elasticsearch 实例的地址。

```yaml
kibana:
  volumes:
    - ./kibana/config/kibana.yml:/usr/share/kibana/config/kibana.yml:ro
  ports:
    - "5601:5601"
  depends_on:
    - elasticsearch01
    - elasticsearch02
    - elasticsearch03
  environment:
    - ELASTICSEARCH_URL=http://es01:9200
    - xpack.security.enabled=false
```

以及需要额外更新 `kibana/config/kibana.yml` 配置中的 `elasticsearch.hosts` 字段信息：

```yaml
elasticsearch.hosts: [ "http://es01:9200" ]
```

### 完整配置文件

上述配置完成内容如下：

```yaml
version: "3.2"

services:
  elasticsearch01:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.6.2
    container_name: es01
    volumes:
      - ./data/es01:/usr/share/elasticsearch/data:rw
    ports:
      - 9200:9200
      - 9300:9300
    environment:
      node.name: "es01"
      cluster.name: "docker-cluster"
      network.host: "0.0.0.0"
      discovery.seed_hosts: "es02,es03"
      cluster.initial_master_nodes: "es01,es02,es03"
      bootstrap.memory_lock: "true"
      xpack.license.self_generated.type: "basic"
      xpack.security.enabled: "false"
      xpack.monitoring.collection.enabled: "true"
      ES_JAVA_OPTS: "-Xmx1g -Xms1g"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    networks:
      - elk

  elasticsearch02:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.6.2
    container_name: es02
    volumes:
      - ./data/es02:/usr/share/elasticsearch/data:rw
    environment:
      node.name: "es02"
      cluster.name: "docker-cluster"
      network.host: "0.0.0.0"
      discovery.seed_hosts: "es01,es03"
      cluster.initial_master_nodes: "es01,es02,es03"
      bootstrap.memory_lock: "true"
      xpack.license.self_generated.type: "basic"
      xpack.security.enabled: "false"
      xpack.monitoring.collection.enabled: "true"
      ES_JAVA_OPTS: "-Xmx1g -Xms1g"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    networks:
      - elk

  elasticsearch03:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.6.2
    container_name: es03
    volumes:
      - ./data/es03:/usr/share/elasticsearch/data:rw
    environment:
      node.name: "es03"
      cluster.name: "docker-cluster"
      network.host: "0.0.0.0"
      discovery.seed_hosts: "es01,es02"
      cluster.initial_master_nodes: "es01,es02,es03"
      bootstrap.memory_lock: "true"
      xpack.license.self_generated.type: "basic"
      xpack.security.enabled: "false"
      xpack.monitoring.collection.enabled: "true"
      ES_JAVA_OPTS: "-Xmx1g -Xms1g"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    networks:
      - elk

  logstash:
    build:
      context: logstash/
      args:
        ELK_VERSION: $ELK_VERSION
    volumes:
      - ./logstash/config/logstash.yml:/usr/share/logstash/config/logstash.yml:ro
      - ./logstash/pipeline:/usr/share/logstash/pipeline:ro
    ports:
      - "5000:5000/tcp"
      - "5000:5000/udp"
      - "9600:9600"
    environment:
      LS_JAVA_OPTS: "-Xmx1g -Xms1g"
    networks:
      - elk
    depends_on:
      - elasticsearch01
      - elasticsearch02
      - elasticsearch03

  kibana:
    build:
      context: kibana/
      args:
        ELK_VERSION: $ELK_VERSION
    volumes:
      - ./kibana/config/kibana.yml:/usr/share/kibana/config/kibana.yml:ro
    ports:
      - "5601:5601"
    networks:
      - elk
    depends_on:
      - elasticsearch01
      - elasticsearch02
      - elasticsearch03
    environment:
      - ELASTICSEARCH_URL=http://es01:9200
      - xpack.security.enabled=false

networks:
  elk:
    driver: bridge
```

其实如果你全部使用官方镜像，而不做二次定制，比如安装插件的话，那么都改为官方镜像名称即可。

启动服务之后，打开 kibana 可以看到多实例的 ELK 环境就配置完毕了。

![ELK  多实例配置完成](https://attachment.soulteary.com/2020/05/04/elk-cluster.png)

## 其他

官网文档对于配置的内容描述有不少，感兴趣的同学可以进一步了解，比如这篇[ Important System Configuration ](https://www.elastic.co/guide/en/elasticsearch/reference/current/system-config.html) ，原理同样适用于一些其他的应用。

关于如何使用各种 beat 服务进行日志上报，可以参考官方之前给出的[示例文件](https://github.com/elastic/stack-docker/blob/master/docker-compose.yml)。

## 最后

接下来我会围绕日志写一些有趣又简单易用的内容，本篇是第一篇内容。

--EOF
