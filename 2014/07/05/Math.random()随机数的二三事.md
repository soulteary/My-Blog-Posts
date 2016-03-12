# Math.random()随机数的二三事


看到题目，如果大家平时被问到：

> 如何生成一个怎么样怎么样的整数随机数。

估计大家都会不屑，但是当你淡定的回答获取一个范围应该是随机数seeds和区间数值差的乘机与最小数相加然后再怎么怎么的时候...有没有发现你的思维已经固化了呢。

这个知识点应该是玩JS肯定会碰到的之一吧。文末有Markdown，可以直接下载阅读，清爽一点。

先来掉书袋，看看[MDN的文档](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math/random)。

打开Node，进入终端命令行模式，输入`Math.random()`:

```js
> Math.random()
0.436846193857491
```

结果是不是依旧如同往常一样稀松平常的小于1的一个伪随机数跳了出来呢。 这个时候，如果别人问你，还有什么其他方案可以生成随机数么，你会想到神马呢。

_逝者如斯夫，不舍昼夜。_

如果你继续在终端里输入new Date()-0：

```js
> new Date()-0
1404488829907
```

我想你可以得到一个自增的数字，对，就是“秒”，如果你说这货哪里随机了，请别着急：

```js
> (new Date()-0)%10086
8657
```

这里的取模%的数值可以是大于2且最好小于当前时间的数值，则可以得到你取模数值概率分之一的概率的随机数。

如果你取模的数值是随机数呢，那么产生这个随机数的可见的两个变量都是随机的，那么是不是近似真的“随机数”了呢？

当然，如果使用这招，还要考虑到硬件以及语言执行过程的耗时，因为我们知道计算机执行的时候，有一个时间的精度的范畴，所以需要使用一点点的延时抑制。

扯了一些没用的，你可能着急了，那么请保持好奇心，我们继续说点无聊的事情。

Math.random会提供给我们一个[0,1)之间的随机数，但是如果我们要[1,10]范围随机整数的话，可以使用以下三个函数：

*   Math.round
*   Math.ceil
*   Math.floor

我们先来生成一个随机数：

```js
> Math.random()*(10-1)+1
8.26644050120376
```

接着我们来使用这三个Math内建函数：

```js
> Math.round(8.26644050120376)
8

> Math.ceil(8.26644050120376)
9

> Math.floor(8.26644050120376)
8
```

把数值换成8.56644050120376后，再来看看：

```js
> Math.round(8.56644050120376)
9

> Math.ceil(8.56644050120376)
9

> Math.floor(8.56644050120376)
8
```

所以区别一目了然，对于浮点数，round会遵守四舍五入规则，ceil无论如何贪心进位+1，floor无论如何都小心翼翼的自断一臂-1，至于整数，自己试试看咯。

说到这里，接下来可以正常的描述内容了：

问：如何快速生成一段随机文本，比如验证码或者我们访问网站常见的随机数token。

答案很多，我说一个经典的，其实思路很简单，把刚刚生成随机数的方法随便选择一个.toString()：

```js
Math.random().toString(36).substring(7);
```

//当然也可以写成这样

```js
Math.random().toString(36).slice(2);
```

//或者利用时间

```js
(new Date()-0).toString(36)
```

随便输出一些，我们可以看到这货输出的字符串长短参次不齐的：

```text
mptzulnb3xr

87jx7vkuik9

761qsolayvi

amqx2mx6r

ce5uyvkuik9

5ioufim5cdi

dirp4hiwwmi

ioe597ldi

ohn9izfr

sprsakk2o6r

5g3ruo6flxr

...

//单纯时间来做随机是不是生成的惨不忍睹

//而且不做随机延时抑制，重复太明显

hx7pom3y

hx7pom3z

hx7pom40

hx7pom41

hx7pom42

hx7pom43

hx7pom44

hx7pom45
```

我们先来看看为什么用toString()可以生成随机数： 首先前面的家伙不管是随机数seeds生成的，还是时间递增的长整型，它们都是Number构造器构造出来的[object Number]，而在ecma.js中，Number的这个方法是这样的：

