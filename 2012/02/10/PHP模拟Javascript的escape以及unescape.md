# PHP模拟Javascript的escape以及unescape

这个类相当好用.作用么,PHP做JSON传递GBK字符,比如中文,日文,韩文神马的Unicode最合适不过了..

<!-- more -->

```php
<?php
class coding
{
    //模仿JAVASCRIPT的ESCAPE和UNESCAPE函数的功能 
    function unescape($str)
    {
        $text = preg_replace_callback("/%u[0-9A-Za-z]{4}/", array(
            &$this,
            'toUtf8'
        ), $str);
        return mb_convert_encoding($text, "gb2312", "utf-8");
    }
    
    function toUtf8($ar)
    {
        foreach ($ar as $val) {
            $val = intval(substr($val, 2), 16);
            if ($val < 0x7F) { // 0000-007F 
                $c .= chr($val);
            } elseif ($val < 0x800) { // 0080-0800 
                $c .= chr(0xC0 | ($val / 64));
                $c .= chr(0x80 | ($val % 64));
            } else { // 0800-FFFF 
                $c .= chr(0xE0 | (($val / 64) / 64));
                $c .= chr(0x80 | (($val / 64) % 64));
                $c .= chr(0x80 | ($val % 64));
            }
        }
        return $c;
    }
    
    function escape($string, $encoding = 'gb2312')
    {
        $return = '';
        for ($x = 0; $x < mb_strlen($string, $encoding); $x++) {
            $str = mb_substr($string, $x, 1, $encoding);
            if (strlen($str) > 1) { // 多字节字符 
                $return .= '%u' . strtoupper(bin2hex(mb_convert_encoding($str, 'UCS-2', $encoding)));
            } else {
                $return .= '%' . strtoupper(bin2hex($str));
            }
        }
        return $return;
    }
    
    function gb2utf8($string, $encoding = 'utf-8', $from_encode = 'gb2312')
    {
        return mb_convert_encoding($string, $encoding, $from_encode);
    }
    
}
?>
```

google code 上找到的另外一个类似脚本

```php
<?php

        function phpescape($str)
        {
                $sublen=strlen($str);
                $retrunString="";
                for ($i=0;$i<$sublen;$i++)
                {
                        if(ord($str[$i])>=127)
                        {
                                $tmpString=bin2hex(iconv("gbk", "ucs-2", substr($str,$i,2)));
                                $tmpString=substr($tmpString,2,2).substr($tmpString,0,2);
                                $retrunString.="%u".$tmpString;
                                $i++;
                        } else {
                                $retrunString.="%".dechex(ord($str[$i]));
                        }
                }
                return $retrunString;
        }


        function escape($str) 
        {
                preg_match_all("/[\x80-\xff].|[\x01-\x7f]+/",$str,$r);
                $ar = $r[0];
                foreach($ar as $k=>$v) 
                {
                        if(ord($v[0]) < 128)
                                $ar[$k] = rawurlencode($v);
                        else
                                $ar[$k] = "%u".bin2hex(iconv("UTF-8","UCS-2",$v));
                }
                return join("",$ar);
        } 

        function phpunescape ($source) 
        { 
                $decodedStr = ""; 
                $pos = 0; 
                $len = strlen ($source); 
                
                while ($pos < $len) 
                { 
                        $charAt = substr ($source, $pos, 1); 
                        if ($charAt == '%') 
                        { 
                                $pos++; 
                                $charAt = substr ($source, $pos, 1); 
                                if ($charAt == 'u')
                                { 
                                        // we got a unicode character 
                                        $pos++; 
                                        $unicodeHexVal = substr ($source, $pos, 4); 
                                        $unicode = hexdec ($unicodeHexVal); 
                                        $entity = "&#". $unicode . ';'; 
                                        $decodedStr .= utf8_encode ($entity); 
                                        $pos += 4; 
                                }else{ 
                                        // we have an escaped ascii character 
                                        $hexVal = substr ($source, $pos, 2); 
                                        $decodedStr .= chr (hexdec ($hexVal)); 
                                        $pos += 2; 
                                } 
                        }else{ 
                                $decodedStr .= $charAt; 
                                $pos++; 
                        } 
                } 
                return $decodedStr; 
        }
        
        
        function unescape($str) 
        {
                $str = rawurldecode($str);
                preg_match_all("/(?:%u.{4})|&#x.{4};|&#\d+;|.+/U", $str, $r);
                $ar = $r[0];
                #print_r($ar);
                foreach($ar as $k=>$v) 
                {
                        if(substr($v,0,2) == "%u")
                                $ar[$k] = iconv("UCS-2", "UTF-8", pack("H4",substr($v,-4)));
                        elseif(substr($v,0,3) == "&#x")
                                $ar[$k] = iconv("UCS-2", "UTF-8", pack("H4",substr($v,3,-1)));
                        elseif(substr($v,0,2) == "&#") 
                        {
                                //echo substr($v,2,-1)."";
                                $ar[$k] = iconv("UCS-2", "UTF-8", pack("n",substr($v,2,-1)));
                        }
                }
                return join("",$ar);
        }

?>
```


