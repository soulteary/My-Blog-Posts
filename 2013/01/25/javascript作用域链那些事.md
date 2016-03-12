# JavaScript作用域链那些事

最近做项目中的东西,发现了一些有意思的事情，说起javascript的作用域链， 或许大家都知道，但是大家究竟是否真的掌握了呢?

如果你看到了例子中的问题，那么恭喜你,你的javascript很扎实哟！

本篇有引用上一篇中的 [从写自己的小脚本库说起](http://soulteary.com/2013/01/25/write-js-lib.htm) 

如果阅读代码中有疑问,可以先看上一篇。

那客官您准备好瓜子和F12，我们边看码边聊。

我们知道,javascript中作用域链是一个很重要的知识点，这里包含了查找变量，变量使用范围等比较基础的知识。

一个作用域的结束，是以函数执行完毕为标记，并将函数内部的变量，内部函数全部销毁。

那么我们来看几个例子吧。

这个例子有几种情况，为了直观我随便写了一个简单的页面，事件绑定使用之前的文章中的简陋的函数库，有疑问的童鞋，请回头翻阅之前的内容。

先贴上结构和样式。

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8">
		<title>demo</title>
		<style type="text/css">
		*{
			margin:0;
			padding:0;
		}
		div.warp{
			width: 280px;
			height: 80px;
			border: 1px dotted #999;
			padding: 10px;
			background: #eee;
			margin: 70px auto;
		}
		div.warp div.inner-warp {
			position: relative;
			width: 100%;
			height: 100%;
		}

		div.inner-warp div.controls {
			border: 1px dotted #999;
			background: #eee;
			position: absolute;
			padding-left: 20px;
			top: -30px;
			left: -11px;
			width: 100%;
			height: 22px;
			font-size: 12px;
			line-height: 20px;
		}

		div.controls label.radio-mode {
			margin: 2px 16px 0 0;
			float: left;

		}
		div.controls input[type=radio] {
			float: left;
			height: 12px;
			margin: 4px 4px 0 0;
		}

		div.warp textarea#tarTxt{
			width: 200px;
			height: 100%;
			resize: none;
			float: left;
		}

		div.warp button#test-me {
			height: 40px;
			width: 68px;
			float: left;
			margin-left: 8px;
		}
		div.warp button#reset-me {
			height: 40px;
			width: 68px;
			float: left;
			margin-left: 8px;
		}
		</style>
	</head>
	<body>

		<div class="warp">
			<div class="inner-warp">
				<div class="controls">
					<label class="radio-mode">
						<input type="radio" name="radio-mode" data-action="A" checked="true">壹
					</label>
					<label class="radio-mode">
						<input type="radio" name="radio-mode" data-action="B">贰
					</label>
					<label class="radio-mode">
						<input type="radio" name="radio-mode" data-action="C">叁
					</label>
					<label class="radio-mode">
						<input type="radio" name="radio-mode" data-action="D">肆
					</label>
					<label class="radio-mode">
						<input type="radio" name="radio-mode" data-action="E">伍
					</label>
					<label class="radio-mode">
						<input type="radio" name="radio-mode" data-action="F">陆
					</label>
				</div>

				<textarea id="tarTxt" cols="30" rows="10"></textarea>

				<button id="test-me">初始化</button>
				<button id="reset-me">重置</button>

			</div>
		</div>
	</body>
