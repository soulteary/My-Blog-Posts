# javascript 浮点数的精确计算

刘磊童鞋遇到的一个问题，大概样子是这样：

两个浮点数在JS里计算，因为JS这类的解释语言使用的是IEEE754，所以计算浮点小数会有各种奇葩问题。

所以就写了一个小函数。

话说，网上的答案怎么都长一个样子，伤不起- -！

## 解决方案

```js
var calc = function(num1,operator,num2,len){
	var numFixs = function(num){
		var arr = num.toFixed(len).toString().split('.');
		return parseInt(arr.join(''));
	}
	switch(operator){
		case '+':
			return ( numFixs(num1) + numFixs(num2) )/ Math.pow(10,len);
		break;
		case '-':
			return ( numFixs(num1) - numFixs(num2) )/ Math.pow(10,len);
		break;
		case '*':
			var tmp = ( numFixs(num1) * numFixs(num2) )/ Math.pow(10,len) / Math.pow(10,len);
			return parseFloat(tmp.toFixed(len));
		break;
		case '/':
		if (num2 == 0) {return 'Error'}
			return ( numFixs(num1) / numFixs(num2) )/ Math.pow(10,len);
		break;
	}
}

//下面是例子
calc(2.01,'-',2.11 ,2);//两个两位小数
calc(2.1,'-',2.111111 ,2);//不同位数的小数
calc(1,'-',2.111111 ,2);//整数和小数
calc(5,'-',2.111111 ,3);//整数和小数
```

