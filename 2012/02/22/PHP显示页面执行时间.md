# PHP显示页面执行时间

之前有个版本:http://promiseforever.com/2012/02/07/php-show-run-time.html

现在来一个更加迷你的版本..

```php
<?php
//这里是页面执行内容
echo microtime(TRUE) - $_SERVER['REQUEST_TIME'];
?>
```

