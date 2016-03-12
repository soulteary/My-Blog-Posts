# Javascript标题滚动

闲着无趣，发一个小脚本，有想把以前的VBSCRIPT全部改写过来的冲动。 有的时候我们需要窗口标题闪动或者滚动，那么不妨使用下面的代码。

```js
(function () {
    var titleScroll = function () {
        var index = 0;
        var text = "欢迎光临Story校园相册！";
        var len = text.length;
        var init = function () {
            document.title = text.substring(0, index + 1);
            index++;
            if (index >= len) {
                index = 0;
                document.title = text;
            }
            setTimeout(arguments.callee, 150);
        }
        init();
    }
    titleScroll();
})("http://soulteary.com")
```

当然有个压缩版本的，使用方法嘛，在最后的括号传递的变量里改为你的提示文本就好。

```js
(function(b){var a=0,c=b.length;(function(){document.title=b.substring(0,a+1);a++;a>=c&&(a=0,document.title=b);setTimeout(arguments.callee,150)})()})("http://soulteary.com");
```
