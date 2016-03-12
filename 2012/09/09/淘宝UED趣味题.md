# 淘宝UED趣味题

博客的新主题还是没写完，看官先将就看吧。 睡起来看到 @明杰 同学发来的淘宝UED趣味题,之前看到 @吴钊 大牛发了,但是是在手机上看到的,木有办法做-，-.. 于是，试一试。

题目地址: http://ued.taobao.com/quiz/ 第一题很简单，不管是用F12看源码，还是右键看源码一目了然，但是呢，前端的话，文艺一点吧。 比如这样

```
document.location = document.getElementById('wrapper').getElementsByTagName('p')[0].innerText;
```

如果你不习惯使用F12的console的话，那么在地址栏输入:

```
javascript:document.location = document.getElementById('wrapper').getElementsByTagName('p')[0].innerText;
```

来到第二题的地址了吧. 页面中的字符串是下一题的答案，一眼就看出来需要在console中跑了吧，但是我要再文艺一点的说。

```
a="hostname,test,value,input,getElementsByTagName,nextQuiz,23805,http,protocol,location,reverse,join,split,w2YCcbRUXqx2COflFW6RWWkfXWO?/ziuq/moc.oaboat.deu//:ptth,GET,...".split(",");this[a[5]]=a[13][a[12]]("")[a[10]]()[a[11]]("");0;
```

还是老规矩console的代码

```
var fir = document.getElementsByTagName('textarea')[0].textContent.split(';');eval(fir[0]);fir=eval(fir[1]);
document.location =fir;
```

当然，也还是有地址栏的版本-，-

```
javascript:var fir = document.getElementsByTagName('textarea')[0].textContent.split(';');eval(fir[0]);fir=eval(fir[1]);document.location =fir;
```

接着是下一题了，这个题很有意思，你可以选择普通青年的margin-left + margin-top移动位置，然后等他跳转。 但是我想更文艺一点。 命令行使用:

```
document.location =document.getElementById('test').value.split('').reverse().join('').substr(7);
```

浏览器使用:

```
javascript:document.location =document.getElementById('test').value.split('').reverse().join('').substr(7);
```

最后一题了，继续文艺嘛 好吧，文艺一下，大概能猜到对面apache上脚本的算法了。 应该是

```
//PHP 伪代码
if(!$_REQUEST['wl']){show_error();}
//这个判断居然木有判断是否强制转为数字..至少我测试的时候,soulteary=soulteary也可以的说
if($_REQUEST['idx'] != $_REQUEST['r_idx']){show_error();};
//展示链接
echo 'http://ued.taobao.com/quiz/?PTaDwGiTs2JlfOC2xqTRRbQPYzK6FG4';
//r_weight这个参数是不是酱油-，-...目测去掉也行,不是取的时候忘记转类型,判断直接酱油了吧..
```

那么,开始自动奔向答案吧! console版本:

```
Libra.prototype.doSubmit =function () {$.ajax({type:"POST",url:"./?action=finish",data:"idx=5&r_idx=5&wl=1&r_weight=11",success:function(msg){document.location=msg;}});}
var libra = new Libra($("#libra"));libra.doSubmit();
```

附上地址栏版本:

```
javascript:Libra.prototype.doSubmit=function(){$.ajax({type:"POST",url:"./?action=finish",data:"idx=5&r_idx=5&wl=1&r_weight=11",success:function(msg){document.location=msg}})};var libra = new Libra($("#libra"));libra.doSubmit();
```

好了，@明杰童鞋，我的文艺青年路线走完了。可惜我木有校园邮箱，吉利大学木有邮箱福利。

对了，淘宝UED的大大如果有富裕的书的话，能不吝送我一本，我也会很开心的说~

