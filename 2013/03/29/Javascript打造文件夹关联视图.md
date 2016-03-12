# Javascript打造文件夹关联视图

写在前面,晓珊姐,我不是故意拖稿的!TAT...

菜鸟练笔，欢迎斧正，高手勿喷。

某些时候,我们需要在网页中实现TreeView和GridView两种视图,用网盘的界面举例吧。

实际需求包含并不局限于：文件管理，项目管理，点餐神马的...

![Image](https://attachment.soulteary.com/2013/03/29/view.png "Image")

乍看毫无难度的东西其实还是有很多值得商酌的地方的。
比如：

- 是使用一套结构，一套数据不同样式来呈现结构，还是使用多套结构，一套公用数据来进行实现效果。
数据存放在什么地方？
- 是HTML标签内，还是添加元素的class状态名，还是使用javascript全局/局部变量?

那么我们先来看看这两种模式的好处坏处吧：(点击切换比较结果)

一套结构

```html
<!doctype html>
<html>
	<head>
		<meta charset="utf-8">
		<title>demo</title>
	</head>
	<body>
		<ul id="view">
			<li class="item"></li>
			<li class="item"></li>
			<li class="item"></li>
			<li class="item"></li>
			<li class="item"></li>
		</ul>
	</body>
</html>
```

两套结构

```html
<!doctype html>
<html>
	<head>
		<meta charset="utf-8">
		<title>demo</title>
	</head>
	<body>
		<ul id="grid-view">
			<li class="item"></li>
			<li class="item"></li>
			<li class="item"></li>
			<li class="item"></li>
			<li class="item"></li>
		</ul>
		<table id="list-view">
			<thead>
				<th></th>
				<th></th>
				<th></th>
			</thead>
			<tbody>
				<tr>
					<td></td>
				</tr>
				<tr>
					<td></td>
				</tr>
				<tr>
					<td></td>
				</tr>
				<tr>
					<td></td>
				</tr>
				<tr>
					<td></td>
				</tr>
			</tbody>
		</table>
	</body>
</html>
```


如上面的代码示例，

**共用结构**

显而易见的优点：

1. HTML的大致结构比较简单。
2. 事件绑定的数量也比较集中，比如都绑定在`ul#view`上，javascript业务书写也集中在一起，方便简单的增加修改。
3. 数据是简单的一套数据，方便维护。

吹毛求疵的缺点：

1. 因为使用javascript+css来控制视图切换，css会产生很多不必要存在的写法，比如重复定义line-height,width等,或者说,会产生class类名的大量使用,同时因为事件绑定比较集中，js中也会掺杂大量的状态判断，比如现在控件处于什么状态，控件内部的元素又是什么状态等，每种状态都要添加class或者标志进行判断，js代码看起来也会比较复杂，虽然其实还是很简单的，不利于后期他人接手和维护，看起来和传说中的意大利面一样，一团。
2. 使用DIV或者其他元素配合CSS实现类似TABLE对于大量数据的展示，或者精确计算每一个元素的offset再进行展示比较麻烦。

** 单独结构，共用数据(或以某视图数据为标准) **

显而易见的优点：

1. HTML的细节结构比较简单（因为把两种状态分别展示，所以每一种状态的结构就向上了一层）。
2. 事件绑定数量也可以和之前一样，只绑定一个元素（外层再包裹一层容器，比如div#main什么的）。
3. CSS样式互相独立，维护会比较简单，JS两种状态的业务也互相独立，维护和开发会比较简单。
4. 分别使用两种结构来展示元素的话，语义结构也会比较清晰，而且可以轻松利用table对大量数据展示的优势。

缺点(或许吹毛求疵)：

1. 首先是要公用数据，那么不论是使用HTML DATA标签来存放数据，或者元素添加class类名来保存状态，还是js使用全局变量，都将存在一定的风险，如果那里书写不严谨，就会发生两边视图不一致的问题。
2. 代码量比之前大，看起来的复杂度比较高。


结合上面的简单比较，以及之前在帮同事写码的感觉，我偏向第二种方式。
另外一说，国内人气很旺的：微盘(vdisk.weibo.com)和百度网盘(pan.baidu.com)都采取了第二种方式。

随便填充了一些模拟的假数据，写了一个结构如下：

`http://thecdn.sinaapp.com/page/demo/flie-view/index-structure.html`

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>Local Test Page</title>
</head>
<body>
<ul id="grid-view">
    <li class="item flie-type-doc">
        <div class="checkbox">
            <input type="checkbox">
        </div>
        <div class="flie-meta">
            <span class="flie-name">考试答案.doc</span><span class="flie-timestamp">2013-3-28 15：02</span>
        </div>
    </li>
    <li class="item flie-type-doc">
        <div class="checkbox">
            <input type="checkbox">
        </div>
        <div class="flie-meta">
            <span class="flie-name">考试答案.doc</span><span class="flie-timestamp">2013-3-28 15：02</span>
        </div>
    </li>
    <li class="item flie-type-doc">
        <div class="checkbox">
            <input type="checkbox">
        </div>
        <div class="flie-meta">
            <span class="flie-name">考试答案.doc</span><span class="flie-timestamp">2013-3-28 15：02</span>
        </div>
    </li>
    <li class="item flie-type-doc">
        <div class="checkbox">
            <input type="checkbox">
        </div>
        <div class="flie-meta">
            <span class="flie-name">考试答案.doc</span><span class="flie-timestamp">2013-3-28 15：02</span>
        </div>
    </li>
    <li class="item flie-type-doc">
        <div class="checkbox">
            <input type="checkbox">
        </div>
        <div class="flie-meta">
            <span class="flie-name">考试答案.doc</span><span class="flie-timestamp">2013-3-28 15：02</span>
        </div>
    </li>
</ul>

<table id="list-view">
    <thead>
    <th>文件名称</th>
    <th>文件大小</th>
    <th>上传时间</th>
    </thead>
    <tbody>
    <tr>
        <td><span class="flie-type-doc">考试答案.doc</span></td>
        <td><span class="flie-size">100KB</span></td>
        <td><span class="flie-timestamp">2013-3-28 15：02</span></td>
    </tr>
    <tr>
        <td><span class="flie-type-doc">考试答案.doc</span></td>
        <td><span class="flie-size">100KB</span></td>
        <td><span class="flie-timestamp">2013-3-28 15：02</span></td>
    </tr>
    <tr>
        <td><span class="flie-type-doc">考试答案.doc</span></td>
        <td><span class="flie-size">100KB</span></td>
        <td><span class="flie-timestamp">2013-3-28 15：02</span></td>
    </tr>
    <tr>
        <td><span class="flie-type-doc">考试答案.doc</span></td>
        <td><span class="flie-size">100KB</span></td>
        <td><span class="flie-timestamp">2013-3-28 15：02</span></td>
    </tr>
    <tr>
        <td><span class="flie-type-doc">考试答案.doc</span></td>
        <td><span class="flie-size">100KB</span></td>
        <td><span class="flie-timestamp">2013-3-28 15：02</span></td>
    </tr>

    </tbody>
</table>
</body>
</html>
```

然后再进一步修改，增加样式后，两个粗糙的界面如下：

`http://thecdn.sinaapp.com/page/demo/flie-view/index-layout.html`

结构如下:

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>Local Test Page</title>
    <link rel="stylesheet/less" type="text/css" href="extra/style/main.less">
    <script type="text/javascript" src="extra/js/less-1.3.3.min.js"></script>
</head>
<body>
<div class="warp">
    <ul id="grid-view">
        <li class="item flie-type-doc">
            <div class="checkbox">
                <input type="checkbox">
            </div>
            <div class="flie-meta">
                <span class="flie-name">考试答案.doc</span><span class="flie-timestamp">2013-3-28 15：02</span>
            </div>
        </li>
        <li class="item flie-type-doc">
            <div class="checkbox">
                <input type="checkbox">
            </div>
            <div class="flie-meta">
                <span class="flie-name">考试答案.doc</span><span class="flie-timestamp">2013-3-28 15：02</span>
            </div>
        </li>
        <li class="item flie-type-doc">
            <div class="checkbox">
                <input type="checkbox">
            </div>
            <div class="flie-meta">
                <span class="flie-name">考试答案.doc</span><span class="flie-timestamp">2013-3-28 15：02</span>
            </div>
        </li>
        <li class="item flie-type-doc">
            <div class="checkbox">
                <input type="checkbox">
            </div>
            <div class="flie-meta">
                <span class="flie-name">考试答案.doc</span><span class="flie-timestamp">2013-3-28 15：02</span>
            </div>
        </li>
        <li class="item flie-type-doc">
            <div class="checkbox">
                <input type="checkbox">
            </div>
            <div class="flie-meta">
                <span class="flie-name">考试答案.doc</span><span class="flie-timestamp">2013-3-28 15：02</span>
            </div>
        </li>
    </ul>

</div>

<div class="warp">
    <table id="list-view">
        <thead>
        <th colpan="2">文件名称</th>
        <th>文件大小</th>
        <th>上传时间</th>
        </thead>
        <tbody>
        <tr>
            <td>
                <div class="checkbox"><input type="checkbox"></div>
                <span class="flie-type-doc">考试答案.doc</span></td>
            <td><span class="flie-size">100KB</span></td>
            <td><span class="flie-timestamp">2013-3-28 15：02</span></td>
        </tr>
        <tr>
            <td>
                <div class="checkbox"><input type="checkbox"></div>
                <span class="flie-type-doc">考试答案.doc</span></td>
            <td><span class="flie-size">100KB</span></td>
            <td><span class="flie-timestamp">2013-3-28 15：02</span></td>
        </tr>
        <tr>
            <td>
                <div class="checkbox"><input type="checkbox"></div>
                <span class="flie-type-doc">考试答案.doc</span></td>
            <td><span class="flie-size">100KB</span></td>
            <td><span class="flie-timestamp">2013-3-28 15：02</span></td>
        </tr>
        <tr>
            <td>
                <div class="checkbox"><input type="checkbox"></div>
                <span class="flie-type-doc">考试答案.doc</span></td>
            <td><span class="flie-size">100KB</span></td>
            <td><span class="flie-timestamp">2013-3-28 15：02</span></td>
        </tr>
        <tr>
            <td>
                <div class="checkbox"><input type="checkbox"></div>
                <span class="flie-type-doc">考试答案.doc</span></td>
            <td><span class="flie-size">100KB</span></td>
            <td><span class="flie-timestamp">2013-3-28 15：02</span></td>
        </tr>
        </tbody>
    </table>
</div>
</body>
</html>
```

样式表如下(LESS):

```less
*{
  padding: 0;
  margin: 0;
  border:none;
  line-height: 1;
  font-size: 12px;
  color:#000;
  list-style: none;
}
body{
  background:#FDFDFD;
}
div.warp{
  width: 600px;
  height: 400px;
  position: relative;
  margin: 100px auto 0;
  border: 1px solid #E0E0E0;
  background: #F7F7F7;
  ul#grid-view{
    height: 180px;
    margin: 10px;
    li.item{
      float: left;
      height: 168px;
      width: 168px;
      margin: 10px;
      position: relative;
      border: 1px solid #BDBDBD;
      div.checkbox{
        width: 20px;
        height: 13px;
        position: absolute;
        z-index: 2;
        background: url(../img/checkbox.png) 0 -13px no-repeat;
        bottom: 9px;
        left: 6px;
        input[type=checkbox]{
          visibility: hidden;
          position: absolute;
          left: -999em;
        }
      }

      div.flie-meta{
        height: 20%;
        width: 100%;
        background: #7E7E7E;
        bottom: 0;
        position: absolute;
        z-index: 1;
        span{
          display: block;
          padding-left: 25px;
        }
        span.flie-name{
          line-height: 22px;
          font-size: 12px;
          font-weight: bold;
          color: #ECECEC;
        }
        span.flie-timestamp{
          margin-top: -2px;
          color: #CFCFCF;
        }
      }
    }
    li.active{
      border: 4px solid #FF7A00;
      margin: 6px;
      div.checkbox{
        background: url(../img/checkbox.png) 0 0 no-repeat;
      }
    }
    li.flie-type-doc{
      background: url(../img/doc.png) center center no-repeat;
    }
  }
  table#list-view{
    width: 100%;
    max-width: 100%;
    background-color: transparent;
    border-collapse: collapse;
    border-spacing: 0;
    thead{
      th {
        vertical-align: bottom;
        font-weight: bold;
        padding: 8px;
        line-height: 20px;
        text-align: left;
        border-top: 1px solid #DDD;
      }
    }


    td {
      padding: 8px;
      line-height: 20px;
      text-align: left;
      vertical-align: top;
      border-top: 1px solid #DDD;

      div.checkbox{
        width: 20px;
        height: 13px;
        background: url(../img/checkbox.png) 0 -13px no-repeat;
        float: left;
        input[type=checkbox]{
          visibility: hidden;
          position: absolute;
          left: -999em;
        }
      }

      span{
        float: left;
      }
    }
    tr.active{
      div.checkbox{
        background: url(../img/checkbox.png) 0 0 no-repeat;
      }
    }

  }
}
```

接着开始写JS了：

`http://thecdn.sinaapp.com/page/demo/flie-view/index-js.html`