</html>
```

应该还是比较清晰的，会出现一个模块，模块中一个包含一堆radio，一个是textarea和两个按钮。 如果你愿意预览的话，可以到这里去预览：http://thecdn.sinaapp.com/page/demo/scope-chain/ 

我们继续看页面业务代码： 

这里为了直观，没有把6种情况的业务使用switch揉在一起。

```html
<script type="text/javascript">
(function() {

	// 引用之前文章中的迷你脚本库
	// 页面加载的初始化 
	var testBtn = $('test-me');
	var resetBtn = $('reset-me');
	resetBtn.bind('click', function() {
		testBtn.unbind();
		testBtn.bind('click', init);
		testBtn.text('初始化');
		$('tarTxt').text('初始化完毕.');
	});

	// 全局初始化函数
	var init = function() {
			$('tarTxt').text('初始化完毕.');
			// 初始化函数下的局部变量
			var counter = 0;
			// 第一级函数
			var LA1 = function(params) {

					counter++;
					console.log('L1:', params, counter);
					// 第二级函数
					var LA2 = function(params) {

							counter++;
							console.log('L2:', params, counter);
							// 第三级函数
							window.LA3 = function(params) {
								counter++;
								console.log('L3:', params, counter);
							}
							LA3(params)
						}
					LA2(params);
				}

			var LB1 = function(params) {
					counter++;
					console.log('L1:', params, counter);
					var LB2 = function(params) {
							counter++;
							console.log('L2:', params, counter);
							window.LB3 = function(params) {
								counter++;
								console.log('L3:', params, counter);
							}
							// 这个例子中不写CB一样的,因为我直接EVAL了
							$('test-me').ajax({
								url: 'b-ajax.js'
							})
						}
					LB2(params);
				}

			var LC1 = function(params) {
					counter++;
					console.log('L1:', params, counter);
					var LC2 = function(params) {
							counter++;
							console.log('L2:', params, counter);
							window.LC3 = function(params) {
								counter++;
								console.log('L3:', params, counter);
							}
							LC3(params);
						}
					LC2(params);
				}

			var LD1 = function(params) {
					counter++;
					console.log('L1:', params, counter);
					var LD2 = function(params) {
							counter++;
							console.log('L2:', params, counter);
							window.LD3 = function(params) {
								counter++;
								console.log('L3:', params, counter);
							}
							$('test-me').callback({
								id: 'ld',
								url: 'd-callback.js'
							})
						}
					LD2(params);
				}
			var LE1 = function(params) {
					counter++;
					console.log('L1:', params, counter);
					var LE2 = function(params) {
							counter++;
							console.log('L2:', params, counter);
							window.LE3 = function(params) {
								counter++;
								console.log('L3:', params, counter);
							}
						}
					LE2(params);
				}
			var LF1 = function(params) {
					counter++;
					console.log('L1:', params, counter);
					var LF2 = function(params) {
							if(params.indexOf('AJAX') !== -1) {
								eval(params)
							} else {
								counter++;
								console.log('L2:', params, counter);
								window.LF3 = function(params) {
									counter++;
									console.log('L3:', params, counter);
								}

							}
						}
					LF2(params);
					$('test-me').ajax({
						callback: LF2,
						url: 'f-ajax.js'
					})
				}
			var getMode = function() {
					var mode = document.getElementsByName('radio-mode');
					for(var oo in mode) {
						if(mode[oo].getAttribute && mode[oo].checked) {
							return mode[oo].getAttribute('data-action');
						}
					}
				}

			testBtn.unbind();
			var mode = getMode();
			switch(mode) {
			case 'A':

				testBtn.bind('click', function() {
					LA1('A');
					if(LA3) {
						LA3('NEW CLICK: A')
					};
					$('tarTxt').text('执行完毕,请查看CONSOLE.');
				});

				break;
			case 'B':

				testBtn.bind('click', function() {
					LB1('B');
					if(LB3) {
						LB3('NEW CLICK: B')
					};
					$('tarTxt').text('执行完毕,请查看CONSOLE.');
				});

				break;
			case 'C':

				testBtn.bind('click', function() {
					LC1('C');
					if(LC3) {
						LC3('NEW CLICK: C')
					};
					$('tarTxt').text('执行完毕,请查看CONSOLE.');
					$('test-me').ajax({
						url: 'c-ajax.js'
					})
				});

				break;
			case 'D':
				testBtn.bind('click', function() {
					LD1('D');
					if(LD3) {
						LD3('NEW CLICK: D')
					};
					$('tarTxt').text('执行完毕,请查看CONSOLE.');
				});
				break;
			case 'E':
				testBtn.bind('click', function() {
					LE1('E');
					if(LE3) {
						LE3('NEW CLICK: E')
					};
					$('tarTxt').text('执行完毕,请查看CONSOLE.');
					$('test-me').callback({
						id: 'le',
						url: 'e-callback.js'
					})
				});
				break;
			case 'F':
				testBtn.bind('click', function() {
					LF1('F');
					if(LF3) {
						LF3('NEW CLICK: F')
					};
					$('tarTxt').text('执行完毕,请查看CONSOLE.');
				});
			}
			testBtn.text('进行测试');
		}

	testBtn.bind('click', init);

})('SOULTEARY');
</script>
```

然后是脚本ajax/callback请求返回的内容 `b-ajax.js`

```js
LB3('AJAX: B');
```

`c-ajax.js`

```js
LC3('AJAX: C');
```

`d-callback.js`

```js
LD3('CALLBACK: D');
```

`e-callback.js`

```js
LE3('CALLBACK: E');
```

`f-ajax.js`

```js
LF3('AJAX: F');
```

接着你可以一边打开预览页面，一边跟着文中的线索去做。 

接下来的描述都是在一个闭包中进行动态解除绑定和添加绑定，以排除外部函数和变量对内部的干扰。 

预览页面: http://thecdn.sinaapp.com/page/demo/scope-chain/ 

首先选模式1, 我们点击初始化按钮，把初始化事件中的新事件绑定到按钮中。 

然后点击按钮若干次，随你的意:D 文本框内显示`执行完毕,请查看CONSOLE.` F12查看CONSOLE,查看记录如下:

```text
L1: A 1
L2: A 2
L3: A 3
L3: NEW CLICK: A 4
L1: A 5
L2: A 6
L3: A 7
L3: NEW CLICK: A 8 
```

我们返回头查看代码

```js
// 初始化函数定义的内容

