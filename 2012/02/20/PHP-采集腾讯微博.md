# [PHP]采集腾讯微博

```php
<?php
header("Content-type:text/html;charset=utf-8");
$weibo = file_get_contents('http://t.qq.com/soulteary');

$preg = '/<div class="msgCnt">(.*)<\/div><div class="mediaWrap">/Uis';
preg_match_all($preg, $weibo, $string);

foreach ($string[1] as $key=>$value){
	echo delhtml($value)."<br/><br/><br/>";
}

function delhtml($str) // 清除HTML标签
{
	$st = -1; //开始
	$et = -1; //结束
	$stmp = array();
	$stmp[] = "&nbsp;";
	$len = strlen($str);
	for($i = 0;$i < $len;$i++)
	{
		$ss = substr($str, $i, 1);
		if (ord($ss) == 60) // ord("<")==60
		{
			$st = $i;
		}
		if (ord($ss) == 62) // ord(">")==62
		{
			$et = $i;
			if ($st != -1)
			{
				$stmp[] = substr($str, $st, $et - $st + 1);
			}
		}
	}
	$str = str_replace($stmp, "", $str);
	return $str;
}

?>
```


