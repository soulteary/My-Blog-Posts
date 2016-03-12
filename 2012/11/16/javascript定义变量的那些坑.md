# javascript定义变量的那些坑

发一个很简单的例子，之前线上代码有个错误，因为发布的时候的失误，重复包含了两个版本的common库。

然后发生了一些看似诡异的事情，其实，事情并不是无迹可寻。

出现的原因，其实是因为javascript的预编译机制，也或者说，是变量定义的机制问题。

这个东西如果要讨论的话，我觉得一个我道行太浅，再一个，网上有大段可以搜索到的，就不需要赘述了。

代码说话吧~ 把下面的测试代码扔进你的浏览器的console中，回车运行，你会看到：

```js
(function() {
	//调用 定义式的someFunc #4 (#4覆盖掉#2)
	someFunc();

	var someFunc = function() {
		console.log('#1.function defined by var.');
	}

	//调用变量赋值式的someFunc
	someFunc();

	function someFunc() {
		console.log('#2.normal.');
	}

	//定义式因为先运行已经被变量复制
	someFunc();

	var someFunc = function() {
		console.log('#3.function defined by var.');
	}

	//变量式#3覆盖#1
	someFunc();

	function someFunc() {
		console.log('#4.normal.');
	}

	//依旧是#3
	someFunc();

})();
```

输出结果应该不外如是是这样：

```js
#4.normal.
#1.function defined by var.
#1.function defined by var.
#3.function defined by var.
#3.function defined by var.
```

至于为什么，我想注释已经很明白了。 顺便发一个javascript前端面试题，感觉其实这个考察还是很必要..

在程序执行完毕后，变量a,b,c的数值是多少呢，比较简单，不过如果你不确定你知道正确答案的话，可以在浏览器里跑一下。

从这里开始基本不说答案了，因为你只需要把源码复制，扔浏览器里，答案就出来了。

```js
(function() {

	var a = 1, b = c = 0;

	function x(n){n = n + 1;}

	b = x(a);

	function x(n){n = n + 3;}

	c = x(a);

	//你认为a, b, c是多少呢?
	console.log('result:', a, b, c);

})();
```

简单变一下，答案略有改动，同时也给出了上面的那道题如果出错了的同学的原因。

```js
(function() {

	var a = 1, b = c = 0;

	function x(n){return n + 1;}

	b = x(a);

	function x(n){return n + 3;}

	c = x(a);

	//你认为a, b, c是多少呢?
	console.log('result:', a, b, c);

})();
```

```js
(function() {

	var a = 1, b = c = 0;

	function x(n){return n + 1;}

	b = x(a);

	function x(n){return n + 3;}

	c = x(a);

	//你认为a, b, c是多少呢?
	console.log('result:', a, b, c);

})();
```

再综合考察一下，这段代码会有什么问题。

提示，javascript预编译会将要声明的变量初始化为undefined。

```js
(function() {

	console.log(typeof(x));

	x();

	function x(){console.log('#1 is Loaded.');}

	console.log(typeof(y))

	y();

	var y = function(){console.log('#2 is Loaded.');}

})();
```

最后就是这两段代码运行结果是否相同

```js
(function() {

	console.log(typeof(x));

	function x(){return;}

	var x = '';

	console.log(typeof(x))

})();

(function() {

	console.log(typeof(x));

	var x = '';

	function x(){return;}

	console.log(typeof(x))

})();
```

大概就是这个样子。

2013年1月25日补充内容: 偶得一博客，是新浪运营部的前端前辈:刘晓龙的一篇分享。

http://blog.imlxl.com/archives/category/jishu

内容如下:

> 很简单的机制，见代码 因为预解析机制的存在，v1在赋值前已经声明在闭包里了，所以alert会出现错误。

```js
(function() {
	v1 = 1;
	if(0) {
		var v1;
	}
})();

alert(v1);
```

确实看似很简单的一个例子，但是执行后会出现什么样子的错误，和为什么会出现这样的错误呢。

首先执行顺序是定义闭包()，以及使用()操作执行闭包。

接着是使用alert显示变量v1的数值。 闭包内的执行顺序是先定义全局变量v1，然后在判断0=true分支中定义闭包内的局部变量。

理论上局部不会干扰全局，打印出来的会是1. 但是犹豫预编译，定义式的变量就会被变量式的覆盖，然后定义式的变量就会失去全局的属性。

于是就会报`ReferenceError: v1 is not defined` 如果你确实想让他可以全局访问，可以考虑挂载全局变量，当然，需要注意的事项在这里：

http://soulteary.com/2013/01/25/talk-about-scope-chain.html

