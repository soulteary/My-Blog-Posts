# MimeTypes Error

更新了一个代码高亮插件,发现提示如上... 显而易见的mime type在服务器中没有定义,浏览器接受的时候,是显示为文件流... 然后搜索了一下对应的MIME type的定义,在DA中添加了一下.. 其实在htaccess中也是可以添加的..但是htaccess规则太多真心不好啊...

<!-- more -->

```text
Resource interpreted as Font but transferred with MIME type application/octet-stream: "some url path/monaco-webfont.woff".
```

```text
application/vnd.ms-fontobject	eot	
application/x-font-ttf	ttf	
font/opentype	otf	
font/x-woff	woff
```

顺便贴一下nginx中的解决方法. http://serverfault.com/questions/186965/how-can-i-make-nginx-support-font-face-formats-and-allow-access-control-allow-o

