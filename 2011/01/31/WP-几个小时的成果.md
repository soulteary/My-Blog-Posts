# [WP]几个小时的成果

翻了一下08年的旧存档，原来很多功能，以前已经写好了，很纠结的改了一阵，然后就是无限清理缓存等。

YSLOW目前80/90/93，主要扣分在CDN服务器,请求数目过多，这里牵扯到均衡负载服务器和外部文件合并以及CSS sprite图片合并。

CDN的话，使用.NET域名所在的国外空间也可以，但是那个线路感觉PING值过高了,有200多，犹豫。

请求数目过多，这个是因为内容限制，内页的话可能效果更差点，不过也有解决方法，动态加载数据。

文件合并的话，重新写了2个模块来动态加载JS,CSS等，慢慢细化吧。

CSS sprite 的话，这个要慎重了，慢慢修改吧，能减少一点是一点，毕竟这个和持续的维护有点冲突。

[![yslow](https://attachment.soulteary.com/2011/01/31/yslow.jpg "yslow")](https://attachment.soulteary.com/2011/01/31/yslow.jpg) 

[![pagespeed](https://attachment.soulteary.com/2011/01/31/pagespeed.jpg "pagespeed")](https://attachment.soulteary.com/2011/01/31/pagespeed.jpg) 

PAGE SPEED的话，也提高了不少，到84了。 感觉PAGE SPEED比YSLOW严格多了。 

突出问题，在于使用干净的域名来分流数据和YSLOW的CDN一样。

不过叫[Parallelize downloads across hostnames。](http://code.google.com/speed/page-speed/docs/rtt.html#ParallelizeDownloads "Learn More") 

然后就是CSS sprite比较凸显了，接下来就是很多细节，压缩CSS,JS，减少静态变量长度,优化图片等。 

稍后做一下伪CDN，试一试效果。

[Learn More](http://code.google.com/speed/page-speed/docs/rtt.html#ParallelizeDownloads "Learn More")

