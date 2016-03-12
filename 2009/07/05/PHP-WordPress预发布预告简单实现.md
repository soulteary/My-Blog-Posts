# [PHP]WordPress预发布预告简单实现

最近生病&有点忙,便没有更新,闲的时候想起来MG有展示过一个预发布状态...[为了不抢MG的生意,我没做成插件,其实稍微了解PHP语法的朋友,稍加修改可以做成带有定制标题,时间显示,文章标题修饰的插件...]

使用方法:在任意WP PHP主题脚本中使用fir_recent_future()即可,效果如下：

[![预发布预告预览](https://attachment.soulteary.com/2009/07/05/2009-07-05_105046.png "预发布预告预览")](https://attachment.soulteary.com/2009/07/05/2009-07-05_105046.png)

<!-- more -->

```php

<?php

/**
*Function Name: 输出预发布文章
**/
function fir_recent_future( $futures = false ) {
if ( !$futures ) {
$futures_query = new WP_Query( array(
'post_type' => 'post',
'post_status' => 'future',
'posts_per_page' => 5,
'orderby' => 'post_date',
'order' => 'ASC'
) );
$futures =& $futures_query->posts;
}

if ( $futures && is_array( $futures ) ) {
$list = array();
foreach ( $futures as $future ) {
$url = get_edit_post_link( $future->ID );
$post_id=$future->ID;
$title = get_the_title($post_id);
if (empty($title))
{
$title = __('(未命名)');
}
$item = '<h4><span>'.$title.'</span><span>'.get_the_time(__('Y/m/d g:i:s A'), $future).'</span></h4>';
if ( $the_content = preg_split( '#\s#', strip_tags( $future->post_content ), 11, PREG_SPLIT_NO_EMPTY ) )
$item .= '<p>' . join( ' ', array_slice( $the_content, 0, 10 ) ) . ( 10 < count( $the_content ) ? '&hellip;' : '' ) . '</p>';
$list[] = $item;
}
?>
<ul><h3>发布预告</h3>
<li><?php echo join( "</li>\n<li>", $list ); ?></li>
</ul>
<div></div><div></div>
<?php
}
}

?>
```

