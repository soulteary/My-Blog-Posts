# JS - 求和趣味题

看到@drdrxp分享了一道题，和凌霄童鞋吵的不可开交，那么就挨着实现一下吧。

> ## Multiples of 3 and 5
> 
> If we list all the natural numbers below 10 that are multiples of 3 or 5, we get 3, 5, 6 and 9\. The sum of these multiples is 23. Find the sum of all the multiples of 3 or 5 below 1000.

这道题要求我们将1000以内的3和5的倍数的家伙们求和，边界是[1,999]，0的话没意义，1000的修饰语是以内。

_有一位童鞋指出可以用高斯公式去做：1000以内X的倍数之和 = X * S(1, 1000/X) 。S(1, Y)表示1到Y的所有整数之和。_

但是，我们应该是用程序来完成吧，所以就有了这篇日记，记录一下实现过程：

首先是，常规解题方法，常规解题无非笨笨的循环来搞，这里的真正的计算循环次数为1000次。注意这里不要轻易splice，你懂的，如果你实在想用，请参考后面的方法。

```js
//生成数据，这里循环是1-1000内
var data = [];
for (var i = 1, j = 1000; i < j; i++) {
    data.push(i);
}
//将合理的数据加入临时数组中
var tmp = [];
for (var i = 0, j = data.length; i < j; i++) {
    if (0 === data[i] % 5 || 0 === data[i] % 3) {
        tmp.push(data[i]);
    }
}
delete data;
//笨笨的求和
var sum = 0
for (var i = 0, j = tmp.length; i < j; i++) {
    sum += tmp[i];
}
console.log(sum) //结果是:233168
```

接下来我们用公倍数的思路来解一下，这个好处是循环次数较少，计算时的循环次数为((1000-1)/5)+((1000-1)/3)=532 次，另外需要加上两个数共同的公倍数次数（66次），一共为598次：

```js
// 生成数据
var data = [];
for (var i = 1, j = 1000; i < j; i++) {
    data.push(i);
}
// 取巧方法
var sum = 0;
for (var i = 5 - 1, j = data.length; i < j; i += 5) {
    sum += data[i];
}
for (var i = 3 - 1, j = data.length; i < j; i += 3) {
    sum += data[i];
}
for (var i = 15 - 1, j = data.length; i < j; i += 15) {
    sum -= data[i];
}
console.log(sum) //233168
```

这里如果仔细观察，我们发现其实和data中生成的东西没多少关系，所以可以化简一下：

```js
var sum = 0;
for (var i = 5, j = 1000; i < j; i += 5) {
    sum += i;
}
for (var i = 3, j = 1000; i < j; i += 3) {
    sum += i;
}
for (var i = 15, j = 1000; i < j; i += 15) {
    sum -= i;
}
console.log(sum) //233168
```

接着我们试一试这样做，先去除某个数字（应该去数量级多的家伙的，不过为了好玩，我们去掉5吧），之前的解决方法如果想用splice可以参考这里， 再然后使用转换进制以及判断尾数是否为零的方法来搞（这个方法如果把3进制变成3使用索引过滤，5使用禁止或许会更快，当然是有点多此一举的意味）：

```js
// 生成数据
var data = [];
for (var i = 1, j = 1000; i < j; i++) {
    data.push(i);
}
var sum = 0;
//我们把倍数是5的家伙从数组里干掉并求和,
//这时3和5公倍数也再见了,安心计算3的就好了
//x用于给data计算偏移
for (var i = 5, x = 5, j = 1000; i < j; i += 5, x += 5) {
    sum += i;
    data.splice((x -= 1),1);
}
//这边选择转换进制，除了二进制外的某进制，那个进制位一定是0啊
//俗一点，结尾是0
for (var i = 0, j = data.length; i < j; i ++) {
    if((data[i]).toString(3).substr(-1) ==='0'){
        sum+=data[i];
    }
}
console.log(sum) //233168
```

接着首先尝试用二进制观察一下，看看能否使用正则匹配出来，过滤掉不合适的家伙：

```js
// 生成数据
var data = [];
for(var i = 1, j= 1000;i<j ;i++){
    data.push(i);
}
// 二进制转换
var ret = [];
data.forEach(function(v,i,arr){
    if(0===v%5 || 0===v%3){
         console.log((v).toString(2))
    }
});
```

有兴趣的童鞋可以看看，输出的东西貌似不太好找规律，分组匹配的话，难度也不小，但是这样就没办法了嘛，NO，我们将之前的方法利用一下。 凌霄童鞋很在意复杂度很高的匹配，那么我们来试一试匹配一个数量级多的家伙吧，首先把999个数字中是5倍数的家伙们去掉吧，接着转换数据类型，和上面操作一样， 不同的是，我们用正则来匹配3的倍数， 下面的数据就是我们要匹配的内容，规则太简单了有木有：

```js
1,2,10,11,20,21,22,100,102,110,111,112,121,122,200,201,...,1020202,1020210,1020211,....,1100002,1100011,1100012,....00222,1101000
```

完整实现（复杂度会升高，但是如果有额外操作的话，或许这种方式也不错）：

```js
// 生成数据
var data = [];
for (var i = 1, j = 1000; i < j; i++) {
    data.push(i);
}
var sum = 0;
for (var i = 5, x = 5, j = 1000; i < j; i += 5, x += 5) {
    sum += i;
    data.splice((x -= 1), 1);
}
//存储所有的剩余的数据的三进制
for (var i = 0, j = data.length; i < j; i++) {
    data[i] = (data[i]).toString(3);
}
//把符合条件的都筛选出来,这里记得在尾巴处也加一个分隔符
data = (data.join(' ') + ' ').match(/\d+0\s/g);
//需要实现一个3进制转10进制的函数，把数字倒回去，
//或者用inArray来匹配计算
var c3to10 = function(s) {
    //这里不能直接用A^b,因为JS的问题
    var dig = function(n, p) {
        if (p === 0) {
            return 1;
        }
        var d = 1;
        for (var m = 0; m < p; m++) {
            d *= 3;
        }
        return d;
    }
    var r = 0,
        t = 0;
    for (var i = 0, j = s.length; i < j; i++) {
        t = parseInt(s[i]);
        r += t * dig(t, j - i - 1);
    }
    return r;
}
//获得266个被匹配出来的家伙
for (var i = 0, j = data.length; i < j; i++) {
    data[i] = c3to10(data[i].substring(data[i].length - 1, -1));
}
for (var i = 0, j = data.length; i < j; i++) {
    sum += data[i];
}
console.log(sum); //233168
```

最后，建议用第二种方法改进版本，除非你要算HASH等，不必使用第三种方法，当然也不是不可行，这个不是也做出来了嘛。

