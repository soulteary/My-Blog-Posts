# [PHP]中文内容截断函数

最近在完善主题，发现中文截断还是有点小问题。改进一下。放在这里以后似乎还能用吧？

```php
<?php
function the_content_strip($content,$len)
{

	$content = preg_replace('@<!--
]*?>.*?
// -->@si', '', $content);
	$content = preg_replace('@<!--
]*?>.*?
-->@si', '', $content);
	$content = mb_strimwidth($content,0,$len,'...','utf-8');
	$content = strip_tags($content);

	return $content;
}
?>
```


