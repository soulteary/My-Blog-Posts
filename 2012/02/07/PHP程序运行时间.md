# PHP程序运行时间


常常看到页面执行完毕,底下有一行,程序执行时间XXX吧.有的时候,用来比较某些函数运行效率,这也不失为一个好办法。

放2种方法,一种是简单的语句模块,一种是简单的类。

<!-- more -->

这种很简单,适合在几个简单的页面内使用.如果结果太小,那么就调整循环的数量

```php
<?php
$time_start = getmicrotime(); 
function getmicrotime()
{
list($usec, $sec) = explode(" ",microtime());
return ((float)$usec + (float)$sec);
}

//待测试代码段
//for ($i=0 ;$i<100; $i++) {


//}
//待测试代码段


$time_end = getmicrotime();
printf ("[页面执行时间: %.2f毫秒]\n\n",($time_end - $time_start)*1000);

?>
```

这个类的好处就是可以安全的调用,而不必担心变量被修改

这个类的出处:[这里](http://promiseforever.com/redirect?url=http%3A%2F%2Fwww.CodeBit.cn&key=c6306f6c6c13aaef84cb9d0b74f0a013)>

```php
<?php
class timer
{
    var $StartTime = 0;
    var $StopTime = 0;
 
    function get_microtime()
    {
        list($usec, $sec) = explode(' ', microtime());
        return ((float)$usec + (float)$sec);
    }
 
    function start()
    {
        $this->StartTime = $this->get_microtime();
    }
 
    function stop()
    {
        $this->StopTime = $this->get_microtime();
    }
 
    function spent()
    {
        return round(($this->StopTime - $this->StartTime) * 1000, 1);
    }
 
}
 
/*
//例子
$timer = new timer;
$timer->start();
 
//你的代码开始
 
$a = 0;
for($i=0; $i<1000000; $i++)
{
    $a += $i;
}
 
//你的代码结束
 
$timer->stop();
echo "页面执行时间: ".$timer->spent()." 毫秒";
*/
?>
```

