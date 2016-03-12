# 从A标签说开去：链接那些事

[![tag-a](https://attachment.soulteary.com/2013/01/12/tag-a.jpg "tag-a")](https://attachment.soulteary.com/2013/01/12/tag-a.jpg)

A标签式诞生最早的一批元素，堪称HTML语言中的元老。 我们常常在各种地方看到它的身影，比如说：

> 兄弟，我看你的站气宇轩昂，骨骼惊奇，隐约之中朦朦胧胧透漏出一股王霸之气，咱们加个友链吧，我的站是：http://www.soulteary.com

这里的交换链接在HTML中的最基本表现形式就是：

```
[苏洋博客](http://www.soulteary.com)
```

类似这样的链接（A标签）随处可见，淘宝里点击宝贝图片跳转到对应宝贝的详情页面；搜索引擎的结果列表中，点击一下，带你进入一个新的世界；SAE里的某些看着像是按钮的东西，点击一下，执行一些用户不使用调试工具看不到的信息...

作为前端爱好者，通过查看各个网站的源码，可以看到我们在追求语义化和良性结构的同时，慢慢的在简单环境中抛弃了使用默认标准控件button

```
<button>我是按钮啊，亲</button>
```

而是越来越多的使用A标签来进行基础构建。

A标签在W3C中的主要有两处（元素定义）

- HTML4中，描述仅仅为 anchor，也就是链接元素，相关地址： http://www.w3.org/TR/html4/struct/links.html#edef-A 
- HTML5中描述变为了hyperlink，也就是我们说的超链接，相关地址： http://dev.w3.org/html5/markup/a.html

那么，我们使用A标签替代button之后，又会有哪些情况呢？我们要如何使用A标签来代替button的动作呢，尤其是在不离开当前页面的时候。

我们应该选择怎么样的方案呢？ 如何替代掉button的动作，这里，我列了几种常见的写法。

```


<div id="link-wrap">

	[A](javascript:test();)

	[B](javascript:;)

	[C](javascript:;)

	[D](http://github.com)

	[E](#CMD:TEST)

	[F](about:blank)

	<a id="g" href="">F</a>
</div>

```

看完上面的源码，大家先别吐槽，有的写法是刻意为之，稍后我们讨论。

如果你没有发现问题所在，可以参考RFC 3987和之前关于A的定义的文档标准。

那么对应上面的结构，下面的测试脚本很随意就写出来了。

```js
//用户直接调用的函数
function test(){
	var msg = 'func is called.';
	if(console && console.log){
		console.log(msg);
	}else{
		alert(msg);
	}
}

var addEvent = function(obj, evt, func){
	typeof obj
	if(!!obj.attachEvent){
		obj.attachEvent('on'+evt, func);
	}else{
		obj.addEventListener(evt, func, false);
	}
}

//这里开始注册绑定事件
var linkB = document.getElementById('b');
	addEvent(linkB, 'click', test);
var linkC = document.getElementById('c');
	addEvent(linkC, 'click', test);
var linkE = document.getElementById('e');
	addEvent(linkE, 'click', test);
var linkF = document.getElementById('f');
	addEvent(linkF, 'click', test);
var linkG = document.getElementById('g');
	addEvent(linkG, 'click', test);
```

测试地址：http://thecdn.sinaapp.com/page/demo/link-test/ 

但是这里只是最基础的测试，一会这个测试脚本还需要升级一下。 

我们先对chrome这类WebKit浏览器进行测试 标签A~E输出:_func is called._ 

但是E将链接地址更改为:http://thecdn.sinaapp.com/page/demo/link-test/#CMD:TEST

F跳转到了空白页，G在输出下面内容后刷新了页面。 

_func is called._

那我们来看看IE吧。 IE7~10，和chrome表现一致。(我是故意放弃IE6的) 然后我们升级一下脚本。 兼容的XHR对象创建我就偷懒了，使用以前写的一个小库。

可以在demo页面中查看然后对应的业务脚本。

```js
function test(){
	var msg = 'func is called.';
	if(console && console.log){
		console.log(msg);
	}else{
		alert(msg);
	}
	//测试XHR
	var soulteary = new HTTP.LIB.APPLICATION(14, '/data.js');
	soulteary.get({},function(msg){
			if(console && console.log){
				console.log(msg);
			}else{
				alert(msg);
			}
		});
	//顺便看调用的元素的id属性
	console.log(this.id)
}
```

这个代码还是留了坑，为什么这样，稍后解释。 测试地址：http://thecdn.sinaapp.com/page/demo/link-test2/

还是先用chrome开刀，啊不是，是测试： 标签A~E第一次都正常，和之前无异，并输出了ajax请求后的返回值： _{'code':'javascript'};_ 

但是再点击E之后，A~D再次请求，因为地址变化了，AJAX实际请求也从： _http://thecdn.sinaapp.com/page/demo/link-test2/data.js_ 变为了： _http://thecdn.sinaapp.com/page/demo/link-test2/_ 所以请求也变为了一个不存在的地址。

因为测试地址是新浪云的应用，而我设置错误地址请求跳转我的blog，所以返回值会是我的博客首页。

接着继续说标签F，在chrome中打开Console设置，激活Log XMLHttpRequests和Preserve log upon navigation两个选项。

标签F没有等到ajax踩着七彩祥云归来，直接跳转到了新的页面，但是跳转之前输出了 _func is called._ 标签G，在刷新页面之前，输出了 _func is called._ 并请求了 _http://thecdn.sinaapp.com/page/demo/link-test2/data.js_ 并且得到了返回数据 _{'code':'javascript'};_ 

接着继续用IE测试： IE10 A~E和之前无差异，区别是，E点击之后使用调试工具捕获URL发现地址为： _/page/demo/link-test2/%23CMD:TEST/data.js_ 之后再点击A~E和chrome一样，由于是无效地址，你懂的。

如果你看到这里，我估计你应该骂博主二X青年好多次了吧。

但是实际上很多人写函数，绑定函数，确实在使用元素后，没有统一的取消冒泡（和本情景无关）以及取消默认动作（和本情景极度有关）。

那么我们再次更新业务脚本，博主继续偷个懒(偷懒的目的是为了大家一起测试其他浏览器)，大家应该不介意吧。

```
 <script type="text/javascript">//用户直接调用的函数
	function test(){
		var msg = 'func is called.';
		if(console && console.log){
			console.log(msg);
		}else{
			alert(msg);
		}

		//测试XHR
		var soulteary = new HTTP.LIB.APPLICATION(14, '/data.js');
		soulteary.get({},function(msg){
				if(console && console.log){
					console.log(msg);
				}else{
					alert(msg);
				}
			});
	}

	//这里开始注册绑定事件
	var linkB = document.getElementById('b');
		linkB.addEventListener('click', function(e){
			test();
			e.preventDefault();
		}, false);
	var linkC = document.getElementById('c');
		linkC.addEventListener('click', function(e){
			test();
			e.preventDefault();
		}, false);
	var linkE = document.getElementById('e');
		linkE.addEventListener('click', function(e){
			test();
			e.preventDefault();
		}, false);
	var linkF = document.getElementById('f');
		linkF.addEventListener('click', function(e){
			test();
			e.preventDefault();
		}, false);
	var linkG = document.getElementById('g');
		linkG.addEventListener('click', function(e){
			test();
			e.preventDefault();
		}, false);</script> 
```

测试地址：http://thecdn.sinaapp.com/page/demo/link-test3/ 这次，所有的地址都不会修改，且都不会跳转了，获取数据也正常了。

和之前的区别在哪里呢？ **event.preventDefault();** 如果我们要使用a标签做按钮使用，那么一定要记得阻止默认事件。 

或许你会说，你这个写的麻烦，我一个_return false_就解决问题了， 如果真的是这样的话，那么请自行尝试，至少我测试chrome中第三个demo中，绑定事件使用_return false_不可以阻止默认事件。 

当然，如果你写在元素的内联事件如onclick中的_return false_还是可以阻止默认的。 

哦，那么这些测试是毫无价值的么，显然不是，上面这段测试，显示了如果使用良性的html/css/js三层分离的注册事件来驱动页面， 那么使用a作为事件驱动的元素，务必加上**event.preventDefault();** 到这里该谈的基本都谈完了，那么我们来谈谈具体的事情吧。 

上面的A~G几种形式的写法，各有利弊，在不同的环境或许都是最有解。 我们先谈谈好处： 

- 第一种：直接调用页面中的全局函数，简单粗暴，无论什么时候都不会修改url，发生上面例子中的url改变的事情发生。 
- 第二种：调用一个noop函数，即空函数，类似写法还有void('0');可参考o‘reilly ajax hacks那本书，由于没有写死调用的函数，我们可以通过绑定事件把动作附加到元素上，也不会发生上面改变url的事情发生。 
- 第三种：兼容第二种的好处之上，防止写码的时候忽略了preventDefault的处理。 
- 第四种：传统的简单粗暴写法/ 
- 第五种：使用锚点+绑定事件的方法来进行动作附加，即使js脚本没有下载完毕，也不会发生报错，而且符合三层分离的原则。而且锚点也可以表现语义化动作。 
- 第六种：某大牛发现了 “空路径对页面性能影响的解决方案”，有兴趣可以搜索并测试，如果你测试了，不妨告诉我，谢谢。（不确定会对a的href也产生影响） 
- 第七种：完全的无额外内容。


然后世界开始分裂，我要开始吐槽了：

- 第一种：直接调用伪协议的方式调用函数，现代浏览器中一致性很好，但是不确定在古老的浏览器中可以保持一致性。而且直接调用只能调用全局函数，不利于函数结构组织。 
- 第二种：集成第一种的劣势。 
- 第三种：集成第一种的劣势，且没有考虑三层分离。 
- 第四种：且没有考虑三层分离。 
- 第五种：如果脚本没用下载完毕，会修改url hash，如果协同写码，别人使用到url操作，且没有做去除hash处理，几率发生问题，如之前的测试。 
- 第六种：不论会不会产生页面性能积极影响，about:blank作为额外的无语义内容就是让人不爽的。 
- 第七种：这货不是一个标准的a标签，还记得之前说要看RFC和W3C标准嘛？ 可能有的童鞋懒的找，我把内容贴一下：

> Non-empty URL potentially surrounded by spaces A URL that is not the empty string, optionally with leading and/or trailing space characters. Absolute URL potentially surrounded by spaces A valid IRI as defined in [RFC 3987], optionally with leading and/or trailing space characters.

接着我们在来看一个小东西，事件绑定次数，即句柄数量： 

- 第一种：0，环保 
- 第二种: 1 
- 第三种: 2 周全的代价 
- 第四种: 1 
- 第五种: 1 
- 第六种: 1 
- 第七种: 1

我发现我有话痨属性，多提一个事情吧：

- 淘宝首页使用javascript: 10处
- 淘宝首页使用#: 10处
- Github内页首页使用javascript: 0处
- Github内页首页使用#: 9处
- 360首页使用javascript: 0处
- 360首页使用#: 7处
- 腾讯首页使用javascript: 37处
- 腾讯首页使用#: 11处
- 新浪首页使用javascript: 26处
- 新浪首页使用#: 0处

其他的大家自己搜索的玩呗~~

现在都流行开放式结局，那么，文章就此EOF。
至于选择哪个方案，我觉得大家都心里有数了，欢迎讨论~

