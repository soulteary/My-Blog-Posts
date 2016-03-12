# [WordPress]highslide4wp 插件补充

[![hs4wp](https://attachment.soulteary.com/2009/05/11/hs4wp.png "hs4wp")](https://attachment.soulteary.com/2009/05/11/hs4wp.png) 

http://www.neoease.com/highslide4wp/ MG的这个插件很不赖，对于俺来说，如果显示图的话，我宁可用lightbox，萝卜青菜各有所爱。

但是表情用这个来show很炫滴... 似乎有些人出现了这个问题吧。

<!-- more -->

```text
Warning: Invalid argument supplied for foreach() in wp-content/plugins/highslide4wp/toys.php on line 8
```

很不幸的是，我也遇到了这个问题...看似是数据类型不正确...去PK这个变量吧 于是将修改为但是发现还是有问题... 翻看wp的表情转换函数，

```php
<!--?php 
function smilies_init() {
global $wpsmiliestrans, $wp_smiliessearch, $wp_smiliesreplace;

// don't bother setting up smilies if they are disabled
if ( !get_option( 'use_smilies' ) )
return;

if ( !isset( $wpsmiliestrans ) ) {
$wpsmiliestrans = array(
':mrgreen:' =--> 'icon_mrgreen.gif',
':neutral:' =&gt; 'icon_neutral.gif',
':twisted:' =&gt; 'icon_twisted.gif',
':arrow:' =&gt; 'icon_arrow.gif',
':shock:' =&gt; 'icon_eek.gif',
':smile:' =&gt; 'icon_smile.gif',
':???:' =&gt; 'icon_confused.gif',
':cool:' =&gt; 'icon_cool.gif',
':evil:' =&gt; 'icon_evil.gif',
':grin:' =&gt; 'icon_biggrin.gif',
':idea:' =&gt; 'icon_idea.gif',
':oops:' =&gt; 'icon_redface.gif',
':razz:' =&gt; 'icon_razz.gif',
':roll:' =&gt; 'icon_rolleyes.gif',
':wink:' =&gt; 'icon_wink.gif',
':cry:' =&gt; 'icon_cry.gif',
':eek:' =&gt; 'icon_surprised.gif',
':lol:' =&gt; 'icon_lol.gif',
':mad:' =&gt; 'icon_mad.gif',
':sad:' =&gt; 'icon_sad.gif',
'8-)' =&gt; 'icon_cool.gif',
'8-O' =&gt; 'icon_eek.gif',
':-(' =&gt; 'icon_sad.gif',
':-)' =&gt; 'icon_smile.gif',
':-?' =&gt; 'icon_confused.gif',
':-D' =&gt; 'icon_biggrin.gif',
':-P' =&gt; 'icon_razz.gif',
':-o' =&gt; 'icon_surprised.gif',
':-x' =&gt; 'icon_mad.gif',
':-|' =&gt; 'icon_neutral.gif',
';-)' =&gt; 'icon_wink.gif',
'8)' =&gt; 'icon_cool.gif',
'8O' =&gt; 'icon_eek.gif',
':(' =&gt; 'icon_sad.gif',
':)' =&gt; 'icon_smile.gif',
':?' =&gt; 'icon_confused.gif',
':D' =&gt; 'icon_biggrin.gif',
':P' =&gt; 'icon_razz.gif',
':o' =&gt; 'icon_surprised.gif',
':x' =&gt; 'icon_mad.gif',
':|' =&gt; 'icon_neutral.gif',
';)' =&gt; 'icon_wink.gif',
':!:' =&gt; 'icon_exclaim.gif',
':?:' =&gt; 'icon_question.gif',
);
}

$siteurl = get_option( 'siteurl' );
foreach ( (array) $wpsmiliestrans as $smiley =&gt; $img ) {
$wp_smiliessearch[] = '/(\s|^)' . preg_quote( $smiley, '/' ) . '(\s|$)/';
$smiley_masked = attribute_escape( trim( $smiley ) );
$wp_smiliesreplace[] = "<img class="wp-smiley" src="$siteurl/wp-includes/images/smilies/$img" alt="$smiley_masked" />";
}
}
?>
```

看到下面的句子了吧，如果没有打开表情自动转换的话...那么你的数组=Null...自然报错如果你还是不想打开表情自动转换的话，那么将添加到出错的函数内...使用的是没问题了，但是呢，评论表情是不会转换的。 如果你使用MG的这个主题的话，那么这样修改一下吧[comments.php文件]如果你只想使用这个炫炫的表情展开特效，可以这么做 替换highslide4wp.php内的函数

```php
<!--?php 
function highslide_emoticons($static = false) {
global $wpsmiliestrans;
$closeAction = $static ? '' : 'return hs.close(\'emoticons\');';
$siteurl = get_option( 'siteurl' );
$emoticons = '';
$smiled = array();
foreach ((array) $wpsmiliestrans as $tag =--> $grin) {
if (!in_array($grin, $smiled)) {
$smiled[] = $grin;
$tag = str_replace(' ', '', $tag);
$emoticons .= '<a style="margin-right: 5px;" onclick="insertEmoticon(\' '.$tag.' \');' . $closeAction . '" href="javascript:void(0);"><img src="'.$siteurl.'/wp-includes/images/smilies/'.$grin.'" alt="'.$tag.'" />'; } } $highslide_emoticon = '<script type="text/javascript" src="'.$siteurl.'/wp-content/plugins/highslide4wp/js/emoticons.js"></script></a><a class="highslide" title="选择一个表情" onclick="return hs.htmlExpand(this, { contentId: \'emoticons\' } )" href="javascript:void(0);"><img src="'.$siteurl.'/wp-includes/images/smilies/icon_smile.gif" alt="选择一个表情" /></a>';</pre>
<div id="emoticons" class="highslide-html-content" style="width: 200px;">
<div class="highslide-body">$highslide_emoticon .= $emoticons; $highslide_emoticon .= '</div>
<div class="highslide-header">
<ul>
	<li class="highslide-close"><a onclick="return hs.close(this)" href="#">关闭</a></li>
</ul>
</div>
</div>
<pre lang="php">';
 echo $highslide_emoticon;
}

function highslide_head() {
 $siteurl = get_option( 'siteurl' );
 print('<script type="text/javascript" src="'.$siteurl.'/wp-content/plugins/highslide4wp/highslide/highslide-with-html.packed.js"></script><script type="text/javascript">// <![CDATA[
	hs.graphicsDir = "'.$siteurl.'/wp-content/plugins/highslide4wp/highslide/graphics/";
	hs.outlineType = "rounded-white";
	hs.outlineWhileAnimating = true;
	hs.showCredits = false;
// ]]></script>');
}
add_action('wp_head', 'highslide_head');

?>
```


