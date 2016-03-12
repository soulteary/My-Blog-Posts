# 禁止鼠标右键JS代码

## 方案一

禁止鼠标左右键代码/禁止网页选中/禁止另存为/防复制代码

```html
<noscript><iframe src="/*.html>";</iframe></noscript>
<script>
function stop(){
	return false;
}
document.oncontextmenu=stop;
</script>
```

## 方案二

禁止鼠标左右键

```js
<script language="javascript"><!--
if (window.Event)
document.captureEvents(Event.MOUSEUP);
function nocontextmenu(){
event.cancelBubble = true
event.returnValue = false;
return false;
}
function norightclick(e){
if (window.Event){
if (e.which == 2 || e.which == 3)
return false;
}
else
if (event.button == 2 || event.button == 3){
event.cancelBubble = true
event.returnValue = false;
return false;
}
}
document.oncontextmenu = nocontextmenu; // for IE5+
document.onmousedown = norightclick; // for all others
//--></script>
```

## 方案三

禁止选中代码

```js
<script language="JavaScript">document.oncontextmenu=new Function("event.returnValue=false;");
document.onselectstart=new Function("event.returnValue=false;");</script> 
```

## 方案四


禁止另存为

```html
<noscript>
<iframe src="/*.htm"></iframe>
</noscript>
```

## 方案五

防拷贝/复制代码6、禁止选择文本

```js
<script type="text/javascript">var omitformtags=["input", "textarea", "select"]
omitformtagsomitformtags=omitformtags.join("|")
function disableselect(e){
if (omitformtags.indexOf(e.target.tagName.toLowerCase())==-1)
return false
}
function reEnable(){
return true
}
if (typeof document.onselectstart!="undefined")
document.onselectstart=new Function ("return false")
else{
document.onmousedown=disableselect
document.onmouseup=reEnable
}</script> 
```

## 方案六

禁止网页另存为

```js
<noscript><iframe src="/*.html>";</iframe></noscript>
```

## 方案七

禁止选择文本：

```js
<script type="text/javascript">var omitformtags=["input", "textarea", "select"]

omitformtagsomitformtags=omitformtags.join("|")

function disableselect(e){
if (omitformtags.indexOf(e.target.tagName.toLowerCase())==-1)
return false
}

function reEnable(){
return true
}

if (typeof document.onselectstart!="undefined")
document.onselectstart=new Function ("return false")
else{
document.onmousedown=disableselect
document.onmouseup=reEnable
}</script> 
```

## 方案八

禁用右键:

```js
 <script>function stop(){
return false;
}
document.oncontextmenu=stop;</script> 
```

## 方案九

真正的鼠标右键屏蔽

```js
<script language="JavaScript"><!--

if (window.Event)
 document.captureEvents(Event.MOUSEUP);

function nocontextmenu()
{
event.cancelBubble = true
event.returnValue = false;

return false;
}

function norightclick(e)
{
if (window.Event)
{
 if (e.which == 2 || e.which == 3)
 return false;
}
else
 if (event.button == 2 || event.button == 3)
 {
 event.cancelBubble = true
 event.returnValue = false;
 return false;
 }

}

document.oncontextmenu = nocontextmenu; // for IE5+
document.onmousedown = norightclick; // for all others
//--></script> 
```


