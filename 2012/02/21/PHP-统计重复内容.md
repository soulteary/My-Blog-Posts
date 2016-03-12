# [PHP]统计重复内容

```php
<?php
	$array=array(1,2,3,4,5,6,8,5,2,3,6,3,5,2,3,6,5,2,2);
	print_r($array);
	echo '<hr>';
	$b=array_count_values($array);//统计重复值
	foreach($b as $key=>$value){
		if($value>1){
			echo '重复值'.'<font color=red>'.$key.'</font>'.'---------'.'重复次数'.$value.'<br>';
		}
	}
?>
```


