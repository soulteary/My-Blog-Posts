# [WordPress]直接更换主题

RSS奇怪的一阵可以用一阵不可以用....郁闷呐...先放篇文看看有啥效果...

wordpress 主题预览太猥琐了，要预览，像是mg的主题根本不用预览，这么优秀，直接用就得了。

于是把语言禁用后，用UE搜索关键字 active..

在wp-admin 的themes.php 中找到了目标。

搜索
<!-- more -->


```php
1, 'template' => $template, 'stylesheet' => $stylesheet, 'TB_iframe' => 'true', 'width' => 600, 'height' => 400 ), $preview_link ) );
//	$preview_text = attribute_escape( sprintf( __('Preview of "%s"'), $title ) );
//	$tags = $themes[$theme_name]['Tags'];
//	$thickbox_class = 'thickbox';
*/
	$activate_link = wp_nonce_url("themes.php?action=activate&amp;template=".urlencode($template)."&amp;stylesheet=".urlencode($stylesheet), 'switch-theme_' . $template);
	$activate_text = attribute_escape( sprintf( __('Activate "%s"'), $title ) );
?>
```

