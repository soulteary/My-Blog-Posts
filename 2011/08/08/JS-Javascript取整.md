# [JS]Javascript取整

int这个函数基本是通用的取整，今天无意百度，发现了其他的几种方法。

```js
<script language=javascript>
var t=3.1415;
alert( "int("+t+") = "+int(t) );
alert( "parseInt("+t+") = "+parseInt(t) );
alert( "Math.floor("+t+") = " +Math.floor(t) );
alert( "Math.round("+t+") = "+Math.round(t) );
alert( "Math.ceil("+t+") = " +Math.ceil(t) );
function int(num){return num-num%1}
</script>
```