```js
(function($) {
	$(document).ready(function() {
		console.log('START!');
		//切换视图展示
		var switcher = $('div#switcher');
		switcher.on('click', function(e) {
			var target = $(e.target).closest('div.btn');
			if (target.length) {
				switcher.find('div.btn').removeClass('active');
				target.addClass('active');
				var grid = $('div.grid');
				var list = $('div.list');
				if (target.hasClass('show-grid')) {
					list.hide();
					grid.show();
				} else {
					grid.hide();
					list.show();
				}
			}
		});
		//处理关联的视图项目,考虑复杂度还是不合并一起写了,虽然看起来可以合并成一个函数，
		//但是实践的时候，不同的视图有会不同视图的额外事件，所以合并之后，单独的那个函数的复杂度会很高。
		var grid = $('ul#grid-view');
		var list = $('table#list-view');
		var gridCheckboxes = grid.find('input[type=checkbox]');
		var listCheckboxes = list.find('input[type=checkbox]');
		//checkbox个数不相等，说明后端吐数据有问题，则不需要提供视图切换
		var initCheck = function() {
			if (gridCheckboxes.length != listCheckboxes.length) {
				return false;
			} else {
				return true;
			}
		}
		//同步数据
		var syncData = function(type) {
			//同步数据
			if (type == 'grid') {
				gridCheckboxes.each(function(k, v) {
					var target = $(v);
					var state = !! $(v).prop('checked');
					listCheckboxes.eq(k).prop('checked', state);
					if (state) {
						listCheckboxes.eq(k).closest('tr').addClass('active');
					} else {
						listCheckboxes.eq(k).closest('tr').removeClass('active');
					}
				});
			} else {
				listCheckboxes.each(function(k, v) {
					var target = $(v);
					var state = !! $(v).prop('checked');
					gridCheckboxes.eq(k).prop('checked', state);
					if (state) {
						gridCheckboxes.eq(k).closest('li.item').addClass('active');
					} else {
						gridCheckboxes.eq(k).closest('li.item').removeClass('active');
					}
				});
			}
		}
		grid.on('click', function(e) {
			if (!initCheck) {
				return;
			}
			var target = $(e.target).closest('li.item');
			if (target.length) {
				var current = target.find('input[type=checkbox]');
				var state = !! !current.prop('checked');
				current.prop('checked', state);
				if (state) {
					target.addClass('active');
				} else {
					target.removeClass('active');
				}
				syncData('grid');
			}
		});
		list.on('click', function(e) {
			if (!initCheck) {
				return;
			}
			var target = $(e.target).closest('tr');
			if (target.length) {
				var current = target.find('input[type=checkbox]');
				var state = !! !current.prop('checked');
				current.prop('checked', state);
				if (state) {
					target.addClass('active');
				} else {
					target.removeClass('active');
				}
				syncData('list');
			}
		});
	});
})(jQuery)
```

这样就简单的实现了，触发那个视图的checkbox，以那个视图的checkbox为数据源设置另外一个视图的checkbox的效果了。

并且只要两边视图的内容一致，事件绑定也一致。
你可以尝试动态增加tbody>tr和ul>li，看一看效果。:D


完整源码下载:
`http://thecdn.sinaapp.com/page/demo/flie-view/flie-view.zip`

