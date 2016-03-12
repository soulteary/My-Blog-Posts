# [Server]htaccess常见使用方法总结

屏蔽访问来源

```apache
order allow,deny

//屏蔽来自 xxx.com 的访问
deny from xxx.com
//屏蔽 IP 为 192.168.1.1 的访问
deny from 192.168.1.1
//屏蔽 IP 范围在 192.168.1.0~192.168.1.255 的访问
deny from 192.168.1.
//屏蔽所有的访问来源
allow from all
防盗链
//禁止盗链gif格式和jpg格式的文件
RewriteEngine On
RewriteCond %{ HTTP_REFERER } !^http://(www.)?域名.com/.*$ [NC]
RewriteRule .(gif|jpg)$ - [F]

//如果要实现替换下载的话
RewriteRule .(gif|jpg)$ &lt;a href="http://www"&gt;http://www&lt;/a&gt;.域名.com/替换的文件 [R,L]
地址转向
//旧地址自动转向到新地址
Redirect /旧目录/旧文档名 新文档的地址

//转向整个目录
Redirect 旧目录 新目录?
重新定义支持文档
//多个时,加载顺序为从左到右
DirectoryIndex index.php index.html index.htm
```

