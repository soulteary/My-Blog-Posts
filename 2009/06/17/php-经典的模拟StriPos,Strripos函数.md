# [php]经典的模拟StriPos,Strripos函数

由于Php 4.x 版本不支持stripos 、 strripos，所以如果要在4.x中实现这两个函数，需要使用下面的方法模拟实现。

```php
<?php

if(!function_exists("stripos"))
{
 function stripos($str,$needle,$offset=0)
 {
  return strpos(strtolower($str),strtolower($needle),$offset);
 }
}

if(!function_exists("strripos"))
{
 function strripos($haystack,$needle,$offset=0)
 {
  if(!is_string($needle))$needle=chr(intval($needle));
  if($offset&lt;0)
  {
   $temp_cut=strrev(substr($haystack,0,abs($offset)));
  }
  else
  {
   $temp_cut=strrev(substr($haystack,0,max((strlen($haystack)-$offset),0)));
  }
  if(($found=stripos($temp_cut,strrev($needle)))===false)return false;
  $pos=(strlen($haystack)-($found+$offset+strlen($needle)));
  return $pos;
 }
}

?>
```

