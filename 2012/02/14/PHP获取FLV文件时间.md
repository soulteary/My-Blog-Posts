# PHP获取FLV文件时间

PHP如何获取FLV文件时间呢,答案是fopen文件后查看FLV文件是HEX数据,并转换为number。

```php
$flv_header_frame_length) {
        fseek($fp, $frame_size_data_length - $flv_header_frame_length, SEEK_CUR);
    }
    $duration = 0;
    while ((ftell($fp) + 1) < $flv_data_length) {
        $this_tag_header = fread($fp, 16);
        $data_length     = BigEndian2Int(substr($this_tag_header, 5, 3));
        $timestamp       = BigEndian2Int(substr($this_tag_header, 8, 3));
        $next_offset     = ftell($fp) - 1 + $data_length;
        if ($timestamp > $duration) {
            $duration = $timestamp;
        }
        fseek($fp, $next_offset, SEEK_SET);
    }
    fclose($fp);
    return $duration;
}

function get_flv_file_time($time)
{
		$time = getTime($time);
    $num = $time;
    $sec = intval($num / 1000);
    $h   = intval($sec / 3600);
    $m   = intval(($sec % 3600) / 60);
    $s   = intval(($sec % 60));
    $tm  = $h . ':' . $m . ':' . $s;
    return $tm;
}

?>
```

直接使用**get_flv_file_time("你的FLV.flv")**即可。

