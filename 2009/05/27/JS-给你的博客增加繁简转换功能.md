# [JS]给你的博客增加繁简转换功能

刚刚给博客添加了一个小功能，自动转换繁体和简体，有得必有失，这个功能是建立在一个18kbJS脚本上的，具体实现也很简单，原理为建立两个数组，然后遍历网页的字符内容进行替换。

<!-- more -->

是从这里扒来的[http://mrmo.cc/](http://mrmo.cc/)

[建议大家在扒取别人的东西的时候至少更人家一个链接... : )]

首先打开你的wordpress的header.php或其他包含导航菜单的文件[你也可以添加到其他的地方]

```php
<ul id="menus">
<!--<?php $FirSiteURL = get_settings('home') . '/';?>-->
	<li class="<?php echo($home_menu);>"><a class="home" title="网站首页" href="<?php echo $FirSiteURL; ?>">网站首页</a></li>
	省略一堆内容......
	<li><a id="translateLink" href="<?php echo $FirSiteURL; ?>">简体转换?</a></li>
</ul>
```

这样就添加了一个导航按钮。

接着打开你的end.php等包含的文件,在之前添加下面的代码,注意内容要根据自己的地址修改

```html
<script src="<?php bloginfo('template_url');" type="text/javascript"><!--mce:0--></script>
<script type="text/javascript"><!--mce:1--></script>
```

最后把这个JS文件下载下来就可以了。[下载方式，打开网页源代码进行查看，找到地址，接着复制到IE地址栏，回车]

