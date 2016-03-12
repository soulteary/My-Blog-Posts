# [php]重定向网页地址

```php
//如果觉得俺的总结不错,就收藏了这个网页吧,www.promiseforever.com
//方法A：PHP内置函数,此方法支持几乎所有的设备[包括手机]语句前不能有HTML输出。
<?php
$GoURL="目标地址";
header(sprintf("Location: %s", $GoURL));
?>
?
//方法B：HTML元标记重定向,此方法支持可解读元标记的设备[部分手机不支持]
<?php echo "<META HTTP-EQUIV="Refresh" CONTENT="0;URL=index.php">";?>
?
//方法C：JS脚本重定向,对于禁用JS脚本以及不支持JS脚本功能的设备,此方法无效。
<?php echo "<script>window.location ="$PHP_SELF";</script>";?>
```

