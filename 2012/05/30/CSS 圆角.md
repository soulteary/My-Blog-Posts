# CSS 圆角

CSS实现圆角其实蛮多方法的,CSS3的border-radius,webkit-border-radius或者-moz-border-radius专属浏览器属性. 图片圆角,再或者px错位进行圆角设计.以及HTML Canvas圆角. 看到百度的一个图片实现,觉得很不错,原因很简单,拼合图片sprite是圆，sprite定位是亮点。

<!-- more -->

```html
<!doctype>
<html>
	<head>
		<title>round</title>

		<style type="text/css">
			a:visited {
				color:purple;
				text-decoration:none;
			}
			a:link {
				color:#06C;
				text-decoration:none;
			}
			div {
				font-size:12px;
				line-height:18px;
				font-family:Arial;
			}
			.b2 {
				background:white;
				border:1px solid #CCC;
				margin-bottom:10px;
			}
			.rct .l,.rct .r {
				top:-1px;
			}
			.rct .l,.rcb .l {
				float:left;
				left:-1px;
			}
			.rct .r,.rcb .r {
				float:right;
				right:-1px;
			}
			.rct .l,.rct .r,.rcb .l,.rcb .r {
				position:relative;
				font-size:1px;
				background-image:url(http://img.baidu.com/hi/img/index/rc.gif);
				width:5px;
				height:5px;
				overflow:hidden;
			}
			.tl1 {
				background-position:top left;
			}
			.tr1 {
				background-position:top right;
			}
			.b2 .cnt {
				padding:10px;
			}
			.f14 {
				font-size:14px;
				line-height:24px;
			}
			.rcb {
				height:5px;
				clear:both;
			}
			.rcb .l,.rcb .r {
				top:1px;
			}
			.rct .l,.rcb .l {
				float:left;
				left:-1px;
			}
			.bl1 {
				background-position:left bottom;
			}
			.rcb .l,.rcb .r {
				top:1px;
			}
			.rct .r,.rcb .r {
				float:right;
				right:-1px;
			}

		</style>


	</head>
	<body>
		<div id="m_wap" class="b2">
			<div class="rct"><div class="l tl1"></div><div class="r tr1"></div></div>
			<div class="cnt f14">
				<div style="background: url(http://img.baidu.com/hi/img/index/wap_phone.jpg) no-repeat; height: 90px; margin-top: 5px;margin-left:10px">
					<div style="margin-left: 70px; font-size: 16px; font-family: '微软雅黑'; font-weight: bold; margin-bottom: 20px; color:#525252;">手机登录百度空间</div>
					<div style="background: url(http://img.baidu.com/hi/img/index/wap_button.png) no-repeat; margin-left: 70px; width: 139px; height: 33px; line-height: 33px; text-align: center;color:#0066cc;font-size:14px;text-decoration:under-line"><a id="wap_entrance" href="http://hi.baidu.com/hi/cms/article/help_phone/index.html" target="_blank">waphi.baidu.com</a></div>
				</div>
			</div>
			<div class="rcb"><div class="l bl1"></div><div class="r br1"></div></div>
		</div>
	</body>
</html>
```

Run Code:

```
<!doctype>
<html>
	<head>
		<title>radius</title>

		<style type="text/css">
			a:visited {
				color:purple;
				text-decoration:none;
			}
			a:link {
				color:#06C;
				text-decoration:none;
			}
			div {
				font-size:12px;
				line-height:18px;
				font-family:Arial;
			}
			.b2 {
				background:white;
				border:1px solid #CCC;
				margin-bottom:10px;
			}
			.rct .l,.rct .r {
				top:-1px;
			}
			.rct .l,.rcb .l {
				float:left;
				left:-1px;
			}
			.rct .r,.rcb .r {
				float:right;
				right:-1px;
			}
			.rct .l,.rct .r,.rcb .l,.rcb .r {
				position:relative;
				font-size:1px;
				background-image:url(http://promiseforever.com/extra-docs/radius/rc.gif);
				width:5px;
				height:5px;
				overflow:hidden;
			}
			.tl1 {
				background-position:top left;
			}
			.tr1 {
				background-position:top right;
			}
			.b2 .cnt {
				padding:10px;
			}
			.f14 {
				font-size:14px;
				line-height:24px;
			}
			.rcb {
				height:5px;
				clear:both;
			}
			.rcb .l,.rcb .r {
				top:1px;
			}
			.rct .l,.rcb .l {
				float:left;
				left:-1px;
			}
			.bl1 {
				background-position:left bottom;
			}
			.rcb .l,.rcb .r {
				top:1px;
			}
			.rct .r,.rcb .r {
				float:right;
				right:-1px;
			}

		</style>


	</head>
	<body>
		<div id="m_wap" class="b2">
			<div class="rct"><div class="l tl1"></div><div class="r tr1"></div></div>
			<div class="cnt f14">
				<div style="background: url(http://promiseforever.com/extra-docs/radius/wap_phone.jpg) no-repeat; height: 90px; margin-top: 5px;margin-left:10px">
					<div style="margin-left: 70px; font-size: 16px; font-family: '微软雅黑'; font-weight: bold; margin-bottom: 20px; color:#525252;">手机登录百度空间</div>
					<div style="background: url(http://promiseforever.com/extra-docs/radius/wap_button.png) no-repeat; margin-left: 70px; width: 139px; height: 33px; line-height: 33px; text-align: center;color:#0066cc;font-size:14px;text-decoration:under-line"><a id="wap_entrance" href="http://hi.baidu.com/hi/cms/article/help_phone/index.html" target="_blank">waphi.baidu.com</a></div>
				</div>
			</div>
			<div class="rcb"><div class="l bl1"></div><div class="r br1"></div></div>
		</div>
	</body>
</html>
```

