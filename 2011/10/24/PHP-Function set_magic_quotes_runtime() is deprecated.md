# [PHP]Function set_magic_quotes_runtime() is deprecated

今天很不淡定,被我敲打了2年多的键盘撒手人寰.以前网吧看着热血青年干架的双飞燕啊,你肿么能这个样子。

说正题。

**Deprecated: Function set_magic_quotes_runtime() is deprecated** 

当你的网站出现这个内容，说明你程序中使用了当前版本PHP废弃的函数。

<!-- more -->

可以选择用@忽略警告，或set_ini 函数修改变量 0,1 开关，但是似乎都挺麻烦，一个要一条一条的修改，一个不一定生效，或许和重启服务以及缓存有关？

修改php.ini,将 error_reporting = E_ALL 或 error_reporting = E_ALL & ~E_NOTICE 修改为 error_reporting = E_ALL & ~E_NOTICE & ~E_DEPRECATED 即可。

如果还是不行的话，下面的函数可以实现相同效果。


```php
<?php
//######################################################################################
// deal with magic_quotes nastiness in GPC data
if (get_magic_quotes_gpc())
{
    function exec_gpc_stripslashes(&$arr)
    {
        if (is_array($arr))
        {
            foreach($arr AS $_arrykey => $_arryval)
            {
                if (is_string($_arryval))
                {
                    $arr["$_arrykey"] = stripslashes($_arryval);
                }
                else if (is_array($_arryval))
                {
                    $arr["$_arrykey"] = exec_gpc_stripslashes($_arryval);
                }
            }
        }
        return $arr;
    }

    $_GET = exec_gpc_stripslashes($_GET);
    $_POST = exec_gpc_stripslashes($_POST);
    $_COOKIE = exec_gpc_stripslashes($_COOKIE);
    if (is_array($_FILES))
    {
        foreach ($_FILES AS $key => $val)
        {
            $_FILES[$key]['tmp_name'] = str_replace('\', '', $val['tmp_name']);
        }
    }
    $_FILES = exec_gpc_stripslashes($_FILES);
    $_REQUEST = array_merge($_GET, $_POST, $_COOKIE);
}
set_magic_quotes_runtime(0);
?>
```

