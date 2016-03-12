# PHP如何伪造IP

如何用PHP实现伪造IP呢,这里有一个简单的例子。

```php
<?php
$address = 'http://要访问的地址';
$myip    = '您的IP地址';

ob_start();
$mtime     = explode(' ', microtime());
$starttime = $mtime[1] + $mtime[0];

$onoff = (function_exists('ini_get')) ? ini_get('register_globals') : get_cfg_var('register_globals');

if ($onoff != 1) {
    @extract($_POST, EXTR_SKIP);
    @extract($_GET, EXTR_SKIP);
}

$self      = $_SERVER['PHP_SELF'];
$myaddress = 'http://' . $_SERVER['HTTP_HOST'] . $self;

$comm = '';

if (isset($url) && $url) {
    $url = str_replace($address, '', $url);
}

$pcomm = '';
foreach ($_POST as $key => $val) {
    if ($pcomm == '') {
        $pcomm .= $key . '' . urlencode($val);
    } else {
        $pcomm .= $pcomm . '&' . $key . '=' . urlencode($val);
    }
}

foreach ($_GET as $key => $val) {
    if ($key != 'url') {
        if ($comm == '') {
            $comm = $key . '=' . rawurlencode($val);
        } else {
            $comm = $comm . '&' . $key . '=' . rawurlencode($val);
        }
    }
}

if (!$url) {
    $url = $address;
} else {
    $url = $address . $url;
    if ($comm) {
        if (strstr($url, '?')) {
            $url = $url . '&' . $comm;
        } else {
            $url = $url . '?' . $comm;
        }
    }
}

if ($url) {
    $cookies = '';
    if (count($_COOKIE)) {
        foreach ($_COOKIE as $cookie_name => $cookie_var) {
            $cookies .= $cookies != '' ? '; ' . $cookie_name . '=' . $cookie_var : $cookie_name . '=' . $cookie_var;
        }
    }
    
    $temp             = @parse_url($url);
    $temp['port']     = isset($temp['port']) ? $temp['port'] : 80;
    $temp['path']     = isset($temp['path']) ? $temp['path'] : '/';
    $temp['file']     = substr($temp['path'], strrpos($temp['path'], '/') + 1);
    $temp['dir']      = substr($temp['path'], 0, strrpos($temp['path'], '/'));
    $temp['base']     = $temp['scheme'] . '://' . $temp['host'] . ($temp['port'] != 80 ? ':' . $temp['port'] : '') . $temp['dir'];
    $temp['prev_dir'] = $temp['path'] != '/' ? substr($temp['base'], 0, strrpos($temp['base'], '/') + 1) : $temp['base'] . '/';
    $fp               = @fsockopen($temp['host'], $temp['port'], $errno, $errstr, 30);
    if ($fp) {
        if ($_SERVER['REQUEST_METHOD'] != 'POST') {
            @fputs($fp, "GET $temp[path]?$temp[query] HTTP/1.1\r\n");
        } else {
            @fputs($fp, "POST $temp[path]?$temp[query] HTTP/1.1\r\n");
        }
        @fputs($fp, "Host: $temp[host]\r\n");
        @fputs($fp, "Accept: */*\r\n");
        @fputs($fp, "Referer: [url]http://$temp[url][host]/\r\n");
        @fputs($fp, "Cookie: $cookies\r\n");
        @fputs($fp, "User-Agent: Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1)\r\n");
        @fputs($fp, "via: 1.1 JEJE1:80 (squid/2.5.STABLE4-NT-CVS)\r\n");
        @fputs($fp, "X-Forwarded-For: $myip\r\n");
        if ($_SERVER['REQUEST_METHOD'] == 'POST') {
            @fputs($fp, "Content-Type: application/x-www-form-urlencoded\r\n");
            @fputs($fp, "Content-Length: " . strlen($pcomm) . "\r\n\r\n");
            @fputs($fp, $pcomm);
        }
        @fputs($fp, "Connection: Close\r\n\r\n");
        
        while ($str = @fread($fp, 4096)) {
            if ($str != "\r\n" && preg_match_all("#set-cookie:([^\r\n]*)#i", $str, $matches)) {
                foreach ($matches[1] as $cookie_info) {
                    preg_match('#^\s*([^=;,\s]*)=?([^;,\s]*)#', $cookie_info, $match) && list(, $name, $value) = $match;
                    preg_match('#;\s*expires\s*=([^;]*)#i', $cookie_info, $match) && list(, $expires) = $match;
                    $expires = isset($expires) ? strtotime($expires) : false;
                    $expires = (!is_numeric($expires) || time() > $expires) ? false : $expires;
                    setcookie($name, $value, $expires);
                }
                $str = str_replace($matches[0], '', $str);
            }
            
            $Content .= $str;
        }
        @fclose($fp);
        if (strpos($Content, 'Content-Type: text/html')) {
            $Content = substr($Content, strpos($Content, 'Content-Type: text/html') + 33);
        } else {
            $Content = substr($Content, strpos($Content, chr(0x0d) . chr(0x0a) . chr(0x0d) . chr(0x0a)) + 4);
        }
        

        
        echo $Content;
    }
}

?>

```

