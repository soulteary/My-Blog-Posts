# PHP获取IP和地理位置

发现之前有一个脚本没有写完，简单扩充了一下。 获取IP使用的是经典的逻辑，地理定位使用的是SINA的通用接口。

使用方法详见源码：

```php
<?php
/*
 * GET LOCATION BY SINA IP INTERFACE
 *
 *              @SOULTEARY 2013.04.14
 */
class IP
{
    private $args = array();

    function __construct()
    {
        $this->args = self::init_args(func_get_args());
        $ip = self::GetIP();

        $ret = preg_match_all('/(\d+\.){3}\d+/i', $ip, $result);
        if (!$ret) {
            return false;
        } else {
            $result = $result[0];
        }

        if (isset($this->args['ONLYIP']) && $this->args['ONLYIP'] == true) {

            if (isset($this->args['FORMAT']) && $this->args['FORMAT'] == 'JSON') {
                $result = json_encode($result);
            } else {
                $result = implode(',', $result);
            }
            if (isset($this->args['ECHO']) && $this->args['ECHO'] == true) {
                echo $result;
                return true;
            } else {
                return $result;
            }
        } else {

            $apiURL = 'http://int.dpool.sina.com.cn/iplookup/iplookup.php?ip=' . $result[0];
            if (isset($this->args['FORMAT']) && $this->args['FORMAT'] == 'JSON') {
                $apiURL .= '&format=json';
                $return = $this->ipCURL($apiURL);
            } else {
                $return = $this->ipCURL($apiURL);
                $return = iconv("GBK//IGNORE", "UTF-8", $return);
            }

            if (isset($this->args['ECHO']) && $this->args['ECHO'] == true) {
                echo $return;
                return true;
            } else {
                return $return;
            }

        }

    }

    public function init_args($args)
    {
        $result = array();
        for ($i = 0, $n = count($args); $i < $n; $i++) {
            $result = self::associative_push($args[$i], $result);
        }
        return $result;
    }

    public function associative_push($arr, $tmp)
    {
        if (is_array($tmp)) {
            foreach ($tmp as $key => $value) {
                $arr[$key] = $value;
            }
            return $arr;
        }
        return false;
    }

    public function GetIP()
    {
        if (isset($_SERVER['HTTP_X_FORWARDED_FOR']) && $_SERVER['HTTP_X_FORWARDED_FOR'] && strcasecmp($_SERVER['HTTP_X_FORWARDED_FOR'], 'unknown')) {
            return $_SERVER['HTTP_X_FORWARDED_FOR'];
        } elseif (isset($_SERVER['REMOTE_ADDR']) && $_SERVER['REMOTE_ADDR'] && strcasecmp($_SERVER['REMOTE_ADDR'], 'unknown')) {
            return $_SERVER['REMOTE_ADDR'];
        }
    }

    private function ipCURL($url)
    {
        $ch = curl_init();
        curl_setopt($ch, CURLOPT_URL, $url);
        curl_setopt($ch, CURLOPT_HEADER, 0);
        ob_start();
        curl_exec($ch);
        curl_close($ch);
        $result = ob_get_contents();
        ob_end_clean();
        return $result;
    }
}

?>
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf8">
    <title>demo</title>
    <script type="text/javascript">
        var ip = "<?php new IP(array('ONLYIP'=>true, 'ECHO'=>true));?>";
        var data = <?php new IP(array('FORMAT'=>'JSON','ECHO'=>true));?>;
        var result = '';
        for (oo in data) {
            result += oo + ':' + data[oo] + "\n";
        }
        alert(result + ip);
    </script>
</head>
<body>
<h1>CODE:</h1>

<h2>GETIP</h2>

<p>'ONLYIP'=>true, 'ECHO'=>true</p>

<p><?php new IP(array('ONLYIP' => true, 'ECHO' => true));?></p>

<P>'ONLYIP'=>true, 'FORMAT'=>'JSON', 'ECHO'=>true</P>

<p><?php new IP(array('ONLYIP' => true, 'FORMAT' => 'JSON', 'ECHO' => true));?></p>

<h2>GET Location</h2>

<p>'ECHO'=>true</p>

<p><?php new IP(array('ECHO' => true));?></p>

<p>'FORMAT'=>'JSON','ECHO'=>true</p>

<p><?php new IP(array('FORMAT' => 'JSON', 'ECHO' => true));?></p>
</body>
</html>
```

