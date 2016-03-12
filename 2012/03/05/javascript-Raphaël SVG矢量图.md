# [javascript]Raphaël SVG矢量图

原文出处:http://raphaeljs.com/

使用方法很简单,只需要把js库引用到你的HTML文件中即可.

```js
// Creates canvas 320 × 200 at 10, 50
var paper = Raphael(10, 50, 320, 200);

// Creates circle at x = 50, y = 40, with radius 10
var circle = paper.circle(50, 40, 10);
// Sets the fill attribute of the circle to red (#f00)
circle.attr("fill", "#f00");

// Sets the stroke attribute of the circle to white
circle.attr("stroke", "#fff");
```

demo在官方上有不少,都很精彩,值得学习.

