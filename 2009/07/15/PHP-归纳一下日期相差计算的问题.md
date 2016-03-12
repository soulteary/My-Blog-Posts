# [PHP]归纳一下日期相差计算的问题

好多新人应该都有做过php计时器，或者说php倒计时,php计时器吧。
记得以前每个网站都有一个牌子，写着本站运行多少天...
就连文曲星上也有一个类似的功能..
下面我给出3段代码，方便需要的朋友，
实现日期计算最少有3种方法，但是说到原理就是一种而已，转换。
第一种效率相比内置函数似乎有点低，不过能够说明问题，
第二种是纯粹的内置函数，具体可见php手册。
第三种是数组分离时间参数，进行累加计算。


推荐使用第二种方法，如果你有兴趣可以自己完成最后一种，别忘记我的提示，要加一哦~

```php
<?php//计算今天和从前某天相差多少天
$date_begin = date(&quot;Y-m-d&quot;);
$date_end   = &quot;2001-01-01&quot;;
$date_rtn   = round((strtotime($date_begin)-strtotime($date_end))/3600/24);

//计算今天到未来某天相差多少天
$date_begin = &quot;2049-12-31&quot;;
$date_end   = date(&quot;Y-m-d&quot;);
$date_rtn   = round((strtotime($date_end)-strtotime($date_begin))/3600/24);

//归纳一下,其实计算的话,可以有两种思路
//1.时间有先后,把后面的时间永远放置到$date_end即可

function fir_datediff($date_begin,$date_end)
{
	$date_rtn   = round((strtotime($date_end)-strtotime($date_begin))/3600/24);
	return $date_rtn;
}

//等效的是系统函数是
function php_datediff($date_end,$date_begin)
{
	$date_rtn = datediff($date_end,$date_begin);
	return $date_rtn;
}

//2.或者可以这么想，不管是哪天差哪天，都是一样的数字吧，符号不同罢了。

function fir_datediff($date_begin,$date_end)
{
	$date_rtn   = round(abs((strtotime($date_end)-strtotime($date_begin))/3600/24));
	return $date_rtn;
}


//等效的是系统函数是
function php_datediff($date_end,$date_begin)
{
	$date_rtn = abs(datediff($date_end,$date_begin));
	return $date_rtn;
}

//其实还有一种方法,不过比较累人了..
explode 分离日期到数组，计算日期天数差，最后结果加1
?>
```

