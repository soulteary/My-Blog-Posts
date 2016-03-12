# Confluence 使用拾遗

[前一篇](https://soulteary.com/2019/03/30/construct-confluence-with-docker.html) 内容介绍了如何快速使用容器搭建 **Confluence **，在一周的实际使用之后，我们发现了一些小问题，本篇将作为之前内容的补充。

## 如何修正应用时区

应用启动之后，你将看到时区默认是：**GMT +0** ，这显然不符合我们的需求。

要解决这个问题，可以通过挂载宿主机 `/etc/localtime` 到容器内，并在JVM变量中添加 `-Duser.timezone=GMT+08` 参数。

同时在挂载的时候要注意，为了避免容器内部应用修改 `/etc/localtime` ，文件需要设置为只读。

上面操作看起来很麻烦，但是实际上配置代码很简单，比如这样：

```yaml
environment:
  - 'CATALINA_OPTS= -Duser.timezone=GMT+08'
volumes:
  - /etc/localtime:/etc/localtime:ro
```

## 修改可用内存资源

当使用人数比较少、内容也比较少的时候，运行 Confluence 并不会出现什么异常。但是当内容多了、或者用户数多了之后，Confluence 会出现一些性能问题，比如卡顿。

此时，可以通过增加可用内存资源来解决这个问题。官方默认数值都是 `1024m`，修改配置的时候，需要我们根据实际情况进行调节：

- 比如我有一台 `4C8G` 的主机，考虑到系统进程、运维软件、容器服务的消耗，我选择给予 Confluence 6GB 内存的上限，而下限和默认保持一致就好。

举个例子，下面这段配置赋予程序可用内存范围就是 `1G` 到 `6G`。

```yaml
environment:
  - 'JVM_MINIMUM_MEMORY=1024m'
  - 'JVM_MAXIMUM_MEMORY=6144m'
```

## 关闭数据分析收集

官方有默认开启数据分析功能，会将你的用户行为（不含数据），发送至厂商数据分析平台。

常规的关闭方式是使用管理员账号，选择“禁用”按钮，但是如果你发现禁用不灵，可以通过接口调用手动关闭分析服务。

在容器内部执行下面的命令（假设超级管理员账号和密码都是 `admin`）：

```bash
curl -vvv -H "Content-Type:application/json" -H "Accept:application/json" --user admin:admin -X PUT -d '{"analyticsEnabled": "false"}' http://localhost:8090/rest/analytics/1.0/config/enable
```

## 完整配置

最后，将上述修正综合一下，完整的配置文件如下：

```yaml
version: '3'

services:

  confluence:
    image: atlassian/confluence-server:6.15.1-alpine
    expose:
      - 8090
      - 8091
    networks:
      - traefik
    labels:
      - "traefik.enable=true"
      - "traefik.port=8090"
      - "traefik.frontend.rule=Host:${DOMAIN}"
      - "traefik.frontend.entryPoints=http,https"
      - "traefik.frontend.auth.basic.removeHeader=true"
      - "traefik.frontend.auth.basic=${BASIC_AUTH}"
    environment:
      - 'CATALINA_OPTS= -Duser.timezone=GMT+08'
      - 'JVM_MINIMUM_MEMORY=1024m'
      - 'JVM_MAXIMUM_MEMORY=6144m'
    volumes:
      - ./data:/var/atlassian/application-data/confluence
      - ./mysql-connector-java-5.1.47.jar:/opt/atlassian/confluence/confluence/WEB-INF/lib/mysql-connector-java-5.1.47.jar
      - ./server.xml:/opt/atlassian/confluence/conf/server.xml
      - /etc/localtime:/etc/localtime:ro

networks:
  traefik:
    external: true
```

## 最后

额外说一句，官方容器镜像的文档真的是一塌糊涂。不过功能设计是真的好用，特别适合定制化需求不强烈的初创公司/团队使用。
