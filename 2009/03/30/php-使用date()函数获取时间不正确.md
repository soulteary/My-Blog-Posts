# [php]使用date()函数获取时间不正确

PHP使用date()函数获取时间不正确
在PHP中使用

```php
<?php
date(Y/n/d H:i)
?>
```

获取时间于真实时间不同的原因可能是

1. 没有修改php.ini的配置文件中的默认时区为当前的时区位置
  - 解决方法：修改php.ini：将date.timezone项修改为date.timezone = PRC
2. 没有在程序中初始化时区
  - 解决方法：在程序中声明时区：`<?php date_default_timezone_set('PRC');?>`


