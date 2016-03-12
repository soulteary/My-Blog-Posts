# [js]屏蔽鼠标右键

彻底屏蔽鼠标右键

```js
oncontextmenu="window.event.returnValue=false"
```

表格中禁止使用鼠标右键

```html
<table border oncontextmenu="return(false);"><td>PromiseForever</td></table>
```

类似代码：
屏蔽鼠标右键

```js
oncontextmenu="return false"
```

禁止选取内容

```js
onselectstart="return false"
```

禁止粘贴内容

```js
onpaste="return false"
```

禁止复制 禁止剪切

```js
oncopy="return false;" oncut="return false;"
```

接下来来个全的

```html
<body oncontextmenu="return false" ondragstart="return false" onselectstart="return false" onselect="document.selection.empty()" oncopy="document.selection.empty()" onbeforecopy="document.selection.empty()" onmouseup="document.selection.empty()>
<noscript><iframe scr=*></iframe></noscript>
```


