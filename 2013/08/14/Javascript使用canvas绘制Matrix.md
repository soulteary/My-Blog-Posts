# Javascript使用canvas绘制Matrix

[![2013-08-14_023153](https://attachment.soulteary.com/2013/08/14/2013-08-14_023153.png "2013-08-14_023153")](https://attachment.soulteary.com/2013/08/14/2013-08-14_023153.png) 

在cssdeck看到了一段简洁有趣的效果，不敢私藏。

## 正文

> 出处：http://cssdeck.com/labs/the-matrix

> 预览地址:http://thecdn.sinaapp.com/page/demo/matrix-canvas/

首先是创建一个画布，比如使用下面简单的结构。

```html
<canvas id="q"></canvas>
```

脚本代码

```js
var s = window.screen;
var width = q.width = s.width;
var height = q.height = s.height;
var letters = Array(256).join(1).split('');

var draw = function () {
  q.getContext('2d').fillStyle='rgba(0,0,0,.05)';
  q.getContext('2d').fillRect(0,0,width,height);
  q.getContext('2d').fillStyle='#0F0';
  letters.map(function(y_pos, index){
    text = String.fromCharCode(3e4+Math.random()*33);
    x_pos = index * 10;
    q.getContext('2d').fillText(text, x_pos, y_pos);
    letters[index] = (y_pos > 758 + Math.random() * 1e4) ? 0 : y_pos + 10;
  });
};
setInterval(draw, 33);
```


