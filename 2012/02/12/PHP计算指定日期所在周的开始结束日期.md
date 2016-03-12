# PHP计算指定日期所在周的开始结束日期

php获取指定日期所在周的开始和结束日期，或者说如何用PHP来获取某个星期的日期范围。

<!-- more -->

```php
<?php
function wholeweek($gdate = "", $first = 0)
{
    if (!$gdate)
        $gdate = date("Y-m-d");
    $w  = date("w", strtotime($gdate));
    $dn = $w ? $w - $first : 6;
    $st = date("Y-m-d", strtotime("$gdate -" . $dn . " days"));
    $en = date("Y-m-d", strtotime("$st +6 days"));
    return array(
        $st,
        $en
    );
}

print_r (wholeweek('2012-02-11'));
//Array ( [0] => 2012-02-05 [1] => 2012-02-11 )
?>
```

