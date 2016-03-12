# PHP模拟登陆

PHP如何模拟登录呢,这里有个简单的例子。

<!-- more -->

```php
<?php
function GetWebContent($host, $method, $str, $sessid = '')
{
    $ip = gethostbyname($host);
    $fp = @fsockopen($ip, 80);
    if (!$fp)
        return;
    fputs($fp, "$method\r\n");
    fputs($fp, "Host: $host\r\n");
    if (!empty($sessid)) {
        fputs($fp, "Cookie: PHPSESSID=$sessid; path=/;\r\n");
    }
    if (substr(trim($method), 0, 4) == "POST") {
        fputs($fp, "Content-Length: " . strlen($str) . "\r\n");
    }

    fputs($fp, "Content-Type: application/x-www-form-urlencoded\r\n");
    fputs($fp, "User-Agent: Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1; InfoPath.1)\r\n)");
    fputs($fp, "Connection: Keep-Alive\r\n\r\n");
    if (substr(trim($method), 0, 4) == "POST") {
        fputs($fp, $str . "\r\n");
    }
    while (!feof($fp)) {
        $response .= fgets($fp);
    }
     // LINUX下是 "\n\n" 
    $hlen   = strpos($response, "\r\n\r\n");
    $header = substr($response, 0, $hlen);

    $entity = substr($response, $hlen + 4);
    if (preg_match('/PHPSESSID=([0-9a-z]+);/i', $header, $matches)) {
        $a['sessid'] = $matches[1];
    }
    if (preg_match('/Location: ([0-9a-z\_\?\=\&\#\.]+)/i', $header, $matches)) {
        $a['location'] = $matches[1];
    }
    $a['content'] = $entity;
    fclose($fp);
    return $a;
}
?>
```

使用方法:

```php

<?php
$host       = "http://你的网站";
$login_page = "/登录页面.php";
$str        = "act=add&user=user";
 //登入得到新的session_id 
$response   = GetWebContent("$host", "POST /$login_page HTTP/1.0", $str);
//使用session_id访问页面
//$response = GetWebContent("$host","GET /$login_page HTTP/1.0", '', $response['sessid']); 
//print_r($response); 
exit;

?>
```

