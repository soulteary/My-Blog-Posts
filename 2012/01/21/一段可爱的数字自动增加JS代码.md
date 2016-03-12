# 一段可爱的数字自动增加JS代码

或许你还记得google邮箱不停增加的MB数量吧。

下面的代码可以实现类似功能，实现很简单（或许应该去掉abs重写一下，只有增长没有减少的数值感觉有点假了。）

代码结构如下：

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>Number</title>
</head>
<body>
<h3>Site Stats</h3>
<ul class="s-stats">
  <li>
    <div class="name">Members Logged in:</div>
    <div class="value"><span id="dispnum2">8,395,455</span></div>
  </li>
  <li class="line">
    <div class="name">Files Shared:</div>
    <div class="value"><span id="dispnum3">53,452,962</span></div>
  </li>
  <li>
    <div class="name">Average DL Speed:</div>
    <div class="value"><span id="dispnum1">3,942</span> KB/sec</div>
  </li>
  <li class="line">
    <div class="name">Movies Online:</div>
    <div class="value"><span id="dispnum">3,567,066</span></div>
  </li>
  <li>
    <div class="name">TV Shows Online:</div>
    <div class="value"><span id="dispnum4">11,498,445</span></div>
  </li>
  <li class="line">
    <div class="name">Music Online:</div>
    <div class="value"><span id="dispnum5">34,346,786</span></div>
  </li>
  <li class="total">
    <div class="name">Software Online:</div>
    <div class="value"><span id="dispnum6">8,344,618</span></div>
  </li>
</ul>
</body>
</html>
```

JS脚本如下：

```js
function formatNumber(number, decimals, dec_point, thousands_sep ) {
 var n = number, c = isNaN(decimals = Math.abs(decimals)) ? 2 : decimals;
 var d = dec_point == undefined ? "," : dec_point;
 var t = thousands_sep == undefined ? "." : thousands_sep, s = n < 0 ? "-" : "";
 var i = parseInt(n = Math.abs(+n || 0).toFixed(c)) + "", j = (j = i.length) > 3 ? j % 3 : 0;
 return s + (j ? i.substr(0, j) + t : "") + i.substr(j).replace(/(\d{3})(?=\d)/g, "$1" + t) + (c ? d + Math.abs(n - i).toFixed(c).slice(2) : "");
}

var ttnum=3567066;var tt;function dis_num() { document.getElementById ("dispnum").innerHTML=formatNumber(ttnum, 0, '.', ','); ttnum	= ttnum+2; tt = setTimeout ("dis_num()",800);	} dis_num();
var ggnum=3942;var gg;function dis_num1() { document.getElementById ("dispnum1").innerHTML=formatNumber(ggnum, 0, '.', ','); ggnum	= ggnum+2; gg = setTimeout ("dis_num1()",1100); } dis_num1();
var hhnum=8395455;var hh;function dis_num2() { document.getElementById ("dispnum2").innerHTML=formatNumber(hhnum, 0, '.', ','); hhnum	= hhnum+1; hh = setTimeout ("dis_num2()",2200);	} dis_num2();
var iinum=53452962;var ii;function dis_num3() { document.getElementById ("dispnum3").innerHTML=formatNumber(iinum, 0, '.', ','); iinum	= iinum+8; ii = setTimeout ("dis_num3()",1000);	} dis_num3();
var jjnum=11498445;var hh;function dis_num4() { document.getElementById ("dispnum4").innerHTML=formatNumber(jjnum, 0, '.', ','); jjnum	= jjnum+5; jj = setTimeout ("dis_num4()",7000);	} dis_num4();
var kknum=34346786;var kk;function dis_num5() { document.getElementById ("dispnum5").innerHTML=formatNumber(kknum, 0, '.', ','); kknum	= kknum+1; kk = setTimeout ("dis_num5()",1600);	} dis_num5();
var llnum=8344618;var ll;function dis_num6() { document.getElementById ("dispnum6").innerHTML=formatNumber(llnum, 0, '.', ','); llnum	= llnum+1; ll = setTimeout ("dis_num6()",800);	} dis_num6();

```


