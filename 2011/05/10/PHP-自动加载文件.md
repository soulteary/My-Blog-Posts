# [PHP]自动加载文件

[![php](https://attachment.soulteary.com/common/sablog-post-headline/php.gif "php")](https://attachment.soulteary.com/common/sablog-post-headline/php.gif) 


php自动加载文件,某些时候很有用,比如你写了一堆类,挨个添加 require的时候..你会想死... 

so,现在介绍2种加载方法,第一种是使用php自动加载函数。  

这个函数php手册有提到，[点击浏览](http://www.php.net/manual/zh/language.oop5.autoload.php)。 

emlog的作者很有才,这个方法很犀利.
```php
<?php
function __autoload($class) {
	$path = somepath;
	$class = strtolower($class);
    if (file_exists($path. $class . '.php')) {
        require_once($path. $class . '.php');
    } else{
    	die($class.'加载失败。');
    }
}
?>
```

第二种使用sql加载的方法，给出2个链接。 


[博客园](http://www.cnblogs.com/yuxing/archive/2010/06/19/1760742.html) [脚本之家](http://cache.baidu.com/c?m=9d78d513d9811bed4fede5697b13c0101f43f0672ba4a4027ea48438e2732d405010e5ac51280443939b733d47e90b4beb832b6f724665a09bbfd01adab8852858d470726d4fc607498247f8d64624ca27944de9d843a1e5&p=91759a45d39612a05ff6de121c4b&user=baidu&fm=sc&query=php+%D7%D4%B6%AF%BC%D3%D4%D8&qid=e0059cc410e7bd84&p1=3)