// init 下的局部变量
var counter = 0;
// 第一级函数
var LA1 = function(params) {
		counter++;
		console.log('L1:', params, counter);
		// 第二级函数
		var LA2 = function(params) {

				counter++;
				console.log('L2:', params, counter);
				// 第三级函数
				window.LA3 = function(params) {
					counter++;
					console.log('L3:', params, counter);
				}
				LA3(params)
			}
		LA2(params);
	}


// 绑定的事件
testBtn.bind('click', function() {
	LA1('A');
	if (LA3) {LA3('NEW CLICK: A')};
	$('tarTxt').text('执行完毕,请查看CONSOLE.');
});

```

这里LA1是init函数内部的函数，没有挂载在init的原型上或公开给window或任何全局对象，LA2同LA1，为了方便描述，以后的LA1/LB1/LC1...我们称呼为一级函数,LA2/LB2/LC2...为二级...以此类推

三级函数则挂载到window对象属性中,在点击按钮后,首先触发一级函数，接着一级函数定义并调用二级函数，二级函数定义并调用三级元素。

执行完毕后，我们再次对三级函数进行调用，发现counter依然被+1;但是例子1不是很明显,我们把差异放大了看。


操作如例子1，例子2的console情况

```text
L1: B 1
L2: B 2
L3: NEW CLICK: B 3
XHR finished loading: "b-ajax.js?c=7744".
L3: AJAX: B 4
L1: B 5
L2: B 6
L3: NEW CLICK: B 7
XHR finished loading: "b-ajax.js?c=505521".
L3: AJAX: B 8 
```

我们来看代码实现

```js
var LB1 = function(params) {
		counter++;
		console.log('L1:', params, counter);
		var LB2 = function(params) {
				counter++;
				console.log('L2:', params, counter);
				window.LB3 = function(params) {
					counter++;
					console.log('L3:', params, counter);
				}
				// 这个例子中不写CB一样的,因为我直接EVAL了
				// 这个定义在上文和之前一篇有提到
				$('test-me').ajax({url:'b-ajax.js'})
			}
		LB2(params);
	}

testBtn.bind('click', function() {
	LB1('B');
	if (LB3) {LB3('NEW CLICK: B')};
	$('tarTxt').text('执行完毕,请查看CONSOLE.');
});

