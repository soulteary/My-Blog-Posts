# jQuery笔记,自动完成

高手莫入，浅显例子而已。最近在更换项目中的javascript库，觉得如果能把实践的过程记录下来，应该可以帮助到一些对javascript感兴趣的前端初学者。

<!-- more -->

[![jquery-auto-complete](https://attachment.soulteary.com/2012/11/30/jquery-auto-complete.png "jquery-auto-complete")](https://attachment.soulteary.com/2012/11/30/jquery-auto-complete.png)

每天使用百度，google，有的时候，你的网站或许需要一个自动完成的功能。 你可以在这个例子中，搜索新浪，或者苏洋，因为是测试PHP，没有数据库请求操作，只有简单的两个词库，你懂的。

完整例子: http://thecdn.sinaapp.com/page/demo/jq-auto-complete/

首先还是页面结构

结构草图: http://thecdn.sinaapp.com/page/demo/jq-auto-complete/step1.html

```html
<!doctype html>
<html>
<head>
	<meta charset="utf-8">
	<title>JQUERY AUTO COMPLETE</title>
	<link rel="stylesheet" href="extra/style.css">
	<script type="text/javascript" src="extra/jquery-1.8.3.min.js"></script>
	<script type="text/javascript" src="extra/jquery.auto-complete.js"></script>
</head>
<body>
	<div id="warp">
		<div id="logo"></div>
		<input type="text" id="autocomplete">
		<input type="submit" id="search" value="搜索" class="s_btn">
		<div id="result">
			<table id="wordlst" cellspacing="0" cellpadding="2">
				<tbody>
					<tr><td><span>新浪</span><b>微博</b></td></tr>
					<tr><td><span>新浪</span><b>微博登陆</b></td></tr>
					<tr><td><span>新浪</span><b>邮箱</b></td></tr>
					<tr><td><span>新浪</span><b>nba</b></td></tr>
					<tr><td><span>新浪</span><b>爱问</b></td></tr>
					<tr><td><span>新浪</span><b>体育</b></td></tr>
					<tr><td><span>新浪</span><b>博客</b></td></tr>
					<tr><td><span>新浪</span><b>爱问共享资料</b></td></tr>
					<tr><td><span>新浪</span><b>网</b></td></tr>
					<tr><td><span>新浪</span><b>邮箱登陆</b></td></tr>
				</tbody>
			</table>
		</div>
	</div>
</body>
</html>
```

然后是样式 

样式: http://thecdn.sinaapp.com/page/demo/jq-auto-complete/step2.html

```css
*,html,body,div,ul,li,h3,p{
	margin: 0;
	padding: 0;
}
body, input{
	font-size: 12px;
	font-family: arial,\5b8b\4f53,sans-serif;
}
body{
	background-color:#F7F7F7;
}
div,ul,li,h3,p{
	float: left;
	display: block;
}

div#warp{
	width: 460px;
	height: 200px;
	top: 50%;
	left: 50%;
	position: absolute;
	margin-left: -200px;
	border: 1px solid #E0E0E0;
	margin-top: -200px;
	background-color: #FEFEFE;
}
div#warp div#logo{
	background: url(logo.png) 0 0 no-repeat;
	width: 220px;
	height: 64px;
	margin: 20px 0 0 130px;
	position: absolute;
}

div#warp input[type=text]{
	display: block;
	float: left;
}
div#warp input[type=text]#autocomplete{
	width: 200px;
	height: 22px;
	padding: 4px 7px;
	font: 16px arial;
	border: 1px solid #CDCDCD;
	border-color: #9A9A9A #CDCDCD #CDCDCD #9A9A9A;
	vertical-align: top;
	outline: none;
	margin: 100px 0 0 110px;
	background-color: white;
}

/*这里JS处理*/
div#warp div#result{
	border: 1px solid #817F82;
	position: relative;
	margin: 0 0 0 110px;
	text-align: left;
	-webkit-user-select: none;
	float: left;
	clear: left;
	min-width: 216px;
	display: none;/*列表首先弄掉，有数据再显示*/
}
div#result table{
	width: 100%;
	background: white;
	cursor: pointer;
	border-collapse: collapse;
	border-spacing: 0;
}
div#result table td {
	color: black;
	font: 14px arial;
	height: 25px;
	line-height: 25px;
	padding: 0 8px;
	text-align: left;
}
div#result table tr.hover {
	background-color: #e2eaff
}
div#result table td b {
	color: black;
}

div#warp input[type=submit]#search{
	display: block;
	float: left;
	margin: 90px 0 0 5px;
	height: 32px;
	cursor: pointer;
	background: url(btn.png) 0 0 no-repeat;
	border: 0;
}
div#warp input[type=submit]#search{
	display: block;
	float: left;
	margin: 100px 0 0 5px;
	height: 32px;
	cursor: pointer;
	background: url(btn.png) 0 0 no-repeat;
	border: 0;
	padding: 0 14px;
	font-size: 14px;
	width: 53px;
}
div#warp input[type=submit]#search.clicked{
	background-position: -55px 0;
}
```

接着一边想需求，一边实现吧。 

基本事件: http://thecdn.sinaapp.com/page/demo/jq-auto-complete/step3.html

```js
//首先是搜索按钮
//搜索框样式设置以及按钮事件
var searchBtn = $('#search');
//鼠标按下
searchBtn.bind('mousedown',function(){
	$(this).addClass('clicked');
	//开始搜索
	//搜索代码...
});

//鼠标移出
//可以体会一下，如果不绑定这个事件
//(只是在鼠标按下的时候去掉样式)
//在按钮按下鼠标，并移出按钮
searchBtn.bind('mouseout',function(){
	$(this).removeClass('clicked');
});


//结果输出
//缓存所有的表格tr元素
var target = $('table#wordlst').find('tr');
//元素数量
var count = target.length;

//if (!count) {return;}

//获取要事件委托的容器
var warp = $('table#wordlst').find('tbody');
//当前有焦点的元素项
var hover = 'hover';
//当前选择的元素的序号
//默认0是无,范围1~元素数量
var index = 0;
//初始化搜索关键词
var keyword = '';

//显示候选词列表
var showWordList = function(arrow){
	var warp = $('div#result');
	if (!arrow) {
		//重置焦点为0
		index = 0;
		//去掉所有的焦点hover
		for(var i=0;i<count;i++){
			if ($(target[i]).hasClass(hover)) {
				$(target[i]).removeClass(hover);
			}
		}
	}

	//根据关键词选择是否展示下拉框
	if(keyword == ""){
		warp.hide();
	}else{
		warp.show();
	}
}


//绑定table>tbody的鼠标移动事件
warp.bind('mouseover', function(e){
	var cur = e.target;
	//去掉所有其他元素的class中的hover
	target.removeClass(hover);
	//给每个元素的父级最上层元素加hover
	$(cur).parentsUntil(warp).addClass(hover);
	//通过循环获取当前有焦点的项目的序号
	for(var i=0;i<count;i++){
		if ($(target[i]).hasClass(hover)) {
			index = i;
		}
	}
	e.stopPropagation();
});

//绑定table>tbody的鼠标点击事件
target.bind('click',function(e){
	var cur = e.target;
	target.removeClass(hover);
	//将选择的元素的内容赋值给关键词
	keyword = $(cur).parentsUntil(warp)[0].innerText;

	//调用搜索
	//搜索代码...

	event.stopPropagation();
});

//绑定搜索框事件
//鼠标按下,方向键上，方向键下，ESC，回车
$('#autocomplete').bind('keydown', function(e){
	//根据index数值来进行候选下拉菜单的高亮
	var userSelect = function(){
		for(var i=0;i<count;i++){
			if ($(target[i]).hasClass(hover)) {
				$(target[i]).removeClass(hover);
			}
		}
		$(target[index-1]).addClass(hover);
		//设置输入框内容为选择中的内容
		$('#autocomplete')[0].value = $(target[index-1])[0].innerText;
		//把选择的内容赋值给关键词
		keyword = $('#autocomplete')[0].value;
	}

	//设置光标到字符串最后
	var setPointEnd = function(){ 
		var text = document.getElementById("autocomplete"); 
		var len =  text.value.length;

		if (text.setSelectionRange){
			setTimeout(function(){
			text.setSelectionRange(len, len);
			text.focus();
		}, 0);
		}else if (text.createTextRange){
			var textArea=document.getElementById("autocomplete");
			var tempText=textArea.createTextRange();
			tempText.moveStart("character",tempText.text.length);
			tempText.select();
		}
	}

	switch(e.keyCode){
		//方向键上
		case 38:
			//如果当前选择的内容比第一项小
			//那么设置焦点为最后一项
			//否则就数值减一
			if (index <= 1) {
				index = count;
			} else {
				index--;
			}
			//调用函数userSelect
			//设置选中项目高亮
			//以及对关键词赋值
			userSelect();
			//设置光标到最后
			setPointEnd();
			//显示候选词
			showWordList(true);
			break;
		case 40:
			//如果当前选择的内容比最后一项大
			//那么设置焦点为第一项
			//否则就数值加一
			if (index > count-1) {
				index = 1;
			} else {
				index++;
			}
			//调用函数userSelect
			//设置选中项目高亮
			//以及对关键词赋值
			userSelect();
			//显示候选词
			showWordList(true);
			break;
		case 13:
			//回车触发搜索
			if (index==0) {
				//没有展示下拉选择框
				keyword = $('#autocomplete')[0].value;
			}else{
				//选择备选
				keyword = $(target[index-1])[0].innerText;
			}
			//刷新备选列表
			//刷新列表代码...
			//调用搜索
			//搜索代码...

			break
		case 27:
			//触发ESC
			//关闭下拉框
			//并把index设置为0
			index=0;
	}

	e.stopPropagation();
});


if($.browser.msie){
	$('#autocomplete').on('propertychange', function(){
		//动态赋值,匹配计算
		keyword = $('#autocomplete')[0].value;
		showWordList();
	});
}else{
	$('#autocomplete').on('input', function(){
		//动态复制,匹配计算
		keyword = $('#autocomplete')[0].value;
		showWordList();
	});
}
```

之前一直是用写死的table数据，实际使用中，我们需要从服务器后端获取数据，所以呢。

接下来开始优化之前的代码和进行数据绑定。
要交互数据，首先要设计数据格式。

```json
{
	query:"新浪",
	result:["新浪微博",
			"新浪微博登陆",
			"新浪邮箱",
			"新浪nba",
			"新浪爱问",
			"新浪体育",
			"新浪博客",
			"新浪爱问共享资料",
			"新浪网",
			"新浪邮箱登陆"]
}
```

接下来用这份数据，生成我们需要的表格。
顺便把刚刚随便想到的草稿代码，整理一下

输出数据: http://thecdn.sinaapp.com/page/demo/jq-auto-complete/step4.html

```js
var searchBtn = $('#search');
searchBtn.bind('mousedown',function(){
	$(this).addClass('clicked');
	//开始搜索
	//搜索代码...
});
searchBtn.bind('mouseout',function(){
	$(this).removeClass('clicked');
});


window.$SAE = [];
//给一份默认的热词列表,优化体验
window.$SAE['QUERY_WORD'] = 
{
	query:"新浪",
	result:[["新浪微博!","http://weibo.com/"],
			"新浪微博登陆!",
			"新浪邮箱!",
			"新浪nba",
			"新浪爱问",
			"新浪体育",
			"新浪博客",
			"新浪爱问共享资料",
			"新浪网",
			"新浪邮箱登陆"]
};

//自动完成列表
var autoWordList = $('table#wordlst');
//输入框
var autoWordText = $('#autocomplete');
//为输入框光标缓存一份
var textCursor = document.getElementById('autocomplete'); 

var warp = autoWordList.find('tbody');
var warpBox = autoWordList.parent();

var target = autoWordList.find('tr');
var count = target.length;

//搜索设置
var searchHost = 'http://www.baidu.com/s?wd=';
//JSON数据接口
var dataHost = 'http://localhost/query.php?';
var keyWord = '';
var sarchPrefix = '+site:sae.sina.com.cn';
var newWindows = true;
//输入框限制搜索内容长度
var keyWordMaxLen = 100 -1;

//延时执行的句柄,用来取消不需要执行的内容
var handle =null;

var hover = 'hover';
var index = 0;


//显示下拉菜单
var doDropMenu = function(arrow){
	//如果被用户用ESC清空数据
	if (target==null) {return;}
	if (!arrow) {
		index = 0;
		for(var i=0;i<count;i++){
			if ($(target[i]).hasClass(hover)) {
				$(target[i]).removeClass(hover);
			}
		}
	}
	//如果获取候选词数量是0也不应该显示
	if(keyWord == ""|| count==0){
		warpBox.hide();
	}else{
		warpBox.show();
	}
}

//进行搜索
//添加第二个参数，为默认有转向地址的候选词提供服务
var doSearch = function(search, justRedirect){
	//如果有候选词，那么直接使用候选词地址
	if (justRedirect) {
		if(newWindows){
			window.open(justRedirect);
		}else{
			document.location.href = justRedirect;
		}
	//没有只好使用搜索的了
	}else{
		if(newWindows){
			window.open(searchHost+search+sarchPrefix);
		}else{
			document.location.href = searchHost+search+sarchPrefix;
		}		
	}
}

//清除之前的事件绑定以及内容
var doClean = function(){
	if(target !== null){
		//解除之前的数据绑定
		warp.unbind('mouseover');
		target.unbind('click');
		target = null;
	}
	//无论如何都应该清空和隐藏下拉的容器
	warp.empty();
	warpBox.hide();
}
//用JSON填充当前的数据
//因为需要回调，添加到JQUERY的全局变量中
$.extend({
	doJSON : function(resp){
		//首先扫地
		var sData = document.getElementById('syData');
		if(sData){sData.parentNode.removeChild(sData)}
	
		if (resp) {
			window.$SAE['QUERY_WORD'] = resp;
			return;
		}else{
			var sData = document.createElement('script');
			sData.id = 'syData';
			sData.type = 'text/javascript';
			sData.src = dataHost + 'keyword=' + autoWordText.val() +'&callback=jQuery.doJSON';
			document.getElementsByTagName('body')[0].appendChild(sData);
		}
	}
});


//获取关键词
var doQuery = function(){
	//之前的绑定单独抽象
	//因为要进行动态创建元素重新绑定
	var onLine = function(e){
		var curTR = $(e.target).parents('tr')
			target.removeClass(hover);
			curTR.addClass(hover);
		for(var i=0;i<count;i++){
			if ($(target[i]).hasClass(hover)) {
				index = i;
			}
		}
		e.stopPropagation();
	}

	var clickLine = function(e){
		var curTR = $(e.target).parents('tr');
			curTR.removeClass(hover);
		//写习惯DOM了- -!之前使用JQ=>DOM
		//然后操作,其实既然选择JQ
		//那么就通用操作都用JQ的封装方法吧
		keyWord = curTR.text();

		if (curTR.attr('data-url')) {
			//如果包含直接的定义跳转,就不去进行搜索
			//使用新的搜索函数进行搜索
			doSearch('',curTR.attr('data-url'));
		}else{
			//调用搜索
			doSearch(keyWord);
		}
		event.stopPropagation();
	}

	//高亮要搜索的内容
	//使用B标签是因为节约流量(@百度)
	var wordHighlight = function(word, keyARR){
		//然后把其他的字符都高亮
		for(var xx in keyARR){
			//排除被分割的字符是空
			if (keyARR[xx] !== '') {
				word = word.replace(keyARR[xx], '<b>'+keyARR[xx]+'</b>');
			}
		}
		return word;
	}

	//将原来的过程封装入函数
	//利于延时执行
	var coreQuery = function(){	
		//获取内容之前打扫卫生
		doClean();
		//因为是延时执行,所以用户碰巧在一瞬间把内容清空
		//延时的任务还是进行了，所以要判断内容是否为空
		if(autoWordText.val() == ''){return;}
		//重新获得并整理数据
		var data = $SAE.QUERY_WORD;
		var query = data.query;
		var list = data.result;
		if (list.length == 0) {return;}

		//最后要输出的HTML变量/
		//和处理过程中用的临时数组
		var tmpSTR = '';
		var tmpLine = '';
		var strHTML = [];
		for(var oo in list){
			//把内容分割为数组根据内容
			if(list[oo] instanceof Array){
				tmpSTR = list[oo][0];
			}else{
				tmpSTR = list[oo];
			}

			//用输入的内容分割字符串,排除要高亮的部分
			//先把内容替换为<tr><td>***关键词***</tr></td>

			//对内容进行高亮处理
			tmpLine = tmpSTR.replace(query, '<span>'+query+'</span>');
			tmpLine = wordHighlight(tmpLine, tmpSTR.split(query));
			//对内容进行链接处理
			if(list[oo] instanceof Array){
				tmpLine ='<tr data-url="'+list[oo][1]+'"><td>'+tmpLine+'</td></tr>';
			}else{
				tmpLine ='<tr><td>'+tmpLine+'</td></tr>';
			}

			//把整理好的内容添加到要输出的数组中
			strHTML.push(tmpLine);
		}
		//把整理好的数组合并字符串添加到文档
		autoWordList.html(strHTML.join(''));
		//展示自动完成的列表
		warpBox.show();
		//刷新数据
		//重新获取数据并绑定事件
		warp = autoWordList.find('tbody');
		target = autoWordList.find('tr');
		count = target.length;
		warp.bind('mouseover', onLine);
		target.bind('click',clickLine);
	}


	//添加延时执行,给用户一个选择的机会
	//如果用户在搜索新内容前移动光标则取消请求
	//而且可以降低服务器被请求数量
	//清除上一次延时任务
	clearTimeout(handle);
	//进入延时执行...
	//从服务器取数据
	$.doJSON();
	handle = setTimeout(coreQuery,300);
	//延时执行完毕...
}

//限制输入最大长度
var limitTextCount = function(){
	var safeSTR = autoWordText.val();
	//如果内容是空，不进行处理
	if (safeSTR=='') {return;}
	if(safeSTR.length>keyWordMaxLen){
		autoWordText.val(safeSTR.substring(0,keyWordMaxLen));
	}
}

autoWordText.bind('keydown', function(e){
	//如果按下ESC清空了缓存
	if(target == null){return;}

	var userSelect = function(){
		for(var i=0;i<count;i++){
			if ($(target[i]).hasClass(hover)){
				$(target[i]).removeClass(hover);
			}
		}
		$(target[index-1]).addClass(hover);
		autoWordText.val($(target[index-1]).text());
		keyWord = autoWordText.val();
	}

	//因为每次按下字符都调用这个函数
	//所以除了使用DOM外真的想不到什么更节约的方法
	//这里为了扩展性取元素未使用
	var setPointEnd = function(){
		if (textCursor.setSelectionRange){
			var len =  textCursor.value.length;
			setTimeout(function(){
			textCursor.setSelectionRange(len, len);
			textCursor.focus();
		}, 0);
		}else if (textCursor.createTextRange){
			var txtRange=textCursor.createTextRange();
			txtRange.moveStart("character",txtRange.text.length);
			txtRange.select();
		}
	}

	switch(e.keyCode){
		case 38:
			//如果用户使用上下方向键
			//说明在选择内容,那么取消要进行的刷新数据
			clearTimeout(handle);
			if (index <= 1) {
				index = count;
			} else {
				index--;
			}
			userSelect();
			setPointEnd();
			doDropMenu(true);
			break;
		case 40:
			//如果用户使用上下方向键
			//说明在选择内容,那么取消要进行的刷新数据
			clearTimeout(handle);
			if (index > count-1) {
				index = 1;
			} else {
				index++;
			}
			userSelect();
			doDropMenu(true);
			break;
		case 13:
			if (index==0) {
				keyWord = autoWordText.val();
			}else{
				keyWord = $(target[index-1]).text();
			}
			//内容不为空的时候
			if (keyWord !=''){
				//调用搜索
				doSearch(keyWord);
			}
			break
		case 27:
			index=0;
			//按下ESC后,应该清空上一次的搜索结果
			//而且既然ESC，那么就是重新输入需求
			//取消要进行的刷新数据
			clearTimeout(handle);
			//并且打扫卫生
			doClean();
			break;
		default:
		//限制输入的长度
			limitTextCount();
	}

	e.stopPropagation();
});

//内容改变不仅仅是按键输入，还有CTRL+V
autoWordText.bind('paste', function(){
	limitTextCount();
});


if($.browser.msie){
	autoWordText.on('propertychange', function(){
		//刷新备选列表
		doQuery();
		keyWord = autoWordText.val();
		//如果关键词为空,那么不展示下拉菜单
		if (keyWord!=='') {
			doDropMenu();
		}
	});
}else{
	autoWordText.on('input', function(){
		//刷新备选列表
		doQuery();
		keyWord = autoWordText.val();
		//如果关键词为空,那么不展示下拉菜单
		if (keyWord!=='') {
			doDropMenu();
		}
	});
}
```

感觉是不是距离成品越来越近了，那么把没有做的功能加上，关键是可以从数据源取数据。

优化和抽象: http://thecdn.sinaapp.com/page/demo/jq-auto-complete/step5.html

```js
var searchBtn = $('#search');
searchBtn.bind('mousedown',function(){
	$(this).addClass('clicked');
	//开始搜索
	//搜索代码...
});
searchBtn.bind('mouseout',function(){
	$(this).removeClass('clicked');
});


window.$SAE = [];
//给一份默认的热词列表,优化体验
window.$SAE['QUERY_WORD'] = 
{
	query:"新浪",
	result:[["新浪微博!","http://weibo.com/"],
			"新浪微博登陆!",
			"新浪邮箱!",
			"新浪nba",
			"新浪爱问",
			"新浪体育",
			"新浪博客",
			"新浪爱问共享资料",
			"新浪网",
			"新浪邮箱登陆"]
};

//自动完成列表
var autoWordList = $('table#wordlst');
//输入框
var autoWordText = $('#autocomplete');
//为输入框光标缓存一份
var textCursor = document.getElementById('autocomplete'); 

var warp = autoWordList.find('tbody');
var warpBox = autoWordList.parent();

var target = autoWordList.find('tr');
var count = target.length;

//搜索设置
var searchHost = 'http://www.baidu.com/s?wd=';
//JSON数据接口
var dataHost = 'http://localhost/query.php?';
var keyWord = '';
var sarchPrefix = '+site:sae.sina.com.cn';
var newWindows = true;
//输入框限制搜索内容长度
var keyWordMaxLen = 100 -1;

//延时执行的句柄,用来取消不需要执行的内容
var handle =null;

var hover = 'hover';
var index = 0;


//显示下拉菜单
var doDropMenu = function(arrow){
	//如果被用户用ESC清空数据
	if (target==null) {return;}
	if (!arrow) {
		index = 0;
		for(var i=0;i<count;i++){
			if ($(target[i]).hasClass(hover)) {
				$(target[i]).removeClass(hover);
			}
		}
	}
	//如果获取候选词数量是0也不应该显示
	if(keyWord == ""|| count==0){
		warpBox.hide();
	}else{
		warpBox.show();
	}
}

//进行搜索
//添加第二个参数，为默认有转向地址的候选词提供服务
var doSearch = function(search, justRedirect){
	//如果有候选词，那么直接使用候选词地址
	if (justRedirect) {
		if(newWindows){
			window.open(justRedirect);
		}else{
			document.location.href = justRedirect;
		}
	//没有只好使用搜索的了
	}else{
		if(newWindows){
			window.open(searchHost+search+sarchPrefix);
		}else{
			document.location.href = searchHost+search+sarchPrefix;
		}		
	}
}

//清除之前的事件绑定以及内容
var doClean = function(){
	if(target !== null){
		//解除之前的数据绑定
		warp.unbind('mouseover');
		target.unbind('click');
		target = null;
	}
	//无论如何都应该清空和隐藏下拉的容器
	warp.empty();
	warpBox.hide();
}
//用JSON填充当前的数据
//因为需要回调，添加到JQUERY的全局变量中
$.extend({
	doJSON : function(resp){
		//首先扫地
		var sData = document.getElementById('syData');
		if(sData){sData.parentNode.removeChild(sData)}
	
		if (resp) {
			window.$SAE['QUERY_WORD'] = resp;
			return;
		}else{
			var sData = document.createElement('script');
			sData.id = 'syData';
			sData.type = 'text/javascript';
			sData.src = dataHost + 'keyword=' + autoWordText.val() +'&callback=jQuery.doJSON';
			document.getElementsByTagName('body')[0].appendChild(sData);
		}
	}
});


//获取关键词
var doQuery = function(){
	//之前的绑定单独抽象
	//因为要进行动态创建元素重新绑定
	var onLine = function(e){
		var curTR = $(e.target).parents('tr')
			target.removeClass(hover);
			curTR.addClass(hover);
		for(var i=0;i<count;i++){
			if ($(target[i]).hasClass(hover)) {
				index = i;
			}
		}
		e.stopPropagation();
	}

	var clickLine = function(e){
		var curTR = $(e.target).parents('tr');
			curTR.removeClass(hover);
		//写习惯DOM了- -!之前使用JQ=>DOM
		//然后操作,其实既然选择JQ
		//那么就通用操作都用JQ的封装方法吧
		keyWord = curTR.text();

		if (curTR.attr('data-url')) {
			//如果包含直接的定义跳转,就不去进行搜索
			//使用新的搜索函数进行搜索
			doSearch('',curTR.attr('data-url'));
		}else{
			//调用搜索
			doSearch(keyWord);
		}
		event.stopPropagation();
	}

	//高亮要搜索的内容
	//使用B标签是因为节约流量(@百度)
	var wordHighlight = function(word, keyARR){
		//然后把其他的字符都高亮
		for(var xx in keyARR){
			//排除被分割的字符是空
			if (keyARR[xx] !== '') {
				word = word.replace(keyARR[xx], '<b>'+keyARR[xx]+'</b>');
			}
		}
		return word;
	}

	//将原来的过程封装入函数
	//利于延时执行
	var coreQuery = function(){	
		//获取内容之前打扫卫生
		doClean();
		//因为是延时执行,所以用户碰巧在一瞬间把内容清空
		//延时的任务还是进行了，所以要判断内容是否为空
		if(autoWordText.val() == ''){return;}
		//重新获得并整理数据
		var data = $SAE.QUERY_WORD;
		var query = data.query;
		var list = data.result;
		if (list.length == 0) {return;}

		//最后要输出的HTML变量/
		//和处理过程中用的临时数组
		var tmpSTR = '';
		var tmpLine = '';
		var strHTML = [];
		for(var oo in list){
			//把内容分割为数组根据内容
			if(list[oo] instanceof Array){
				tmpSTR = list[oo][0];
			}else{
				tmpSTR = list[oo];
			}

			//用输入的内容分割字符串,排除要高亮的部分
			//先把内容替换为<tr><td>***关键词***</tr></td>

			//对内容进行高亮处理
			tmpLine = tmpSTR.replace(query, '<span>'+query+'</span>');
			tmpLine = wordHighlight(tmpLine, tmpSTR.split(query));
			//对内容进行链接处理
			if(list[oo] instanceof Array){
				tmpLine ='<tr data-url="'+list[oo][1]+'"><td>'+tmpLine+'</td></tr>';
			}else{
				tmpLine ='<tr><td>'+tmpLine+'</td></tr>';
			}

			//把整理好的内容添加到要输出的数组中
			strHTML.push(tmpLine);
		}
		//把整理好的数组合并字符串添加到文档
		autoWordList.html(strHTML.join(''));
		//展示自动完成的列表
		warpBox.show();
		//刷新数据
		//重新获取数据并绑定事件
		warp = autoWordList.find('tbody');
		target = autoWordList.find('tr');
		count = target.length;
		warp.bind('mouseover', onLine);
		target.bind('click',clickLine);
	}


	//添加延时执行,给用户一个选择的机会
	//如果用户在搜索新内容前移动光标则取消请求
	//而且可以降低服务器被请求数量
	//清除上一次延时任务
	clearTimeout(handle);
	//进入延时执行...
	//从服务器取数据
	$.doJSON();
	handle = setTimeout(coreQuery,300);
	//延时执行完毕...
}

//限制输入最大长度
var limitTextCount = function(){
	var safeSTR = autoWordText.val();
	//如果内容是空，不进行处理
	if (safeSTR=='') {return;}
	if(safeSTR.length>keyWordMaxLen){
		autoWordText.val(safeSTR.substring(0,keyWordMaxLen));
	}
}

autoWordText.bind('keydown', function(e){
	//如果按下ESC清空了缓存
	if(target == null){return;}

	var userSelect = function(){
		for(var i=0;i<count;i++){
			if ($(target[i]).hasClass(hover)){
				$(target[i]).removeClass(hover);
			}
		}
		$(target[index-1]).addClass(hover);
		autoWordText.val($(target[index-1]).text());
		keyWord = autoWordText.val();
	}

	//因为每次按下字符都调用这个函数
	//所以除了使用DOM外真的想不到什么更节约的方法
	//这里为了扩展性取元素未使用
	var setPointEnd = function(){
		if (textCursor.setSelectionRange){
			var len =  textCursor.value.length;
			setTimeout(function(){
			textCursor.setSelectionRange(len, len);
			textCursor.focus();
		}, 0);
		}else if (textCursor.createTextRange){
			var txtRange=textCursor.createTextRange();
			txtRange.moveStart("character",txtRange.text.length);
			txtRange.select();
		}
	}

	switch(e.keyCode){
		case 38:
			//如果用户使用上下方向键
			//说明在选择内容,那么取消要进行的刷新数据
			clearTimeout(handle);
			if (index <= 1) {
				index = count;
			} else {
				index--;
			}
			userSelect();
			setPointEnd();
			doDropMenu(true);
			break;
		case 40:
			//如果用户使用上下方向键
			//说明在选择内容,那么取消要进行的刷新数据
			clearTimeout(handle);
			if (index > count-1) {
				index = 1;
			} else {
				index++;
			}
			userSelect();
			doDropMenu(true);
			break;
		case 13:
			if (index==0) {
				keyWord = autoWordText.val();
			}else{
				keyWord = $(target[index-1]).text();
			}
			//内容不为空的时候
			if (keyWord !=''){
				//调用搜索
				doSearch(keyWord);
			}
			break
		case 27:
			index=0;
			//按下ESC后,应该清空上一次的搜索结果
			//而且既然ESC，那么就是重新输入需求
			//取消要进行的刷新数据
			clearTimeout(handle);
			//并且打扫卫生
			doClean();
			break;
		default:
		//限制输入的长度
			limitTextCount();
	}

	e.stopPropagation();
});

//内容改变不仅仅是按键输入，还有CTRL+V
autoWordText.bind('paste', function(){
	limitTextCount();
});


if($.browser.msie){
	autoWordText.on('propertychange', function(){
		//刷新备选列表
		doQuery();
		keyWord = autoWordText.val();
		//如果关键词为空,那么不展示下拉菜单
		if (keyWord!=='') {
			doDropMenu();
		}
	});
}else{
	autoWordText.on('input', function(){
		//刷新备选列表
		doQuery();
		keyWord = autoWordText.val();
		//如果关键词为空,那么不展示下拉菜单
		if (keyWord!=='') {
			doDropMenu();
		}
	});
}
```

这里随便写了一个PHP，模拟输出搜索内容的返回数据

```php
<?php
/* SOULTEARY.COM
   _____ ____  __  ____  _______________    ______  __
  / ___// __ \/ / / / / /_  __/ ____/   |  / __ \ \/ /
  \__ \/ / / / / / / /   / / / __/ / /| | / /_/ /\  / 
 ___/ / /_/ / /_/ / /___/ / / /___/ ___ |/ _, _/ / /  
/____/\____/\____/_____/_/ /_____/_/  |_/_/ |_| /_/   
*/

/***
*	简单的搜索示例文件
					***/
//首先你要设置HEADER,如果你不想浏览器提示异常的话
header('Content-Type:application/javascript');

//初始化关键词
$KeyWord = '';
if (isset($_REQUEST['keyword']) && !empty($_REQUEST['keyword'])) {
	//这里需要加入过滤判断
	$KeyWord = $_REQUEST['keyword'];
}else{
	//空的话,不继续执行
	die();
}

//初始化回调函数
$CallBack = '';
if (isset($_REQUEST['callback']) && !empty($_REQUEST['callback'])) {
	$CallBack = $_REQUEST['callback'];
}

//这里是例子,
//真实环境是从数据库或者MC缓存中取
//下面就简单的用测试关键字来搞吧
switch ($KeyWord) {
	case '新':
			$Data = '{
				query:"'.$KeyWord.'",
				result:[["新浪微博!","http://weibo.com/"],
						"新浪微博登陆!",
						"新浪邮箱!",
						"新浪nba",
						"新浪爱问",
						"新浪体育",
						"新浪博客",
						"新浪爱问共享资料",
						"新浪网",
						"新浪邮箱登陆"]
			}';
		break;
	case '新浪':
			$Data = '{
				query:"'.$KeyWord.'",
				result:[["新浪微博!","http://weibo.com/"],
						"新浪微博登陆!",
						"新浪邮箱!",
						"新浪nba",
						"新浪爱问",
						"新浪体育",
						"新浪博客",
						"新浪爱问共享资料",
						"新浪网",
						"新浪邮箱登陆"]
			}';
		break;
	case '苏':
			$Data = '{
				query:"'.$KeyWord.'",
				result:[["苏洋博客","http://soulteary.com/"],
						"苏洋微博",
						"苏洋邮箱",
						"苏洋nba",
						"苏洋爱问",
						"苏洋体育",
						"苏洋博客",
						"苏洋爱问共享资料",
						"苏洋网",
						"苏洋邮箱登陆"]
			}';
		break;
	case '苏洋':
			$Data = '{
				query:"'.$KeyWord.'",
				result:[["苏洋博客","http://soulteary.com/"],
						"苏洋微博",
						"苏洋邮箱",
						"苏洋nba",
						"苏洋爱问",
						"苏洋体育",
						"苏洋博客",
						"苏洋爱问共享资料",
						"苏洋网",
						"苏洋邮箱登陆"]
			}';
		break;
	default:
			$Data = '{
				query:"'.$KeyWord.'",
				result:[]
			}';
		break;
}


//最后就是输出
echo $CallBack.'('.$Data.')';
?>
```

最后就是为了以后的复用，插件化

插件化: http://thecdn.sinaapp.com/page/demo/jq-auto-complete/index.html

调用方法还是简单的

```js

	$('#warp').autoComplete({
		wdList:'table#wordlst',					//下拉列表
		wdText:'#autocomplete',					//输入框
		engine:'http://www.baidu.com/s?wd=',	//搜索引擎地址
		kwFix:'+site:sae.sina.com.cn',			//搜索引擎参数
		djson:'http://localhost/query.php?',	//数据源地址
		wdMax:100,								//最多输入字符
		newWin:false,							//在新窗口打开
		wait:300,								//延时操作(毫秒)
		gData:$SAE['QUERY_WORD'],				//全局变量
		hover:'hover'							//鼠标和方向键给于高亮的类名
```


写在最后，又写完了一篇。

因为很多内容在之前的两篇，还有之前的注释中提到过，所以不会再次赘述，如果有疑问，不妨先看看之前的内容。

实在无解，可以留言提问，一起学习，一起成长。我觉得内容还是很简单的，尤其是分解动作之后。