```js
/**

@param {Number} [radix]

@return {string}

*/

Number.prototype.toString = function(radix) {};
```

作为一个好人，我给你指条明路，[MDN的文档](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number/toString)

看过之后是不是想到了parseInt函数的第二个参数？我们发现，这个原生函数支持2~36（如果超出36，那么26个字母就不够用了亲）进制的转换，所以如果在生成的时候，随机切换进制，取结果的随机位置，效果会不会更好呢，你可以试一试。

如果你想获得字母多一点且平均一点，那么只有使用36进制了，但是不管怎么躲，都有可能出现一串数字。

```text
xjuk0zxofen1xlxr
```

有没有好方法来解决这个问题呢，答:

```js
Math.random().toString(36).replace(/[^a-z]+/g, '')

//或者这样

Math.random().toString(36).slice(2).replace(/\d/g, '')
```

这个时候，你或许会说，文章该就此结束了吧，这个方法看起来很爽很简洁。 不过，你有思考过一个问题么，回顾前文，随机数可以是0,1,...这些整数...

当随机数是这些数值的时候，很抱歉，返回值是原来的数值，即0,1,...，我们得到的最后的结果就会是一个空字符串，而如果是0.5这类某些以5结尾的浮点数的时候，结果依然如此。还有当数值是某些时候，生成的随机数位数会比较短...

解决这个问题，你当然可以重新生成这个随机数，直到它输出一个你心满意足的随机数再放过他，但是，我们刚刚了解到的生成一个大的随机数的方法是依赖时间，小学还是初中学过的用一个比较小的数值除以一个比较大的数值，得到的结果是一个更小的浮点数来解决这个问题呢。

```js
(Math.random() / +new Date()).toString(36).replace(/\d/g, '').slice(1)
```

这样就得到了一个比较长，且比较公平的随机数。可以用node验证一下：

```js
var c = {}, r;for (var i = 0, j = 10000000; i < j; i++) {r = (Math.random() / +new Date()).toString(36).replace(/\d/g, '').slice(1);c[r] ? c[r] += 1 : c[r] = 1;}for (var i in c) {if (c[i] === 1) {delete c[i];}}console.log(c);
```

运行结果是没有任何冲突，当然这可能是小概率事件。如果你觉得你点很背，你可以试一试下面这段，手动执行几次，看看，有没有不是空数组这个结果的结果。

```js
for(var i = 0,j=100;i<j;i++){(function(){var c = {}, r;for (var i = 0, j = 1000; i < j; i++) {r = (Math.random() / +new Date()).toString(36).replace(/\d/g, '').slice(1);c[r] ? c[r] += 1 : c[r] = 1;}for (var i in c) {if (c[i] === 1) {delete c[i];}}console.log(c);}())}
```

当然，你也可以不用这两个测试例子上面的那段代码，改用下面这种方式，多输出几次随机字符串，然后拼合在一起。

```js
for(var c = ''; c.length < 5;) c += Math.random().toString(36).substr(2)
```

这个把戏估计你看腻了，我们来看下面这个系列的示例，从固定的字典中抽取字符构成随机字符串：

依赖Array的map方法，要注意兼容性，当然，你从MDN那边copy一段hacks也可以无缝兼容，是不是看起来高大上一点：

```js
Array.apply(0, Array(5)).map(function () {

    return (function (charset) {

        return charset.charAt(Math.floor(Math.random() * charset.length))

    }('ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789'));

}).join('');
```

不过或许你更容易接受这种多一点：

```js
function rand(length, current) {

    current = current ? current : '';

    return length ? rand(--length, "0123456789ABCDEFGHIJKLMNOPQRSTUVWXTZabcdefghiklmnopqrstuvwxyz".charAt(Math.floor(Math.random() * 60)) + current) : current;

}
```

或者更传统的：

```js
function rand() {

    var text = "";

    var possible = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789";

    for (var i = 0; i < 5; i++) text += possible.charAt(Math.floor(Math.random() * possible.length));

    return text;

}
```

