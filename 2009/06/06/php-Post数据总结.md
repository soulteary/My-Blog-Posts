# [php]Post数据总结

文章出处：http://hi.baidu.com/qfans/blog/item/33896c81dedc41d0bd3e1e71.html 

似乎上面的兄弟也是从网上搜集来的...出处未知。 

但是总结还是挺全的，除了使用管道或者调用其他程序的例子~

```php
<?php 
/** 
* Socket版本 
* 使用方法： 
* $post_string = "app=socket&version=beta"; 
* request_by_socket('facebook.cn','/restServer.php',$post_string); 
*/ 
function request_by_socket($remote_server,$remote_path,$post_string,$port = 80,$timeout = 30){ 
    $socket = fsockopen($remote_server,$port,$errno,$errstr,$timeout); 
    if (!$socket) die("$errstr($errno)"); 
    
    fwrite($socket,"POST $remote_path HTTP/1.0\r\n"); 
    fwrite($socket,"User-Agent: Socket Example\r\n"); 
    fwrite($socket,"HOST: $remote_server\r\n"); 
    fwrite($socket,"Content-type: application/x-www-form-urlencoded\r\n"); 
    fwrite($socket,"Content-length: ".strlen($post_string)+8."\r\n"); 
    fwrite($socket,"Accept:*/*\r\n"); 
    fwrite($socket,"\r\n"); 
    fwrite($socket,"mypost=$post_string\r\n"); 
    fwrite($socket,"\r\n"); 
    
    $header = ""; 
    while ($str = trim(fgets($socket,4096))) { 
        $header.=$str; 
    } 
    
    $data = ""; 
    while (!feof($socket)) { 
        $data .= fgets($socket,4096); 
    } 
    
    return $data; 
}


/** 
* Curl版本 
* 使用方法： 
* $post_string = "app=request&version=beta"; 
* request_by_curl('http://facebook.cn/restServer.php',$post_string); 
*/ 
function request_by_curl($remote_server,$post_string){ 
    $ch = curl_init(); 
    curl_setopt($ch,CURLOPT_URL,$remote_server); 
    curl_setopt($ch,CURLOPT_POSTFIELDS,'mypost='.$post_string); 
    curl_setopt($ch,CURLOPT_RETURNTRANSFER,true); 
    curl_setopt($ch,CURLOPT_USERAGENT,"Jimmy's CURL Example beta"); 
    $data = curl_exec($ch); 
    curl_close($ch); 
    return $data; 
} 
/** 
* 其它版本 
* 使用方法： 
* $post_string = "app=request&version=beta"; 
* request_by_other('http://facebook.cn/restServer.php',$post_string); 
*/ 
function request_by_other($remote_server,$post_string){ 
    $context = array( 
        'http'=>array( 
            'method'=>'POST', 
            'header'=>'Content-type: application/x-www-form-urlencoded'."\r\n". 
                      'User-Agent : Jimmy\'s POST Example beta'."\r\n". 
                      'Content-length: '.strlen($post_string)+8, 
            'content'=>'mypost='.$post_string) 
        ); 
    $stream_context = stream_context_create($context); 
    $data = file_get_contents($remote_server,FALSE,$stream_context); 
    return $data; 
}

?>
```

