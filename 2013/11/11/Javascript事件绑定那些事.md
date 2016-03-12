# Javascript事件绑定那些事

## 写在前面

前一阵拜读了@民工精髓V 前辈的[文章](http://www.ituring.com.cn/article/60218?q=%E4%BB%8E%E9%9B%B6%E5%BC%80%E5%A7%8B%E4%BD%BF%E7%94%A8JavaScript%E7%BC%96%E5%86%99%E6%95%B0%E6%8D%AE%E8%A1%A8%E6%A0%BC%E6%8E%A7%E4%BB%B6 "从零开始使用JavaScript编写数据表格控件")，微博有了一些交流，但是不敢苟同现在这个年代还要一点一点进化代码，而不是直接使用较新的方式，接着又答应了@easy 大叔，所以简单的写一下我眼中的事件绑定吧。（不要吐槽这个奇怪的逻辑）

## 一些例子

类似《从零开始使用JavaScript编写数据表格控件》这篇文章中会为了初学爱好者便于理解，有`someone.onclick = function`这样的代码，这样对于一些业务极其简单和固定的页面没有问题，只是维护起来呢，略微有点不妥。 我比较推崇这样来实现事件绑定，10个月前曾写过一篇粗浅的文章《[从A标签说开去：链接那些事](http://www.soulteary.com/2013/01/11/let-talk-about-tag-a.html)》 如果童鞋你的英文水平以及可以打开google translate的话，不妨看看这个《[Call of the wild (scripts)](http://www.onlinetools.org/articles/unobtrusivejavascript/chapter4.html)》，这篇文章也简单的讲了一下不要怎么使用事件绑定。

## 啰嗦的开始

任何技巧都要建立在合适的上下文中，那我给我们一个假定的需求吧：做一个内容可能会变动的表格，表格内有可以点击交互的元素，表格的内容可以添加和删除。 我们来分析一下这个需求，如果使用民工前辈的方法，每个元素都添加onclick，然后会发生什么呢，我来写一段代码，大家看一下。

```html
<!doctype html>
<html lang="en-US">
<head>
    <meta charset="UTF-8">
    <title>如果你使用onclick来绑定事件的话</title>
</head>
<body>
<table>
    <tbody>
    <tr>
        <td><button>1</button></td>
    </tr>
    <tr>
        <td><button>2</button></td>
    </tr>
    <tr>
        <td><button>3</button></td>
    </tr>
    </tbody>
</table>
<script type="text/javascript">
    //选取所有的元素
    var btns = document.getElementsByTagName('button');
    for(var i=0, j=btns.length; i<j; i++){
        var target = btns[i];
        //挨着给他们用onClick绑定事件
        target.onclick = function(){
            alert('someone like u');
        }
        //移除页面中的这些家伙们，模拟删除操作
        target.parentNode.removeChild(target);
        //触发事件
        target.onclick()
    }
</script>
</body>
</html>
```

在线预览:http://thecdn.sinaapp.com/page/demo/link-test2-1/test1.html

代码很简单，甚至简陋，但是足够说明问题了，即使元素干掉了，DOM内存中还是有绑定的东西，如果绑定了10万个事件呢...那么这个句柄是不是保存10w份呢...那如果20万呢,100万呢...

当页面不刷新的时候，事件会越来越多，而浏览器给我们分配的内存是有限的，先不说你的脚本能不能写到牛逼的把内存占满，乃至窗口崩溃，就单说让页面卡卡的也不好吧。（chrome v26版本，我曾写死循环，让可爱的win8蓝屏过...）

那么我们来改进一下这个东西吧。

```html
<!doctype html>
<html lang="en-US">
<head>
    <meta charset="UTF-8">
    <title>如果你使用onclick来绑定事件的话</title>
</head>
<body>
<table id="event-table">
    <tbody>
    <tr>
        <!--我们给这些熊孩子添加了一些属性 data-cmd ，并且后面的数值不一样 -->
        <td><button data-cmd="HELLO">1</button></td>
    </tr>
    <tr>
        <td><button data-cmd="HI">2</button></td>
    </tr>
    <tr>
        <td><button data-cmd="BYE">3</button></td>
    </tr>
    </tbody>
</table>
<script type="text/javascript">
    //这次我们不直接对页面中的元素进行操作，
    //我们选择直接对话他们的爹地，
    //由这些熊孩子的监护人table来处理事件
    //入口合并，代理事件
    var box = document.getElementById('event-table');
    //监听click事件
    box.addEventListener('click', function(e){
        //获取当前被点击的元素
        var target = e.target;
        //这里可以先针对不同的元素进行不同的获取参数的方法，稍后添加
        var cmd = target.getAttribute('data-cmd');
        //如果不满足条件，尽早跳出过程
        if(!cmd){return;}
        //阻止符合条件的元素的冒泡以及默认行为
        e.preventDefault(),e.stopPropagation();
        //根据我们设置的cmd属性不一样，执行不同的操作
        switch(cmd){
            case 'HELLO':
                alert('hello!');
                break;
            case 'HI':
                alert('hi!');
                break;
            case 'BYE':
                //同样执行删除，但是会干净好多
                target.parentNode.removeChild(target);
                alert(target.onclick);
                break;
        }
    }, false);
</script>
</body>
</html>
```

在线地址：http://thecdn.sinaapp.com/page/demo/link-test2-1/test2.html

如果你对项目感兴趣，可以看这里：http://www.soulteary.com/2013/05/21/error-tracer.html

我们看到了，这个例子中第三个按钮按了之后，删除掉自己，它身上的句柄为null，没有任何污染和遗留。 并且在写业务逻辑的时候，我们可以专注的在一个入口中写，就是box的click绑定事件，而不需要针对每个不同的元素写好多个过程。 入口一旦减少了，某些时候复杂度也就下来了。 并且，使用标签属性的方式来定义“命令”，可以在脚本中也使用同样的“命令”去定义我们的功能，方便查找，而不需要去明确那个id，那个class对应的元素该干嘛，减少了我们页面中的事件耦合性。

说了这么多，我偷个懒，截张图。

[![2013-11-11_022449](https://attachment.soulteary.com/2013/11/11/2013-11-11_022449.png "2013-11-11_022449")](https://attachment.soulteary.com/2013/11/11/2013-11-11_022449.png)

这个页面中源码是这样的：

```html
<!DOCTYPE html>
<html lang="zh_CN">
<head>
    <meta charset="utf-8">
    <title>后台管理</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link href="../static/css/bootstrap.min.css" rel="stylesheet">
    <link href="../static/css/style.css" rel="stylesheet">
    <script type="text/javascript" src="../static/js/jquery-1.10.2.min.js"></script>
    <script type="text/javascript" src="../static/js/moment.min.js"></script>
    <script type="text/javascript" src="../static/js/jstorage.min.js"></script>
    <script type="text/javascript" src="../static/js/bootstrap.min.js"></script>
</head>
<body>
<div class="progress progress-striped active hide" id="page-loader">
    <div class="bar" style="width: 0;"></div>
</div>
<div class="container-fluid">
    <div class="row-fluid">
        <h1 id="logo" class="pull-right">Error Tracer</h1>
    </div>
    <div class="row-fluid">
        <div class="span12">
            <ul class="nav nav-tabs" id="control-nav">
                <li class="active"><a href="#CMD:MSG">Message</a></li>
                <li><a href="#CMD:SCRIPT">Script</a></li>
                <li><a href="#CMD:BROWERS">Browers</a></li>
                <li><a href="#CMD:Page">Page</a></li>
            </ul>
        </div>
    </div>
    <div class="row-fluid">
        <table class="table table-striped" id="data-table"></table>
    </div>
</div>
<script type="text/javascript" src="../static/js/application.js"></script>
<script type="text/html" id="table-msg-tpl">
    <thead>
    <tr>
        <th class="status"><span class="status invalid">status: invalid</span></th>
        <th class="message">Message</th>
        <th class="file">Source File</th>
        <th class="browser"></th>
        <th class="times">Times</th>
        <th class="lifespan">Lifespan</th>
        <th class="report">Last Report</th>
    </tr>
    </thead>
    <tbody>
    <tr>
        <td colspan="7" class="loading-text">载入中...</td>
    </tr>
</script>
<script type="text/html" id="table-browser-tpl">
    <thead>
    <tr>
        <th class="browser">Type</th>
        <th class="times">Times</th>
        <th class="lifespan">Lifespan</th>
        <th class="report">Last Report</th>
    </tr>
    </thead>
    <tbody>
    <tr>
        <td colspan="7" class="loading-text">载入中...</td>
    </tr>
</script>
<script type="text/html" id="table-script-tpl">
    <thead>
    <tr>
        <th class="script-file">File</th>
        <th class="script-times">Times</th>
        <th class="script-lifespan">Lifespan</th>
        <th class="script-report">Last Report</th>
    </tr>
    </thead>
    <tbody>
    <tr>
        <td colspan="7" class="loading-text">载入中...</td>
    </tr>
</script>
<script type="text/html" id="common-modal-tpl">
<div class="modal-header">
    <button type="button" class="close" data-dismiss="modal" aria-hidden="true">&times;</button>
    <h3>Modal header</h3>
</div>
<div class="modal-body"></div>
<div class="modal-footer"><button type="button" class="btn btn-primary" data-dismiss="modal" aria-hidden="true">OK</button></div>
</script>
</body>
</html>
```

我们看到里面的所有数据内容都是动态插入（如果有童鞋需求，或许我会写一点动态模版的事情），并且有一些链接，如果你点开[这个页面](http://errorboard.sinaapp.com/push/?mode=admin)，你会看到显示的链接的**[R]**，这个东西的href属性为**#CMD:REFER-INFO**, 如果我不想要这个链接了，只需要把页面模版中的[R]相关的节点删除，CSS中[R]这个家伙相关的内容删除（如果有），然后JS中删除REFER-INFO后的一段内容就可以了。 而不需要去一段一段的翻阅那些id，class，如果你有不止一个html模版/css文件/js脚本，那么使用id,class的方式来做，就是噩梦...

比如，我在刚刚的例子中只是需要跑到js脚本中，把下面的内容删除，就干干净净的了。

```js
//一段字符串
'<a href="#CMD:REFER-INFO" data-url="'+result[oo]['refer']+'" title="'+result[oo]['refer']+'">[R]</a>'
//一个switch分支
case 'REFER-INFO':
   window.open($(e.target).data('url'),'refer-url');
   break;
```

好了，关于事件绑定，我就简单的说到这里了，由于个人水平有限，肯定有许多不足和疏漏，欢迎继续讨论更合理的方案。

## 项目使用

目前这套方案已经使用在新版的[SAE在线管理平台](http://sae.sina.com.cn)，[支教联盟的报名管理页面](http://www.go9999.com)，以及一堆个人随意写的小项目等。

--EOF-- by 苏洋 2013.11.11

