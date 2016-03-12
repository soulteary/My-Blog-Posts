# javascript简单操作元素

豆瓣上看到一个问题..http://www.douban.com/group/topic/29938097/ 感觉自己也做的不是很好,甚至可以说做的不好了吧... 动画效果，应该加个是否在播放的标志,否则会产生闪烁... 不过实在不想写了,过阵吧- -...(尝试写成优雅的apply,却发现各种出错...无奈只好经典里找了一个简单的动画实现- -!)

<!-- more -->

```html
<!doctype>
<html>
	<head>
		<title>boring</title>
		<style type="text/css">
			body{background-color: gray;color:#cd0;}
			/* 初始化属性为mouseout*/
			.init{
				width:40px;
				height:40px;
				border:2px solid #cf0;
				position:absolute;
				top:60px;
				left:60px;
			}

		</style>


	</head>
	<body>

		<div id="brick" class="init">fir.</div>

		<script type="text/javascript">
			(function(){
				//判断是否IE
					var isIE = navigator.userAgent.indexOf('MSIE') != -1 && !window.opera;
				//包装下getelementbyid
					function $(id){return document.getElementById(id);}
				//声明绑定事件函数
					function addEvent(elem, type, handler){
						if(isIE){
							elem.attachEvent('on' + type, (function(elm){return function() {handler.call(elm);}})(elem));
						}else{elem.addEventListener(type, handler, true);}
					}


				var objBrick = $("brick");
					changeBrickStyle(objBrick);

				//更换样式的方法
					function changeBrickStyle(objBrick){
						//定义mouserout等数值
						var objMouseOver = {
							width:80,
							height:80,
							top:40,
							left:40
						}

						var objMouseOut = {
							width:40,
							height:40,
							top:60,
							left:60
						}


						//给mouseover绑定样式
						addEvent(objBrick,"mouseout",function(obj){
							return function(){
								obj.style.color = "#cd0";
								obj.style.border= "1px solid #cd0";

								  
								//下面几个参数分别表示动画从第一帧开始，持续30帧，也就是1.2s，移动的距离为100px，开始的位置为0px
								//t为当前时间，c为总改变值，d为持续时间，b为初始值
								var t=0,c=20,d=3,b=0;   
								function easyAnim(){   
									obj.style.width =-t*c/d+objMouseOver.width+'px';
									obj.style.left  =t*c/d+objMouseOver.left+'px';
									obj.style.height=-t*c/d+objMouseOver.height+'px';
									obj.style.top   =t*c/d+objMouseOver.top+'px';
    								t++;
    								if(t<=d){setTimeout(easyAnim,40);}
								}easyAnim();

								//obj.style.width = objMouseOut.width  + "px";
								//obj.style.height= objMouseOut.height + "px";
								//obj.style.top	  = objMouseOut.top    + "px";
								//obj.style.left  = objMouseOut.left   + "px";
							};
						}(objBrick));
						
						//给mouseout绑定样式
						addEvent(objBrick,"mouseover",function(obj){
							return function(){
								obj.style.color = "#fff";
								obj.style.border= "2px solid #cf0";
								var t=0,c=20,d=3,b=0;   
								function easyAnim(){   
									obj.style.width =t*c/d+objMouseOut.width+'px';
									obj.style.left  =-t*c/d+objMouseOut.left+'px';
									obj.style.height=t*c/d+objMouseOut.height+'px';
									obj.style.top   =-t*c/d+objMouseOut.top+'px';
    								t++;
    								if(t<=d){setTimeout(easyAnim,40);}
								}easyAnim();
								//obj.style.width = objMouseOver.width  + "px";
								//obj.style.height= objMouseOver.height + "px";
								//obj.style.top	= objMouseOver.top    + "px";
								//obj.style.left	= objMouseOver.left   + "px";
							};
						}(objBrick));
					}
					
					
			})();
			</script>

	</body>
</html>
```

Run Code:

```
<!doctype>
<html>
	<head>
		<title>boring</title>
		<style type="text/css">
			body{background-color: gray;color:#cd0;}
			/* 初始化属性为mouseout*/
			.init{
				width:40px;
				height:40px;
				border:2px solid #cf0;
				position:absolute;
				top:60px;
				left:60px;
			}

		</style>


	</head>
	<body>

		<div id="brick" class="init">fir.</div>

		<script type="text/javascript">
			(function(){
				//判断是否IE
					var isIE = navigator.userAgent.indexOf('MSIE') != -1 && !window.opera;
				//包装下getelementbyid
					function $(id){return document.getElementById(id);}
				//声明绑定事件函数
					function addEvent(elem, type, handler){
						if(isIE){
							elem.attachEvent('on' + type, (function(elm){return function() {handler.call(elm);}})(elem));
						}else{elem.addEventListener(type, handler, true);}
					}


				var objBrick = $("brick");
					changeBrickStyle(objBrick);

				//更换样式的方法
					function changeBrickStyle(objBrick){
						//定义mouserout等数值
						var objMouseOver = {
							width:80,
							height:80,
							top:40,
							left:40
						}

						var objMouseOut = {
							width:40,
							height:40,
							top:60,
							left:60
						}


						//给mouseover绑定样式
						addEvent(objBrick,"mouseout",function(obj){
							return function(){
								obj.style.color = "#cd0";
								obj.style.border= "1px solid #cd0";

								  
								//下面几个参数分别表示动画从第一帧开始，持续30帧，也就是1.2s，移动的距离为100px，开始的位置为0px
								//t为当前时间，c为总改变值，d为持续时间，b为初始值
								var t=0,c=20,d=3,b=0;   
								function easyAnim(){   
									obj.style.width =-t*c/d+objMouseOver.width+'px';
									obj.style.left  =t*c/d+objMouseOver.left+'px';
									obj.style.height=-t*c/d+objMouseOver.height+'px';
									obj.style.top   =t*c/d+objMouseOver.top+'px';
    								t++;
    								if(t<=d){setTimeout(easyAnim,40);}
								}easyAnim();

								//obj.style.width = objMouseOut.width  + "px";
								//obj.style.height= objMouseOut.height + "px";
								//obj.style.top	  = objMouseOut.top    + "px";
								//obj.style.left  = objMouseOut.left   + "px";
							};
						}(objBrick));
						
						//给mouseout绑定样式
						addEvent(objBrick,"mouseover",function(obj){
							return function(){
								obj.style.color = "#fff";
								obj.style.border= "2px solid #cf0";
								var t=0,c=20,d=3,b=0;   
								function easyAnim(){   
									obj.style.width =t*c/d+objMouseOut.width+'px';
									obj.style.left  =-t*c/d+objMouseOut.left+'px';
									obj.style.height=t*c/d+objMouseOut.height+'px';
									obj.style.top   =-t*c/d+objMouseOut.top+'px';
    								t++;
    								if(t<=d){setTimeout(easyAnim,40);}
								}easyAnim();
								//obj.style.width = objMouseOver.width  + "px";
								//obj.style.height= objMouseOver.height + "px";
								//obj.style.top	= objMouseOver.top    + "px";
								//obj.style.left	= objMouseOver.left   + "px";
							};
						}(objBrick));
					}
					
					
			})();
			</script>

	</body>
</html>
```


