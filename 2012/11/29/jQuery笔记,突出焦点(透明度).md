# jQuery笔记,突出焦点(透明度)

高手莫入，浅显例子而已。最近在更换项目中的javascript库，觉得如果能把实践的过程记录下来，应该可以帮助到一些对javascript感兴趣的前端初学者。

[![jq-obj-opacity](https://attachment.soulteary.com/2012/11/29/jq-obj-opacity.png "jq-obj-opacity")](https://attachment.soulteary.com/2012/11/29/jq-obj-opacity.png) 

或许有的时候，你需要在一堆一样的图片或者图标中突显某一个元素。 

有一个不错的方案，就是将你要凸显的内容的透明度降低，而其他的元素的透明度调高。 

完整例子: http://thecdn.sinaapp.com/page/demo/jq-obj-opacity/

页面结构可以是下面的样子，对你想要突出显示的内容带上相同的class属性。

比如：obj-opacity

```html
<!doctype html>
<html>
<head>
	<meta charset="utf-8">
	<title>JQUERY OBJECT OPACITY</title>
	<link rel="stylesheet" href="extra/style.css">
	<script type="text/javascript" src="extra/jquery-1.8.3.min.js"></script>
	<script type="text/javascript" src="extra/jquery.object-opacity.js"></script>
</head>
<body>
	<div id="warp">
		<div id="logo" class="obj-opacity"></div>
		<ul id="img-list">
			<li class="obj-opacity"></li>
			<li class="obj-opacity"></li>
			<li class="obj-opacity"></li>
			<li class="obj-opacity"></li>
			<li class="obj-opacity"></li>
		</ul>
		<div id="single-pic">
			<img src="extra/single.gif" class="single">
			<p>这个小图需要点击来触发,原因看代码.</p>
		</div>
	</div>
</body>
</html>
```

接着把你的元素简单的展示组合一下

```css
*,html,body,div,ul,li{
	margin: 0;
	padding: 0;
}
html,body,div,ul,li{
	display: block;
	float: left;
}
body{
	background-color: #EEE;
}
div#warp{
	width: 260px;
	height: 180px;
	left: 50%;
	top: 50%;
	margin-left: -130px;
	margin-top: -90px;
	background-color: #DDD;
	position: absolute;
}
div#warp div#logo{
	background: url(logo.png) 0 0 no-repeat;
	width: 64px;
	height: 64px;
	margin: 30px;
	opacity:.9;
}
div#warp ul#img-list{
clear: left;
}
ul#img-list li.obj-opacity{
	background: url(small.png) 0 0 no-repeat;
	width: 32px;
	height: 32px;
	margin: 10px;
	cursor: pointer;
}
div#single-pic img.single{
position: absolute;
top: 20px;
left: 130px;
}
div#single-pic p{
position: absolute;
font: 9px/1.25 sans-serif;
color: #999;
display: block;
width: 90px;
left: 160px;
top: 12px;
}
/*初始化的透明度*/
.obj-opacity{
	opacity: .1;
}
```

接着就是jQuery插件的源代码: 因为有上一篇文章的铺垫，这篇就可以简单的注释了吧。

```js
/*    SOULTEARY.COM
	    _____ ____  __  ____  _______________    ______  __
	   / ___// __ \/ / / / / /_  __/ ____/   |  / __ \ \/ /
	   \__ \/ / / / / / / /   / / / __/ / /| | / /_/ /\  / 
	  ___/ / /_/ / /_/ / /___/ / / /___/ ___ |/ _, _/ / /  
	 /____/\____/\____/_____/_/ /_____/_/  |_/_/ |_| /_/   */

;(function($){
	$.fn.extend({
		"objOpacity": function(params){
			params = $.extend({
				warp:'#warp',			//查找元素的限定容器
				target:'.obj-opacity',	//目标元素具备的类名
				hover:'effect',			//具备焦点后添加类名
				event:'mouseover',		//触发焦点的事件
				bevent:'mouseout',		//失去焦点的事件[唯一时触发]
				focus:1,				//获得焦点后的透明度
				blur:.1,				//失去焦点后的透明度
				speed:600				//动画速度(单位ms)
			},params);
$(document).ready(function(){
	var target = $(params.target);
	var count = target.length;
	if (!count) {return;}
	var warp = $(params.warp);
	var o = 0;
	var tarClass = params.target.replace('.','');
	warp.bind(params.event, function(e){
		var cur = e.target;
		if (!$(cur).hasClass(tarClass)) {return;}
		target.removeClass(params.hover);
		$(cur).addClass(params.hover);
		for(var i=0;i<count;i++){
			if($(target[i]).hasClass(params.hover)){
				o =i;
				var index = target[o];
				if($(index).is(':animated')){
					$(index).stop().animate({'opacity':params.focus}, params.speed);
				}else{
					$(index).animate({'opacity':params.focus}, params.speed);
				}
				break;
			}
		}
		for(var i=0;i<count;i++){
			var index = target[i];
			if (i!==o){
				if($(index).is(':animated')){
					$(index).stop().animate({'opacity':params.blur}, params.speed);
				}else{
					$(index).animate({'opacity':params.blur}, params.speed);
				}
			}
		}
		if (count == 1) {
			//如果只是单一元素,添加失去焦点事件
			$(cur).bind(params.bevent,function(){
				if($(cur).is(':animated')){
					$(cur).stop().animate({'opacity':params.blur}, params.speed);
				}else{
					$(cur).animate({'opacity':params.blur}, params.speed);
				}
			});
		}
		e.stopPropagation();
	});
});
			return this;
		}
	});
}
)(jQuery,'SOULTEARY.COM');
```
		
最后的是使用方法

```js
//对于有一堆相同属性的元素
$('#warp').objOpacity({
	warp:'#warp',
	target:'.obj-opacity',
	hover:'effect',
	event:'mouseover',
	bevent:'mouseout',
	focus:1,
	blur:.1,
	speed:600
});
//对于只有一个属性的元素
$('#warp').objOpacity({
	warp:'#warp',
	target:'.single',
	hover:'effect',
	event:'click',
	bevent:'mouseout',
	focus:1,
	blur:.1,
	speed:600
});
```

