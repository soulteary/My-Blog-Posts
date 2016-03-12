# jQuery笔记,选项卡

高手莫入，浅显例子而已。最近在更换项目中的javascript库，觉得如果能把实践的过程记录下来，应该可以帮助到一些对javascript感兴趣的前端初学者，@胖子哥，是吧。

[![tab-switch](https://attachment.soulteary.com/2012/11/29/tab-switch.png "tab-switch")](https://attachment.soulteary.com/2012/11/29/tab-switch.png) 我们常常看到网站上有一些可以切换的选项卡，那么这些选项卡是怎么制作的呢。

首先看一个简单的栗子-v-

http://thecdn.sinaapp.com/page/demo/jq-tab-switch/

那么这样的效果是怎么实现的呢。

我们来一步一步分解任务，首先在纸上把你的需求草图画一下。我鼠绘比较丑，PS随便拖几个色块，如图所示。

 [![tab-struct](https://attachment.soulteary.com/2012/11/29/tab-struct.png "tab-struct")](https://attachment.soulteary.com/2012/11/29/tab-struct.png) 
 
 首先这个选项卡是分两个部分的，上面一条比较窄的是导航按钮，下面比较大面积的是展示内容。导航按钮中又包含了一些深红色的按钮。
 
  接着想一下用户和这个选项卡的交互，比如鼠标划过会切换内容，或者点击会切换内容。再往深一点想，内容是怎么切换。 
  
  比如你要使用Ajax的XHR来填充内容，还是使用已经存在的数据，比如CALLBACK回来的JSON来动态填充，还是简单的切换隐藏页面内已经存在的内容。切换内容你又要使用神马形式的特效，是淡入淡出，还是左飞出，还是对角线移动... 
  
  好了，把心思收回来，先从简单的开始。先用HTML把刚刚的想法的草图实现一下。 
  
  关于结构，大家仁者见仁，我觉得下面的结构用来展示还是比较不错的。

```html
<!doctype html>
<html>
<head>
	<meta charset="utf-8">
	<title>JQUERY TAB SWITCH</title>
</head>
<body>
	<div id="tab-warp">
		<ul class="tab-label">
			<li class="tab-label-item hover">标签A</li>
			<li class="tab-label-item">标签B</li>
			<li class="tab-label-item">标签C</li>
			<li class="tab-label-item">标签D</li>
		</ul>
		<ul class="tab-panel">
			<li class="tab-panel-item hover">
				<p>我欲与君相知，长命无绝衰。 山无陵，江水为竭，冬雷震震，夏雨雪， 天地合，乃敢与君绝!...</p>
			</li>
			<li class="tab-panel-item">
				<p>愿得一心人,白头不相离。</p>
			</li>
			<li class="tab-panel-item">
				<p>得成比目何辞死愿作鸳鸯不羡仙。</p>
			</li>
			<li class="tab-panel-item">
				<p>身无彩凤双飞翼心有灵犀一点通。</p>
			</li>
		</ul>
	</div>
</body>
</html>
```

结构展示:

http://thecdn.sinaapp.com/page/demo/jq-tab-switch/step-1.html

现在貌似很丑陋，没关系，你可以用CSS来美化一下它。当然，作为例子，我就不细致的去搞它了。

使用下面的CSS就能看出一个大概的样子，足够下一步写JS使用的样式。

```css
*,html,body,div,ul,li,h3,p{
	margin: 0;
	padding: 0;
}
body{
	background-color:#caefff;
}
div,ul,li,h3,p{
	float: left;
	display: block;
}

div#tab-warp{
	width: 400px;
	height: 200px;
	top: 50%;
	left: 50%;
	position: absolute;
	margin-left: -200px;
	margin-top: -200px;
}
div#tab-warp ul.tab-label, div#tab-warp ul.tab-panel{
	width:100%;
}
div#tab-warp ul.tab-label{
	height: 30px;
	border-bottom: none;
}
div#tab-warp ul.tab-panel{
	background-color: #a3cadc;
	height: 150px;
	border: 2px solid #fff;
}
ul.tab-label li.tab-label-item{
	background-color: #555;
	width: 50px;
	height: 24px;
	line-height: 24px;
	font-weight: normal;
	margin-top: 5px;
	border: 1px solid #EEE;
	color: white;
	text-align: center;
	cursor: pointer;
}
ul.tab-label li.tab-label-item.hover{
	background-color: #64AEFF;
	width: 60px;
	height: 29px;
	line-height: 29px;
	font-weight: bold;
	margin-top: 0;
}
ul.tab-panel li.tab-panel-item{
	width: 100%;
	height: 100%;
	color: #434343;
	display: none;
}
ul.tab-panel li.tab-panel-item p{
	margin:10px;
}
ul.tab-panel li.tab-panel-item.hover{
	display: block;
}
```

> 样式展示:
> http://thecdn.sinaapp.com/page/demo/jq-tab-switch/step-2.html

如果上一步添加样式，让你觉得这个选项卡有点丑，那么你可以继续在上一步精雕细琢，完成你要的选项卡样式。 如果你完成了，或者你想看如何把交互内容添加到选项卡上，那么我们继续。 

在页面中引入jQuery库后我们开始写可以让选项卡活起来的代码，接下来的说明都会写到注释中。

```js
$(document).ready(
     //如果网络速度不佳，页面结构没有下载完毕
     //这时脚本先于文档结构解析完毕的时候执行或许会出错
     //所以我们使用这个方法把主要的逻辑代码包裹起来
 
     //对应HTML文档中的选项卡的上部的导航标签
     //获取这货的所有子元素，然后为他们绑定一个事件:鼠标点击
     $('div#tab-warp>ul.tab-label').children().bind("click",
         function(e) {
             //当鼠标点击后
             //先分别获取选项卡两个部分的子元素
             var nav = $('div#tab-warp>ul.tab-label').children();
             var pal = $('div#tab-warp>ul.tab-panel').children();
             //判断他们的子元素数量是否一致
             var navc = nav.length;
             var palc = nav.length;
             //如果不一致，那么没有办法正常切换，所以不继续执行
             if (navc != palc) {
                 return;
             }
 
             //去掉所有的按钮的class中的'hover'
             for (var i = 0; i < navc; i++) {
                 $(nav[i]).removeClass('hover');
             }
             //给当前点击的元素的class中添加'hover'
             $(this).addClass('hover');
 
             var o = 0;
             //定义一个变量获取当前具有焦点的元素的序号(index)
             //然后获取当前的具有'hover'的元素的序号
             for (var i = 0; i < navc; i++) {
                 //如果某个元素被选中，那么这个元素的class包含'hover'
                 if ($(nav[i]).hasClass('hover')) {
                     //注意这里因为一一对应,下面注释有说
                     o = i;
                 }
             }
             //因为上边的选项卡按钮和下面的内容是一一对应的。
             //所以内容和选项卡进行同样的操作
             //去掉所有内容class中的'hover'
             pal.removeClass('hover');
             //接着给和按钮序号一样的家伙添加一个'hover'属性
             $(pal[o]).addClass('hover');
             //最后终止事件冒泡
             e.stopPropagation();
         }
     )
 );
```

> 交互展示:
> http://thecdn.sinaapp.com/page/demo/jq-tab-switch/step-3.html

在这一步，你可以给你的选项卡切换添加不同的特效或者根据不同的事件触发，比如鼠标移动到元素上，或者按下了某个键盘按键等。

那么这样写法固定的脚本意义大嘛，个人感觉不大，既然这个东西有共性，那么我们应该把这个家伙抽象出来，变成一个公共的东西。

jQuery提供一种叫做插件的形式来组织这样可以公共使用的内容。

在此之前，我们需要把插件中效率不是太高的方法，重写一下。

```js
$(document).ready(

    //首先是绑定元素替换为绑定元素的父级，统称事件代理，
    //这个事件中绑定导航按钮的父元素可以有效缩减3次绑定
    $('div#tab-warp>ul.tab-label').parent().bind('click',
        //把点击事件传到e这个参数中
        //让函数中可以引用
        function(e) {
            //对e.target元素进行处理
            //触发的元素得到了鼠标的眷顾哦
            //先来判断下是不是我们要的按钮元素
            //我们要的按钮元素的class具备tab-label-item
            //所以没有这个class属性的元素，就不和他玩了
            if (!$(e.target).hasClass('tab-label-item')) {
                return;
            }

            var nav = $('div#tab-warp>ul.tab-label').children();
            var pal = $('div#tab-warp>ul.tab-panel').children();
            var navc = nav.length;
            var palc = nav.length;
            if (navc != palc) {
                return;
            }

            //把之前的循环也用已经缓存了的对象来操作吧
            nav.removeClass('hover');
            //之前的this到这里要修改为假定的目标元素
            $(e.target).addClass('hover');

            var o = 0;
            for (var i = 0; i < navc; i++) {

                if ($(nav[i]).hasClass('hover')) {
                    o = i;
                }
            }
            pal.removeClass('hover');
            $(pal[o]).addClass('hover');
            e.stopPropagation();
        }
    )
);
```

> 简单优化:
> http://thecdn.sinaapp.com/page/demo/jq-tab-switch/step-4.html

假定上面的javascript优化的差不多了，我们就可以制作插件了。

```js
//为了保护世界的和平
//为了防止世界被破坏
//先把内容扔匿名函数中
;
(function($) {
    //用覆盖的方式写插件
    $.fn.extend({
        //插件就叫switchTab吧
        //传入一个对象类型的参数叫做params
        //之前的固定的字符串修改为参数
        // 'div#tab-warp>ul.tab-label' ==> params.tabNav
        // 'div#tab-warp>ul.tab-panel' ==> params.tabPan
        // 'tab-label-item'			==> params.tabNavBtn
        // 'hover'					==> params.hover
        // 'click'					==> params.event
        //接着我们调用的时候传入的参数大概就是这样
        //params ==> {
        //	tabNav:'div#tab-warp>ul.tab-label',
        //	tabPan:'div#tab-warp>ul.tab-panel',
        //	tabNavBtn:'tab-label-item',
        //	hover:'hover',
        //	event:'click'
        //	}
        "switchTab": function(params) {
            //设置参数默认值
            params = $.extend({
                tabNav: 'div#tab-warp>ul.tab-label', //导航按钮的容器
                tabPan: 'div#tab-warp>ul.tab-panel', //对应的内容的容器
                tabNavBtn: 'tab-label-item', //导航按钮的元素
                hover: 'hover', //获得焦点后要添加的css类名
                event: 'click' //要绑定的事件类型
            }, params);
            //这里开始之前的函数内容
            $(document).ready(
                $(params.tabNav).parent().bind(params.event,
                    function(e) {
                        if (!$(e.target).hasClass(params.tabNavBtn)) {
                            return;
                        }
                        var nav = $(params.tabNav).children();
                        var pal = $(params.tabPan).children();
                        var navc = nav.length;
                        var palc = nav.length;
                        if (navc != palc) {
                            return;
                        }
                        nav.removeClass(params.hover);
                        $(e.target).addClass(params.hover);
                        var o = 0;
                        for (var i = 0; i < navc; i++) {
                            if ($(nav[i]).hasClass(params.hover)) {
                                o = i;
                            }
                        }
                        pal.removeClass(params.hover);
                        $(pal[o]).addClass(params.hover);
                        e.stopPropagation();
                    }
                )
            );
            //这里结束之前函数内容

            return this; //返回插件本身
            //让插件可以链式操作
        }
    });
})(jQuery, 'SOULTEARY.COM');
//第一个参数是传入我们要选项卡化的元素


//最后就是插件化的意义
//简单的传入参数实现调用
$('#tab-warp').switchTab({
    tabNav: 'div#tab-warp>ul.tab-label',
    tabPan: 'div#tab-warp>ul.tab-panel',
    tabNavBtn: 'tab-label-item',
    hover: 'hover',
    event: 'click'
});
```

> 插件化:
> http://thecdn.sinaapp.com/page/demo/jq-tab-switch/step-5.html

最后抽离所有的内联元素，让页面略微规范。

> 最后结果:
> http://thecdn.sinaapp.com/page/demo/jq-tab-switch/index.html

至此这篇肤浅的文章就结束了，希望路过的前端爱好者可以从这里获取一点点的知识。

如果有任何的疑问，或者勘误，欢迎留言斧正。



