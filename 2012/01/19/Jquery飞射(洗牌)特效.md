# jQuery飞射(洗牌)特效

算下来,现在还差以前的登录页面没有独立做出来.所以手痒,打算做一下.

但是单调的登录感觉没什么意思,于是就开始写这个..

做了2个效果，一个是洗牌,一个是飞射出去再聚合一起。其实是一样的。下面贴下代码。

先贴基本的CSS和HTML结构.

```css
<style type="text/css">
#warp {
	width:auto;
	height:400px;
	background-color:#FCF;
	overflow:hidden;
	position:relative;
}
.slide {
	position:absolute;
	width:300px;
	left:50%;
	margin-left:-150px;
	height: 200px;
	top:50%;
	margin-top:-100px;
	overflow:hidden;
	line-height:200px;
	background-color: lightyellow;
	float:left;
}
#left {
	z-index:50;
	border: solid 5px red;
	height:240px;
}
#right {
	z-index:40;
	border: solid 5px green;
	height:180px;
}
.slide .inner {
	height: 100%;
	width: 100%
}
.slide #leftinner {
	line-height: normal;
	position: relative;
	text-align: center;
	top: 10%;
}
</style>
```

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>soulteary's jquery</title>
</head>
<body>
<button> play </button>
<br />
<div id="warp">
  <div id="left" class="slide">
    <div class="inner" id="leftinner">Left</div>
  </div>
  <div id="right" class="slide">
    <div class="inner" id="rightinner">Right</div>
  </div>
</div>
</body>
</html>
```

接着贴飞射的人代码,原理很简单,先做一个小变形,让人感觉box有变化的感觉。 然后设置speed快速的把box移动到屏幕外，接着再移动回来，并调整位置和z-index. 添加jquery库后,添加如下。

```js
$(document).ready(function(){
	function boxset($a,$b,$act){
		if ($act==1){
			$b.animate({width:"300px"},400);
			with($a){
				animate({height:"180px"},200);
 				fadeTo(100,0.4);
				animate({width:"250px"},100);
				css('z-index',30);
				animate({marginLeft:"2500px"});
				animate({marginLeft:"-240px"});
			}
			with($b){
				animate({marginLeft:"-2500px"});
				animate({height:"240px"},50);
 				fadeTo(100,1);
				css('z-index',60);
			}
		return -155;
		}else{
			$a.animate({width:"300px"},400);
			with($b){
				animate({height:"180px"},200);
 				fadeTo(100,0.4);
				animate({width:"250px"},100);
				css('z-index',40);
				animate({marginLeft:"-2500px"});
			}
			with($a){
				animate({marginLeft:"2500px"});
				animate({marginLeft:"-150px"});
				animate({height:"240px"},50);
 				fadeTo(100,1);
				css('z-index',50);
			}
		return 0;
		}
	}

	$("button,#left,#right").click(function(){
		var $divL = $("#left");
		var $divR = $("#right");
			$divR.animate({
				marginLeft:parseInt($divR.css('marginLeft'),0) >= -150 ? boxset($divL,$divR,1) : boxset($divL,$divR,2)
			});
	});


});
```

洗牌效果的话，就不需要移动到窗外了。

```js
$(document).ready(function(){
	function boxset($a,$b,$act){
		if ($act==1){
			$b.animate({width:"300px"},400);
			with($a){
				animate({height:"180px"},200);
 				fadeTo(100,0.4);
				animate({width:"250px"},100);
				css('z-index',30);
			}
			with($b){
				animate({height:"240px"},50);
 				fadeTo(100,1);
				css('z-index',60);
			}
		return -330;
		}else{
			$a.animate({width:"300px"},400);
			with($b){
				animate({height:"180px"},200);
 				fadeTo(100,0.4);
				animate({width:"250px"},100);
				css('z-index',40);

			}
			with($a){
				animate({height:"240px"},50);
 				fadeTo(100,1);
				css('z-index',50);
			}
		return 10;
		}
	}

	$("button,#left,#right").click(function(){
		var $divL = $("#left");
		var $divR = $("#right");
			$divR.animate({
				marginLeft:parseInt($divR.css('marginLeft'),0) >= -150 ? boxset($divL,$divR,1) : boxset($divL,$divR,2)
			});
	});


});
```

效果的话，可以看这里，只不过没做css样式，很丑陋...再考虑下再说吧。