```

这个按钮执行的事件是这样的，调用了一级函数之后，一级函数调用二级函数，二级函数定义了三级全局函数，并使用ajax方法在成功后eval调用全局三级函数。 

根据执行结果，我们知道，我了个去，counter又被+1了。 

如果你现在迷惑了的话，不妨继续看，不着急，我们还有4个例子。 

如果你不迷惑的话，可以直接看文末或者关闭窗口了，因为再往下面看，收益也不大。 

关于第三个例子，是这个样子的。

```text
L1: C 1
L2: C 2
L3: C 3
L3: NEW CLICK: C
XHR finished loading: "c-ajax.js?c=260100".
L3: AJAX: C 5
L1: C 6
L2: C 7
L3: C 8
L3: NEW CLICK: C 9
XHR finished loading: "c-ajax.js?c=99856".
L3: AJAX: C 10 
```

代码实现：

```js
var LC1 = function(params) {
		counter++;
		console.log('L1:', params, counter);
		var LC2 = function(params) {
				counter++;
				console.log('L2:', params, counter);
				window.LC3 = function(params) {
					counter++;
					console.log('L3:', params, counter);
				}
				LC3(params);
			}
		LC2(params);
	}

testBtn.bind('click', function() {
	LC1('C');
	if (LC3) {LC3('NEW CLICK: C')};
	$('tarTxt').text('执行完毕,请查看CONSOLE.');
	$('test-me').ajax({url:'c-ajax.js'})
});
```

事件的执行是这个样子滴：

调用一级函数后，依次定义了二级和三级函数，并逐次调用。

不同的是，按钮点击后，我们使用ajax的成功返回时的eval进行调用全局三级函数。（好累，好长，呼呼...） 

一个看似不走运的结果，我们的counter依然被+1; 难道这货和十万个冷笑话中的王二一样，拥有百分百被加数值加一的武林失传已久的绝技！？ 

显然...不是的，我们继续往下看，还有两个例子。 

第四个例子：

```text
L1: D 1
L2: D 2
L3: NEW CLICK: D 3
L3: CALLBACK: D 4
L1: D 5
L2: D 6
L3: NEW CLICK: D 7
L3: CALLBACK: D 8 
```

代码在此：

```js
var LD1 = function(params) {
		counter++;
		console.log('L1:', params, counter);
		var LD2 = function(params) {
				counter++;
				console.log('L2:', params, counter);
				window.LD3 = function(params) {
					counter++;
					console.log('L3:', params, counter);
				}
				$('test-me').callback({id:'ld', url:'d-callback.js'})
			}
		LD2(params);
	}

testBtn.bind('click', function() {
	LD1('D');
	if (LD3) {LD3('NEW CLICK: D')};
	$('tarTxt').text('执行完毕,请查看CONSOLE.');
});
```

二级函数在创建之后没有直接调用三级函数，而是创建了一个callback使用了外部方法来执行三级全局函数。

 很不幸，我们猜错了么，这货有百分比数值被加一的绝技?! 我们继续来看吧。 
 
 第五个绝技，哦不，是第五个例子：

```text
L1: E 1
L2: E 2
L3: NEW CLICK: E 3
L3: CALLBACK: E 4
L1: E 5
L2: E 6
L3: NEW CLICK: E 7
L3: CALLBACK: E 8 
```

代码实现：

```js
var LE1 = function(params) {
		counter++;
		console.log('L1:', params, counter);
		var LE2 = function(params) {
				counter++;
				console.log('L2:', params, counter);
				window.LE3 = function(params) {
					counter++;
					console.log('L3:', params, counter);
				}
			}
		LE2(params);
	}
