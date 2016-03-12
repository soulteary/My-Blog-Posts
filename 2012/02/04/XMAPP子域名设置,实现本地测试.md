# XMAPP子域名设置，实现本地测试

XMAPP是一款不错的LAMP集成环境,不论是Ubuntu还是Windows,我都习惯用它来做测试。

虽然说是XMAPP独立子域名设置,但是其实还是Apache的子域名设置。

首先你要检查httpd.conf设置是否开启了**Virtual hosts** Linux套件环境httpd.conf在**/opt/lampp/etc/httpd.conf** Windows套件环境httpd.conf在**xampp安装目录\apache\conf** 打开conf文件,查找**Virtual hosts** 找到

```apache
# Virtual hosts
#Include etc/extra/httpd-vhosts.conf
```

如果

```apache
Include etc/extra/httpd-vhosts.conf
```

前有“#”号的话,去掉“#”，并保存文件。 开始配置虚拟目录，或者说是子域名 Linux 修改**/opt/lampp/extra/httpd-vhosts.conf**文件 在里面添加下面的内容。 Windows下则是**xampp安装目录\apache\conf\extra\httpd-vhosts.conf** 注意我的配置里的logs目录设置和DocumentRoot的路径,下面的设置是Windows下的。

```apache
NameVirtualHost *:80
#www.test.com
 <virtualhost *:80="">ServerAdmin admin@test.com
    DocumentRoot "C:/xampp/htdocs/"
    ServerName test.com
    ServerAlias www.test.com
    ErrorLog "logs/test.localhost.log"
    CustomLog "logs/test.localhost-access.log" combined</virtualhost> 

#cache.test.com
 <directory "c:="" xampp="" htdocs="" cache"="">Options Indexes FollowSymLinks ExecCGI Includes
    AllowOverride All
    Order allow,deny
    Allow from all</directory> 
 <virtualhost *:80="">ServerAdmin admin@test.com
    DocumentRoot "C:/xampp/htdocs/cache"
    ServerAlias cache.test.com
    ErrorLog "logs/cache.test.localhost.log"
    CustomLog "logs/cache.test.localhost-access.log" combined</virtualhost> 
```

因为我是要在本地做测试，所以呢，还要在本地做好解析 Linux在**/etc/hosts**文件中添加下面的内容 Windows在**\system32\drivers\etc\hosts**文件中

```apache
127.0.0.1 test.com
127.0.0.1 www.test.com
127.0.0.1 cache.test.com
```


