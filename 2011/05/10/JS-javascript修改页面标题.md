# [JS]javascript修改页面标题

或许这个灰常多的人都会吧.而且也是一个比较老的技巧.. 比如很早的时候就收录过.>>[浏览](http://promiseforever.com/2008/01/21/js-code-622.html)<< 

但是,当习惯性的写下window.title='';后,发现并木有任何改变的时候...

突然就意识到了,知识需要更新了. 所以.window.title,很自然的被新的模型中的document.title取代.并且被各种浏览器支持解析. 

附送一个跑马灯标题栏改变特效.

```js
<script language="JavaScript">
var msg="欢迎光临 苏洋博客！   ";
var interval=150
var spacelen=120;
var space10=" ";
var seq=0;
function Scroll(){len=msg.length;document.title=msg.substring(0,seq+1);seq++;if(seq>=len){seq=0;document.title='';window.setTimeout("Scroll();",interval);}
else
window.setTimeout("Scroll();",interval);}
Scroll();</script>
```