或许你以为这样把戏就玩完了，too young too simple，之前玩过了Number的原生函数，我们还可以动手脚的是String的原生函数，比如我们知道，字母的ASCII码是[65,97]，那么当这个区间是字典，我们随机抽取这个区间是不是会有一些好玩的事情发生呢。

```js
function rand(x) {

    var s = "";

    while (s.length < x && x > 0) {

        var r = Math.random();

        s += String.fromCharCode(Math.floor(r * 26) + (r > 0.5 ? 97 : 65));

    }

    return s;

}
```

随便输入一个电话号码（推荐输入短一点的数），然后等等看，是不是会出现下面的结果：

```text
JrxKnxrHwJzGCKoKCzrpFxMxDBFqBrpAEGvpvMtCIHIHFCxtMxrHtpEGDyxzxwMBBqDvEBwxprwHqDMCErIzLwuyFApnxpxJoxBAFEoHsAEqvIxBIJvpFBoLnvurHJsvDFwGtFvDsELMLwzowvqBJtCwrGCsCHGvsxCLunGxtrtnyvvwyFqEsstotxnsrqLHAIyCxzLxDqtuzsoFJAGHyxrwxJusJrtpuvIyIILoJrGLKnptqHLBAKwGEpnIwzCtFAnrHIqLHynGwuyupsDpLJFGxBuJBouwCDGKsKFCLKnAzoupsxFqIynKFCoBApyBsJKzJpKwEFCyGywrBoFpvorMzBrBAFowrKxvuJLoKtzpEoDsEsBExyGBMssnADnBwrvJDGunJsMyFGCxIFApEnoLyGxBrHroBsLAICGLDIwvqp
```

这个话题，好像说的有点麻木了，换个需求吧。有的网站会玩一些随机背景色的小花样。有没有优雅的解决方案呢：

遵守随机数函数的定义，获取颜色数值之间的数值就好，看过了上面的代码，这句，很好懂了伐：

```js
'#'+(0x1000000+(Math.random())*0xffffff).toString(16).substr(1,6)
```

当然，你可以写一个更加直观的方案：

```js
//用十进制数据来代替16进制数据运算，请注意因为右边是开区间，要比上面的0xffffff +1，最后将结果转换回十六进制，然后修剪一下字符串，输出

Math.floor(Math.random() * 16777216).toString(16);

'#000000'.slice(0, -color.length) + color;

//或者更简短的样子如下

"#" + ("000000" + Math.floor(Math.random() * 16777216).toString(16)).substr(-6);
```

还有一类花样，如下：

```js
function randomColor() {

    var r = function () { return Math.floor(Math.random()*256) };

    return "rgb(" + r() + "," + r() + "," + r() + ")";

}
```

输出如下:

```js
rgb(29,236,191)
```

时间不早了，最后说一下随机排序数组吧。

关于洗牌算法，网上流传很多，随便选择一种模拟一下就好，比如随便写的全重排：

```js
var i = 0, data = [], r;

for (; i < 10; data[i++] = i);

while (--i) {

    r = Math.round(Math.random() * 9 + 1) - 1;

    data[i] = data[i] + data[r], data[r] = data[i] - data[r], data[i] = data[i] - data[r];

}

console.log(data)
```

或者利用Array.prototype.sort()函数，这里可以不把里面的数值带进来运算。

首先Math.random()会生成一个[0,1)之间的数值，用0.5这个比较公平的数值减去它，概率得到小于0，等于0,大于0三种状况，而Array.prototype.sort()期待的数值恰好是[-1,0,1]，是不是很省事。

```js
var i = 0, data = [], r;

for (; i < 10; data[i++] = i);

data.sort(function () {

    return .5 - Math.random();

});
```

时间不早了，明天还得早起，先写到这里，想到什么，再补充什么吧。

多谢浏览，欢迎反馈。

资料参考: Google && StackOverflow

晓白于2014.7.5夜

-EOF-

[js-math.random.story](https://attachment.soulteary.com/2014/07/js-math.random.story_.zip)

