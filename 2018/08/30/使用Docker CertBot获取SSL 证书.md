# 使用 Docker CertBot 获取 SSL 证书

`Let‘s Encrypt` 在很久之前就开始了证书的免费申请，但是随着 API 的升级、功能的增加，之前使用`acme.sh` 脚本就能够轻松获取证书的操作，变得越来越麻烦，而且随着配置项越来越多，浏览文档很难快速了解到什么才是当前的最佳实践。

## 原来的方案

在更新服务器操作系统之后，原本一直使用的 `acme.sh` 出现了问题：读取不到我配置的 DNS 账户名称，不管我是否直接将账号写入了执行脚本中。考虑到未来这套证书获取客户端还存在升级的情况，继续修改它显然不是一个理智的选择。

## 使用官方推荐的客户端

虽然我使用的 Traefik 支持自动申请证书，但是一来我希望有更多的配置项，比如[加密方案的选择](https://certbot.eff.org/docs/ciphers.html)、比如[证书单独保存以备复用](https://certbot.eff.org/docs/using.html#where-certs)。

二来在查看官方文档之后，发现官方推荐了一套名为 `certbot` 的客户端，并提供了 `Docker` 镜像，[官方文档](https://certbot.eff.org/docs/install.html#running-with-docker)。

在翻阅文档之后，我们发现客户端使用很简单，只需要2条命令，几个输入确认就可以了。看起来还不错，那么试试看吧。

## DNS 模式获取证书

为了保证解耦，个人使用无侵入的 DNS 模式，其实使用网站根目录放置验证文件也是一样的。

官方提供了十几种主流的 DNS 服务商的镜像，还提供了示例支持你自己[进行封装](https://github.com/certbot/certbot/tree/master/certbot-dns-rfc2136)。

这里我选择的是 `cloudflare` 这家服务商，所以对应的镜像是 `certbot/dns-cloudflare`，你可以酌情修改。

因为使用 DNS 模式，需要提供 DNS 的验证文件（包含邮箱和私钥），所以这里需要先创建一个验证文件。

```bash
mkdir -p /data/letsencrypt/
touch /data/letsencrypt/cloudflare.ini
```

然后在 `cloudflare.ini` 中写入你的数据，比如：

```ini
# Cloudflare API credentials used by Certbot
dns_cloudflare_email = example@example.com
dns_cloudflare_api_key = e89af204ab0e06def9c0846c202d1dec40e80
```

之后使用 Docker 的一次性执行模式启动一个客户端容器即可：

```bash
docker run -it --rm --name certbot \
            -v "/etc/letsencrypt:/etc/letsencrypt" \
            -v "/var/lib/letsencrypt:/var/lib/letsencrypt" \
            -v "/data/letsencrypt:/.secrets" \
            certbot/dns-cloudflare certonly \
            --dns-cloudflare-credentials /.secrets/cloudflare.ini \
            --dns-cloudflare-propagation-seconds 60 \
            --server https://acme-v02.api.letsencrypt.org/directory \
            -d soulteary.com -d '*.soulteary.com'
```


如果你不需要签通配符证书的话，那么可以去掉 `--server https://acme-v02.api.letsencrypt.org/directory` 参数，另外，如果你的域名生效时间很长，可以考虑适当调大 `--dns-cloudflare-propagation-seconds 60` 中的等待时间（单位秒）。

执行脚本之后，会问你几个简单的问题，依次是选择申请证书的验证方式（DNS记录、临时验证Web服务、网站根目录的静态文件），用户协议是否同意，询问你的邮箱，并分享给基金会，如果你没有使用 `-d` 参数声明要签的网站域名还会询问你网站域名是什么。

如果一切顺利，你的证书公钥私钥等文件将会一家人整整齐齐的摆放在我们映射好的目录：

```text
/etc/letsencrypt/live
```

通过查看文件目录，可以得知，这里存放的是最新申请到的证书的软链。

```text
lrwxrwxrwx 1 root root   37 Aug 30 12:49 cert.pem -> ../../archive/soulteary.com/cert1.pem
lrwxrwxrwx 1 root root   38 Aug 30 12:49 chain.pem -> ../../archive/soulteary.com/chain1.pem
lrwxrwxrwx 1 root root   42 Aug 30 12:49 fullchain.pem -> ../../archive/soulteary.com/fullchain1.pem
lrwxrwxrwx 1 root root   40 Aug 30 12:49 privkey.pem -> ../../archive/soulteary.com/privkey1.pem
```

所以你大可不必担心你更新证书失败，连回滚的机会都没有的情况。

至于如何更新、续签证书呢，答案也很简单，上面那段 `docker run` 申请命令，重新运行一遍即可，在输入了域名之后，选择 `[renew]` 即可。

当然，客户端也提供了免交互的方案，把 `docker run` 镜像后的参数修改为 `renew` 即可一次性续签所有证书。

## 其他

其实也没有太大必要去调整加密算法，除非你要刻意兼容特别老的设备，使用默认配置即可。

加上 Traefik ，或者应该说 golang 已经支持了 `CHACHA20_POLY1305`（[文档](https://golang.org/pkg/crypto/tls/#pkg-constants)），签出来的证书，直接挂载到 Traefik 即可支持当前主流的所有设备。

详见下面的证书测试报告：

- [SSL Lab 证书分析](https://www.ssllabs.com/ssltest/analyze.html?d=soulteary.com)

--EOF

