# Apache2 用.htaccess防盗链

[![防止盗链](https://attachment.soulteary.com/2008/01/16/daolian.gif "防止盗链")](https://attachment.soulteary.com/2008/01/16/daolian.gif)

在百度贴吧中进行测试，结果如图：

[![盗链时提示](https://attachment.soulteary.com/2008/01/16/2008-01-16_224756.jpg "盗链时提示")](https://attachment.soulteary.com/2008/01/16/2008-01-16_224756.jpg)

以下是配置参考：

```apache
RewriteEngine on  
RewriteCond %{HTTP_REFERER} !^http://promiseforever.com/.*$ [NC]  
RewriteCond %{HTTP_REFERER} !^http://promiseforever.com$ [NC]  
RewriteCond %{HTTP_REFERER} !^http://www.promiseforever.com/.*$ [NC]  
RewriteCond %{HTTP_REFERER} !^http://www.promiseforever.com$ [NC]  

RewriteRule .(zip|rar|7z|gif|jpg|psd|jpeg|bmp|ani|cur|ico|torrent|mp3|wma|mpg|rm|3gp|mp4|rmvb|avi|ra|mov|txt) /replace/down.gif [R,NC,L]  
RewriteRule .(zip) /replace/down.zip [R,NC,L]  
RewriteRule .(rar) /replace/down.rar [R,NC,L]  
RewriteRule .(7z) /replace/down.7z [R,NC,L]  
RewriteRule .(gif|jpg|psd|jpeg|bmp|ico|cur|ani) /replace/down.gif [R,NC,L]  
RewriteRule .(torrent) /replace/down.torrent [R,NC,L]  
RewriteRule .(mp3|wma|mpg|rm|3gp|mp4|rmvb|avi|ra|mov|txt) /replace/down.txt [R,NC,L] 
```

如此一来，就完成了通过apache RewriteRule模块来保护绝大多数资源不会被盗用了。这样既减少了不必要的服务器资源浪费，又可以在提醒引用你站点资源而没有通知你的使用者，何乐不为？

直接使用以上配置，需要在网站根目录建立同名的replace目录，并在里面放置你自己的防盗链替换文件：

- 例如包含版权提示和防止盗链提示的文字，图片...

不过这个方式有一个缺点，就是仅使用HTTP_REFERER进行判断，而这个判断依据可以被绕过，这个问题抽空换个更靠谱的方式来解决吧...

