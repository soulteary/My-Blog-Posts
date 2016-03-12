# [js]准确获得页面的高度以及宽度

原文出处：http://www.cnblogs.com/huqingyu/archive/2006/11/09/446712.html

根据出处的网友评论已经修改了源代码。

```js
function getPageSize(){

var xScroll, yScroll;

if (window.innerHeight && window.scrollMaxY) {
xScroll = document.body.scrollWidth;
yScroll = window.innerHeight + window.scrollMaxY;
} else if (document.body.scrollHeight > document.body.offsetHeight){ // all but Explorer Mac
xScroll = document.body.scrollWidth;
yScroll = document.body.scrollHeight;
} else { // Explorer Mac...would also work in Explorer 6 Strict, Mozilla and Safari
xScroll = document.body.offsetWidth;
yScroll = document.body.offsetHeight;
}

var windowWidth, windowHeight;
if (self.innerHeight) { // all except Explorer
windowWidth = self.innerWidth;
windowHeight = self.innerHeight;
} else if (document.documentElement && document.documentElement.clientHeight) { // Explorer 6 Strict Mode
windowWidth = document.documentElement.clientWidth;
windowHeight = document.documentElement.clientHeight;
} else if (document.body) { // other Explorers
windowWidth = document.body.clientWidth;
windowHeight = document.body.clientHeight;
}

// for small pages with total height less then height of the viewport
if(yScroll < windowHeight){
pageHeight = yScroll;
} else {
pageHeight = windowHeight;
}

// for small pages with total width less then width of the viewport
if(xScroll < windowWidth){
pageWidth = xScroll;
} else {
pageWidth = windowWidth;
}

arrayPageSize = new Array(pageWidth,pageHeight,windowWidth,windowHeight)
return arrayPageSize;
}
```


