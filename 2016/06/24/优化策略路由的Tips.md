# 优化策略路由的Tips

最近一直在优化家庭网络体验，还特意组装了一台新的小主机(N3700)，除了兼顾软路由和NAS的需求之外，还提供了代码仓库等功能，具体经验之后会慢慢写出来。

先聊其中一个细节吧，关于使用“策略路由优化上网体验”。

## 优化内容

关于策略路由的优化，通常做法是从apnic.net获取全球的地址列表，然后经过地址筛选后，做具体的策略。

但是，随着近几年列表中添加的ipv6地址越来越多，列表文件也越来越大，上述策略中常常看到的脚本方案或许就存在了问题：

```bash
wget -O- 'http://ftp.apnic.net/apnic/stats/apnic/delegated-apnic-latest' | grep ipv4 | grep CN | awk -F\| '{ printf("%s/%d\n", $4, 32-log($5)/log(2)) }' > /yourpath/your-ip-list
```

这条不需要过多解释，然而这句却存在着不止一个优化点，尤其是在性能孱弱的路由器设备上。

_为了追求真实场景，下面的命令在路由器上执行。_

## 场景测试

先将IP列表文件下载并保存，提供给之后的测试使用。

```bash
wget 'http://ftp.apnic.net/apnic/stats/apnic/delegated-apnic-latest'

```

顺手查看一下文件的总行数，快破50k了。


```bash
cat delegated-apnic-latest | wc -l
47133
```

最常规的做法是先过滤符合ipv4的数据行，那么我们在路由上执行（多次执行，耗时范围在0.92~0.93s）


```bash
[root@PandoraBox:/tmp]#time cat delegated-apnic-latest | grep ipv4 | grep CN | wc -l  
real    0m 0.92s
user    0m 0.00s
sys 0m 0.06s
7481
```

如果先过滤符合区域的数据呢，（多次执行，0.58~0.60s）

```bash
[root@PandoraBox:/tmp]#time cat delegated-apnic-latest | grep CN | grep ipv4 | wc -l  
real    0m 0.59s
user    0m 0.01s
sys 0m 0.03s
7481
```

其实港澳台的网站，除了台湾地区的会出现不能访问，多数港澳的IP还是很靠谱的，没必要走策略路由，如果遇到了再添加也不迟。

```bash
[root@PandoraBox:/tmp]#time cat delegated-apnic-latest | egrep "CN|MO|HK" | grep ipv4 | wc -l
real    0m 0.65s
user    0m 0.01s
sys 0m 0.03s
9414
```

## 优化结果

最后修改完毕的命令为：

```bash
wget -O- 'http://ftp.apnic.net/apnic/stats/apnic/delegated-apnic-latest' | egrep "CN|MO|HK" | grep ipv4 | awk -F\| '{ printf("%s/%d\n", 
$4, 32-log($5)/log(2)) }' > YOURFILE
```

同状况下，目前47k条数据节约路由计算时间大概0.3s+。 另外测试网络连通性，由于robots的限制，wget版本过旧，本地证书过老等问题，实测用下面的命令会更靠谱。

```bash
wget --execute robots=off --inet4-only --spider --quiet --tries=3 --timeout=10 --no-check-certificate www.google.com
```


