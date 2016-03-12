# [apache]本地测试子域名

两个方法，个人感觉第一个好点，第二个有点不好用...至少WIN2K3感觉...

<!-- more -->

想一个你觉得不错的域名，比如test.com
在你本地的host(win2k3:C:\WINDOWS\system32\drivers\etc\hosts)文件中添加DNS映射.
127.0.0.1 test.com
127.0.0.1 cache.test.com


方法1:

新建一个名为vhost.map的文件，内容格式如下:

```text
test.com C:\xampp\htdocs\
www.test.com C:\xampp\htdocs\
cache.test.com C:\xampp\htdocs\cache
</pre>
修改httpd.conf开启mod_rewrite并添加内容
<pre lang="apache">
RewriteLog         logs/rewrite.log
RewriteLogLevel    0
RewriteEngine      on
RewriteMap         lowercase int:tolower
RewriteMap         vhost txt:C:\xampp\apache\conf\vhost.map
RewriteCond        ${lowercase:%{HTTP_HOST}|NONE} ^(.+)$
RewriteCond        ${vhost:%1} ^(C:/.*)$
RewriteRule        ^/(.*)$ %1/$1 [E=VHOST:${lowercase:%{HTTP_HOST}}]
```


注意修改其中绝对路径为你的路径,以及重启APACHE

方法2:

使用APCHE的ServerAlias模块
修改httpd.conf开启ServerAlias并添加内容
<VirtualHost *:80>
ServerAlias cache.test.com
DocumentRoot C:\xampp\htdocs\
</VirtualHost>
注意修改其中绝对路径为你的路径,以及重启APACHE

