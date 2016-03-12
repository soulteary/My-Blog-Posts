# [PHP]header技巧一则

最近看日志又有小白用工具乱扫了...

想htaccess -d -f直接定义到某php脚本上.

很老的技巧了。

脚本内容如下:

```php
<?php
if(conditions){
	header('HTTP/1.1 500 Internal Server Error');
	die();
}
?>
```

conditions可以包含很多东西，来路，ua，ip，时间，方式，等。

