# PHP 缓存实现

如何用PHP实现缓存功能，看过这篇，你应该就明了了。

<!-- more -->

```php
<?php
function array2str($array, $level = 0)
{
    if (!is_array($array)) {
        return "'" . $array . "'";
    }
    if (is_array($array) && function_exists('var_export')) {
        return var_export($array, true);
    }
    
    $space = '';
    for ($i = 0; $i <= $level; $i++) {
        $space .= "\t";
    }
    $evaluate = "Array\n$space(\n";
    $comma    = $space;
    if (is_array($array)) {
        foreach ($array as $key => $val) {
            $key = is_string($key) ? '\'' . addcslashes($key, '\'\\') . '\'' : $key;
            $val = !is_array($val) && (!preg_match("/^\-?[1-9]\d*$/", $val) || strlen($val) > 12) ? '\'' . addcslashes($val, '\'\\') . '\'' : $val;
            if (is_array($val)) {
                $evaluate .= "$comma$key => " . array2str($val, $level + 1);
            } else {
                $evaluate .= "$comma$key => $val";
            }
            $comma = ",\n$space";
        }
    }
    $evaluate .= "\n$space)";
    return $evaluate;
}

//typeid :文章类别 
//nums：文章数量 
//len：标题截取长度 
//contentlen：内容截取长度 
//zhiding:置顶级别 
//attach:是否取附件 
function getnews($typeid = 0, $nums = 1, $len = 30, $contentlen = 100, $zhiding = 0, $attach = 0)
{
    global $xy_db;
    $len  = 100;
    $mkey = md5($typeid . $nums . $len . $contentlen . $zhiding . $attach);
    if (intval($nums) < 1)
        $nums = 1;
    $path = SITE_ROOT_PATH . './temp/cache/' . $mkey . '.php';
    if (is_file($path)) {
        @require_once($path);
        if (($expiration + 60) >= time()) {
            if (is_array($cachenews)) {
                $j = 0;
                for ($i = 0; $i < count($cachenews); $i++) {
                    if ($j < $nums) {
                        $data[] = $cachenews[$i];
                        $j++;
                    }
                }
                return $data;
            }
        }
    }
    $time = time();
    if ($typeid) {
        $cd .= " and type_id='$typeid'";
    }
    if ($zhiding) {
        $cd .= " and level='$zhiding' ";
    }
    if ($attach) {
        $cd .= " and attach_num>0 ";
    }
    
    if (empty($typeid)) {
        $cd .= " and (xy_id='" . XY_ID . "' or xy_id=0) ";
    } else {
        $cd .= " and xy_id='" . XY_ID . "' ";
    }
    
    $query     = $xy_db->query("select * from xy_news where 1 $cd 
     order by orders desc, id desc limit 0,$nums ");
    $cachenews = array();
    $i         = 0;
    $j         = 0;
    while ($a = $xy_db->fetch_array($query)) {
        if ($attach) {
            $r           = $xy_db->fetch_first("select * from xy_news_attach where news_id='$a[id]' and is_image=1 limit 0,1");
            $a['picurl'] = $r['file_url'];
        }
        $a['addtime']     = date('m-d', $a['addtime']);
        $a['content']     = cutstr(strip_tags($a['content']), $contentlen);
        $a['fullsubject'] = $a['subject'];
        $a['linkurl']     = 'news_detail.html?id=' . $a['id'];
        $cachenews[$i]    = $a;
        $data1[]          = $cachenews[$i];
        if ($i < $nums) {
            $cachenews[$i]['subject'] = cutstr($cachenews[$i]['subject'], $len);
            $data[]                   = $cachenews[$i];
        }
        $i++;
    }
    if ($cachenews) {
        $cachenews = '<?php '; 
        $cachenews .= '$expiration = ''.time().'\';'."\n\n"; 
        $cachenews .= "\$cachenews = ".array2str($data1).";\n\n"; 
        $cachenews .= ' ?>'; 
        if ($fp = @fopen($path, 'w')) {
            @flock($fp, 2);
            fwrite($fp, $cachenews);
            fclose($fp);
        }
    }
    return $data;
}
?>
```