testBtn.bind('click', function() {
	LE1('E');
	if (LE3) {LE3('NEW CLICK: E')};
	$('tarTxt').text('执行完毕,请查看CONSOLE.');
	$('test-me').callback({id:'le', url:'e-callback.js'})
});
```

例子老五，定义了一级二级三级后，一级二级函数依次调用，三级函数在按钮点击中创建callback，调用全局三级函数。 

OMG, 我们依然没有阻拦到counter+1的趋势，这货要是中国股票该多好，挡不住的涨势- -！ 

如果你看到这里还是没有想明白的话，那么，继续看完老六吧。 

如果你现在中途离开，可能会获得一个错误的结论：死掉的函数，销毁的作用域链被复活了，或者其他更诡异的结论。 

最后一个例子来了：

```text
L1: F 1
L2: F 2
L3: NEW CLICK: F 3
XHR finished loading: "f-ajax.js?c=140625".
L3: AJAX: F 4
L1: F 5
L2: F 6
L3: NEW CLICK: F 7
XHR finished loading: "f-ajax.js?c=134689".
L3: AJAX: F 8 
```

代码如下：

```js
var LF1 = function(params) {
		counter++;
		console.log('L1:', params, counter);
		var LF2 = function(params) {
				if(params.indexOf('AJAX') !== -1) {
					eval(params)
				} else {
					counter++;
					console.log('L2:', params, counter);
					window.LF3 = function(params) {
						counter++;
						console.log('L3:', params, counter);
					}

				}
			}
		LF2(params);
		$('test-me').ajax({callback: LF2,url:'f-ajax.js'})
	}
testBtn.bind('click', function() {
	LF1('F');
	if (LF3) {LF3('NEW CLICK: F')};
	$('tarTxt').text('执行完毕,请查看CONSOLE.');
});
```
这里是之前的ajax调用的扩展版本，ajax请求后，把事件传给某个苦力函数，上面的过程中，二级函数好像出镜率比较低，我们就用它了。 

一级函数定义后，二级函数定义，初始化的时候，传入参数么有ajax这个关键词，于是定义全局函数三，接着按钮点击调用一级事件，一级事件链式调用二级事件，以及创建一个ajax，ajax请求回调二级函数，接着二级函数再次调用三级函数... 

嗯，最后一个例子，很遗憾，counter依然没有被逆袭，还是自增了一位数。 

但是结论并没有就此被扭曲，作用域链没有被复活。 执行环境，是我们的变量和函数的作用域的根本，某个环境中的函数执行完毕后，该环境中的变量和事件都会被销毁。（个别浏览器个别闭包销毁不干净例外...本文不讨论） 

上面的例子非但没有把这个结论推翻，反而更加印证了上面的事实。 

当定义一级函数，一级函数定义二级函数并调用后，二级函数定义全局函数的时候，整个函数的环境就被长期扩展在了window，即全局环境下。 

全局环境只有在窗口关闭，或者刷新才会全部销毁. 你是不是想说，你的例子无法印证你的观点呢，说的好（如果你说了的话），今天天气不错，我刚好有一个修正版的例子，我们不妨再次运行下例子1~6. 

地址在此：http://thecdn.sinaapp.com/page/demo/scope-chain2/index.html 

初始化后，当你点击第一次之后，再次执行完毕是否会报以下的错误呢？

 `Uncaught TypeError: object is not a function` 
 
扩展讨论话题，闭包是不是也不一定是绝对安全的了呢。
我们在项目中，如果私有过程要打开，打开后，一定记得关闭，否则会有隐患。

那么，文章就此结束，有疑惑欢迎留言讨论。
作者水平略菜，有不当之处，欢迎指出，我会努力改正和改进。


插播一条广告，新浪总部-新浪云计算 目前招技术实习生。

我也是这里实习生之一，现在我要吐槽:SAE怎么能这样！大牛怎么能手把手教实习生！从业若干年的架子那里去了！怎么能不像别的地方让实习生无限看书度过实习期！怎么能给实习生项目去实战中成长！怎么能不分上下级一起玩呢！实习生租不到房子！怎么能大家帮忙转发租房信息...

想不想在实习的时候，就参与非不牛逼的项目，然后在实习简历上低调的写上SAE呢。

顺便一说，这里早餐很便宜，网速很快，前一阵42G神马下载事件的据说45分钟...具体你懂的

还有就是...你来了我偷偷告诉你，还在等神马, 大三大四的童鞋果断投递简历吧! 

SAE微博招募贴:http://weibo.com/1220149481/zfRoNuDqM

之前大家帮忙转发租赁信息的帖:http://weibo.com/1220149481/zdE5p91JS

关于租房,其实很多童鞋(大牛)都帮忙找人,想办法的,现实中的SAE团队比网上更好相处!

期待和优秀你的一起吃饭，一起写码，一起进步！我们在SAE，你在那里?

