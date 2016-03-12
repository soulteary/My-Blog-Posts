# APACHE禁止使用IP访问网站

在APACHE中配置虚拟站点后，如果没有对原来的IP进行域名绑定，默认会将IP解析到某一个VHOST站点上。

那么如何解决呢，其实有一种很简单的方法，就是将IP作为一个VHOST的`ServerName`，如下面配置中的第二节：

```bash
<virtualhost *:80="">ServerAdmin i@soulteary.com
    DocumentRoot /var/www/html/domain/public_html
    ServerName domain
    ErrorLog /var/www/html/domain/logs/error_log
    CustomLog /var/www/html/domain/logs/access_log common
    <directory "="" var="" www="" html="" domain="" public_html"="">Options FollowSymLinks
     AllowOverride All</directory>
</virtualhost> 

<virtualhost *:80="">ServerName AAA.BBB.CCC.DDD
    DocumentRoot /var/www/welcome/public_html
</virtualhost> 
```

再次访问`AAA.BBB.CCC.DDD`是不是发现不在是出现之前的页面了呢？

当然如果你想拒绝访问的话，可以使用`deny from ALL`，或者使用转向，这个看你的心情了。

