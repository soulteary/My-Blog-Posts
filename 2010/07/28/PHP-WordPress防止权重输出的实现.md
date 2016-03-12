# [PHP]WordPress防止权重输出的实现

转载请保留出处，谢谢合作。

其实这个东西很早以前就做完了，如果细细追究时间的话，应该是09年7月16日，那个时候好像DZ还没有流行这个方法。

要是怎么防止权重输出，其实很简单，链接替换和链接重定向。

首先看个例子，比如，某君和我做链接之后，在月黑风高夜，悄悄的给我去掉了。[说起这个我想不少朋友都遇到过吧。]

于是，礼尚往来，我们在保留链接的情况下，稍微做点手脚。

<!-- more -->

假设之前和我做链接的朋友的地址是：http://www.qq.com，

在我稍作手脚后，地址就变成了：[http://promiseforever.com/redirect?&url=http://www.qq.com&key=84c80d1640dc544b9fc92e29ee7af002](http://promiseforever.com/redirect?&url=http://www.qq.com&key=84c80d1640dc544b9fc92e29ee7af002) 

有的同学，或许还在疑问，这样就怎么了呢，如果有疑问，不妨点击一下看看。提示，点击图片浏览大图。 

[![网站重定向结果](https://attachment.soulteary.com/2010/07/28/redirect.png "网站重定向结果")](https://attachment.soulteary.com/2010/07/28/redirect.png) 

是不是出现了图片所示的页面“您访问的页面安全，请点击此处继续”云云。 

我们来模拟一下蜘蛛的抓取吧，机器人跟踪到这里的时候，首先链接变成了站内的内链，然后跳转进内页，如果是Google之类的遵守国际约定的蜘蛛的话，看到链接的nofollow，就不会继续抓取链接，这个链接也就不会给对方你的权重蛋糕了。

如果是Baidu之类的非主流机器人的话，虽然看到nofollow还是会继续勇往直前，但是这个已经是内页了，所以。权重输出也微乎..

光说不练不是好孩子，我们来看看如何实现吧，首先WordPress有个比较强大的功能，模板输出。

我的小站就是用模板输出实现的这个功能。在你的主题文件夹内，新建一个php文件，名字任意，合法就可以，然后复制下面的代码到你的文件内，稍加修改，把图片等修改掉即可。

如果你要实现自动跳转以及seo标题，描述，关键字，只需要在header内进行微量修改，即可。【突然发觉描述很累..】

我的跳转就加了一句，如果访问地址无效，或者提交地址为空，则6秒后返回主页。

或许有的人会问，为什么要有公钥私钥验证呢，是为了防止一些小人用你的网站发送带有问题的网站。

这个东西的主要应用在什么地方呢，
1.软件站和新闻站的引用地址,
2.个人blog提高自己的跳转安全[filter一下似乎不错]
3.一些小气的SEOer.[结合判断蜘蛛访问,应该能做的滴水不漏..]


欢迎大家留言讨论~


接下来，我把我的模板贴上来，大家稍加修改，就可以做出属于自己的redirect了。


```php

<?php
/*
Template Name: MyRedirect
*/
?>

<?php
//获取POST 提交信息
$url_target  = $_REQUEST["url"];
$url_referer = $_SERVER["HTTP_REFERER"];
$url_to      = $url_target;
//URL是否为空的判断
$url_empty = false;


if (empty($url_to)){$url_empty = true;}
else
{
//判断网站链接是否正确
	if (!preg_match("#^(http|news|https|ftp|ed2k|rtsp|mms)://#", $url_to)){$result = '网站地址错误';$url_to = '';}
		$diskey = array("\\",' ',"'",'"','*',',','<','>',"\r","\t","\n",'(',')','+',';');
		foreach($diskey as $value)
		{
			if (strpos($url_to,$value) !== false){$result = '网站地址错误';$url_to = '';}
		} 
}
//输出结果
function fir_echo_results($is_empty = false)
{
	$url_target  = $_REQUEST["url"];
	$url_referer = $_SERVER["HTTP_REFERER"];
	$url_to      = $url_target;
	//有个比较简单的加密验证,混合验证,使用公钥(网站地址)和私钥(下面设置的)
	//进行混合运算。我给的例子只是字串链接后MD5一下
	$url_fix     = 'fix';
	echo '<div id="redirect-info"><p>';

	if ($is_empty == true)
	{
		echo '链接地址为空,点此返回<a href="http://promiseforever.com">主页</a>。<br />';
	}
	else
	{
		echo '原始目标：'.$url_target.'<br />';
		echo '来源地址：'.$url_referer.'<br />';
		echo '目标地址：'.urlencode($url_to).'<br />';
		echo '网址指纹：'.md5($url_target).'<br />';
	
	//测试模式,提交debug,返回钥匙
	$url_debug  = $_REQUEST["debug"];
	if ('debug'==$url_debug){echo '安全判定：'.md5(($url_target.$url_fix)).'<br />';}

		//输出链接
		echo '点击这里<a target="_blank" rel="nofollow" href="'.$url_target.'" title="继续访问">继续访问</a><br />';
		echo '点击这里<a href="">返回来源</a><br />';
	}
	
	echo '</p></div>';
}

?>

<?php get_header(); ?>
	<div id="wrap">
	<!-- Main Content-->
		<img src="<?php bloginfo('stylesheet_directory');?>/images/content-top.gif" alt="content top" class="content-wrap" />
		<div id="content">
			<!-- Start Main Window -->
			<div id="main">
			<?php the_post(); ?>
				
				<div class="new_post entry clearfix">

					<h1 id="post-title"><a title="网站重定向" href="http://promiseforever.com/redirect"><?php the_title(); ?></a></h1>
						<div class="postcontent">

<div class="redirect-item-wrap">
<span class="redirect-item-top"></span>
<div class="redirect-item-inside">
<?
	$url_key  = $_REQUEST["key"];
	//如果要修改私钥，记得修改2处哦~
	$url_fix     = 'fix';
	$url_cer  = md5(($url_to.$url_fix));
	
	if ($url_empty == true)
	{
		$url_to ='';
		echo '<a href="'.$url_to.'" title="链接地址为空。">';
		echo '<img class="archive-item-cap" src="http://cache.promiseforever.com/images/pages/redirect/empty.gif" alt="链接地址为空"/></a>';
	}
	else
	{
		if ($url_key == $url_cer)
		{
			echo '<a href="'.$url_to.'" title="点击图片继续访问。">';
			echo '<img class="archive-item-cap" src="http://cache.promiseforever.com/images/pages/redirect/pass.gif"/></a>';
		}
		else
		{
			echo '<a href="#" title="该网址不被信任，继续访问？">';
			echo '<img class="archive-item-cap" src="http://cache.promiseforever.com/images/pages/redirect/tips.gif"/></a>';
		}
	}
?>
</div>
<span class="redirect-item-btm"></span>
</div>

<?php

	fir_echo_results($url_empty);

?>

							<div class="clear"></div>
						</div> <!-- end .post -->
				</div> 
			
			</div>
			<!-- End Main -->
	
	<?php get_sidebar(); ?>
	<?php get_footer(); ?>
```	


