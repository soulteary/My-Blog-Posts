# Javascript Tips

via http://www.cnblogs.com/chenguangyin/archive/2012/12/10/2812122.html 代码略有改动。

正确的代码说明一切问题。

## 代码时间

转换字符串为数字。

```js
var str = '3.14';

//强制转换
Number(str);
//隐式转换
+str;
//故意与数字计算
str - 0;

//使用函数
parseInt(str);		//取整
parseFloat(str);	//转换浮点
```


关于各种数据隐式转换Boolean，给一个测试例子吧。

```js
//测试数据
var testData = [true, false, 'soulteary', '', 123, Infinity, 0, NaN, {}, null, undefined];

for(var oo in testData){
		console.log(!!testData[oo],typeof testData[oo], testData[oo]);
}

//或者直接强制转换
Boolean("你的变量");
//双否运算直接强制转换你懂的
!!testData
```


创建多维数组

```js
//创建定长的数组
var arr = new Array(2);
	arr[0] = new Array(2);
	arr[1] = new Array(2);
	arr[0][0] = 0.0;
	arr[0][1] = 0.1;
	arr[1][0] = 1.0;
	arr[1][1] = 1.1;

//创建不定长的数组
var arr = new Array();
	arr[0] = new Array();
	arr[1] = new Array();
	arr[0][0] = 0.0;
	arr[0][1] = 0.1;
	arr[1][0] = 1.0;
	arr[1][1] = 1.1;

//创建索引不连续的
//PS.不连续的数组大小算你的最后索引加一
var arr = new Array();
	arr[0] = new Array();
	arr[1] = new Array();
	arr[0][0] = 0.0;
	arr[0][6] = 0.1;	//arr[0].length 为 7
	arr[1][0] = 1.0;
	arr[1][1] = 1.1;

//简写
var arr =[[],[]];
	arr[0][0] = 0.0;
	arr[0][1] = 0.1;
	arr[1][0] = 1.0;
	arr[1][1] = 1.1;

//再简写
var arr =[[0.0, 0.1],[1.0, 1.1]];
```

防止被iframe

```js
//防止被iframe
if(top !== window) {
	top.location.href = window.location.href;
}
```


正则对象结果转数组的思路

```js
var testStr = '12a3@4!d5ff6gg7hh890';
var rule = /\d/g;
var regexp = testStr.match(rule, testStr);

var arr = [];

for(var i = 0, len = regexp.length; i < len; i++) {
	arr.push(regexp[i]);
}

console.log(arr);
```

第二种思路

```js
var testStr = '12a3@4!d5ff6gg7hh890';
var rule = /\d/g;
var arr = [];
testStr.replace(rule, function(){
	arr.push(arguments[0]);
})

console.log(arr);
```

第三种思路

```js
var testStr = '12a3@4!d5ff6gg7hh890';
var rule = /\D/g;
var arr = [];
var delimiter = '::';
arr = testStr.replace(rule, delimiter).split(delimiter);
console.log(arr);
```

第四种思路

```js
var testStr = '12a3@4!d5ff6gg7hh890';
var rule = /(\d)/g;
var arr = [];
while(rule.test(testStr)){
	arr.push(RegExp.$1);
}

console.log(arr);
```

* * *

数据寻找最大值(直接比较)

```js
var arr = [1, 5, 4, 12355, 43, 123, 123, 3, 4454, 43];
var max = arr[0];

for(var i in arr) {
	if(arr[i] > max) {
		max = arr[i];
	}
}

console.log(max);

```

内置Math函数方法

```js
var arr = [1, 5, 4, 12355, 43, 123, 123, 3, 4454, 43];
var max = Math.max.apply(this, arr);

console.log(max);
```

* * *

数字取整

```js
var data = 3.14;

var theMethod = [];
//常规方法
theMethod.push( Math.floor(data) );
//常规方法
theMethod.push( parseInt(data) );
//常规方法
theMethod.push( data.toFixed(0) );

//只可用于不小于0的数
theMethod.push( data>>>0 );
//正负数都可用
theMethod.push( ~~data );

console.log(theMethod)
```


避免精度问题 如:0.1+0.2 != 0.3

```js
var a = 0.1;
var b = 0.2;
var c = 0.3;

console.log( a+b == c );

console.log( (a+b).toFixed(1) == c );
```

整数前补0 一般方法:

```js
var addZero = function(num, n) {
	var len = num.toString().length;
	while(len < n) {
		num = "0" + num;
		len++;
	}
	return num;
}
console.log(addZero(5, 8));//输出 00000005
```

简洁方法:

```js
var addZero = function(num, n) {
	y = '00000000000000000000000000000' + num;
	return y.substr(y.length - n);
}
console.log(addZero(5, 8));//输出 00000005
```

动态添加:

```js
var addZero = function(num, n) {
	return new Array(Math.max(0,n + 1 -num.toString().length)).join("0") + num;
}
console.log(addZero(5, 8));//输出 00000005
```

