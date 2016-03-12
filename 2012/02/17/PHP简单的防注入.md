# PHP简单的防注入

php实现简单的防止注入。

<!-- more -->

```php
<?php
function site_addslashes($string, $force = 0)
{
    !defined('MAGIC_QUOTES_GPC') && define('MAGIC_QUOTES_GPC', get_magic_quotes_gpc());
    if (!MAGIC_QUOTES_GPC || $force) {
        if (is_array($string)) {
            foreach ($string as $key => $val) {
                $string[$key] = addslashes($val, $force);
            }
        } else {
            $string = addslashes($string);
        }
    }
    return $string;
}

?>
```

