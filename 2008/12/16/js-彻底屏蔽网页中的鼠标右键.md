# [js]彻底屏蔽网页中的鼠标右键

彻底屏网页中的蔽鼠标右键。 屏蔽网页中的鼠标右键

```js
<SCRIPT LANGUAGE=JAVASCRIPT>
<!--
oncontextmenu="window.event.returnValue=false" ;
// -->
</SCRIPT>
```


屏蔽表单中的鼠标右键

```
<table border oncontextmenu=return(false)><td>no</table>
```



