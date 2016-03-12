# [JS]javascript页面载入特效

某些时候,不需要使用AJAX库的时候,或许一个很老的例子可以帮我们轻巧的解决我们需要的效果. 

javascript代码部分:

```js
<script type="text/javascript">
var t_id=setInterval(animate,20);var pos=0;var dir=2;var len=0;function animate(){var elem=document.getElementById('progress');if(elem!=null){if(pos==0)len+=dir;if(len>32||pos>79)pos+=dir;if(pos>79)len-=dir;if(pos>79&&len==0)pos=0;elem.style.left=pos;elem.style.width=len;}}
function remove_loading(){this.clearInterval(t_id);var targelem=document.getElementById('loader_container');targelem.style.display='none';targelem.style.visibility='hidden';}
</script>
```

css代码部分:

```css
#loader_container{text-align:center;position:absolute;top:40%;width:100%;left:0;padding:10px;}
#loader{font-family:Tahoma, Helvetica, sans;font-size:11.5px;color:#7c7c7c;;background-color:#c0c0c0;padding:10px 0 16px 0;margin:0 auto;display:block;width:130px;border:1px solid #5a667b;text-align:left;z-index:2;}
#progress{height:5px;font-size:1px;width:1px;position:relative;top:1px;left:0px;background-color:#8894a8}
#loader_bg{background-color:#e4e7eb;position:relative;top:0px;left:0px;height:7px;width:113px;font-size:1px}
```

html代码部分:

```html
<html>
<body onload="remove_loading();">
<div id="loader_container">
<div id="loader">
<div align="center">页面正在加载中 ...</div>
<div id="loader_bg"><div id="progress"> </div></div></div>
</div>
</body>
</html>
```

使用方便很简单,就不多说了.

