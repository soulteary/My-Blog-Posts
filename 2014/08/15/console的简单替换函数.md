# console的简单替换函数

常常稍不注意就会发现IE低版本或者某个覆写了console的浏览器环境因为调用了console而功能出现了一些意外，或者收集了一堆console undefined的错误，推荐的做法不是对浏览器做shim，而是使用自建的函数去代替程序中使用的console。

对于开发过程中，我们需要设置日志输出的等级，去过滤不必要的信息，原生的console目测暂时还不支持这个功能。

还有诸如向指定服务器提交错误日志，以供后续优化和bugfix。

或者提供性能检查支持，估计还需要多添加一些其他的兼容的收集写法，以及常用的打时间戳功能。

代码很简单，没有几行，就不多说了。

```js
/**!
 * CONFIG FOR DEBUG
 * disable (0) < log (5) < debug (4) < info (3) < warn (2) < error (1)
 */

var Y = {
    "debug": {
        "api"  : "www.soulteary.com/?r.gif",
        "level": 1
    }
};

(function (w, y, z) {
    var c = w.console || null, p = w.performance || null, v = function () {}, o = new Image, d = {}, f = ['count', 'error', 'warn', 'info', 'log', 'debug', 'time', 'timeEnd'];
    for (var i = 0, j = f.length; i < j; i++) {
        (function (x, i) {d[x] = c && c[x] ? function () {y.level >= i && y.level <= 5 && c[x].apply(c, arguments)} : v})(f[i], i);
    }
    d["timeStamp"] = function(){return +new Date;};
    d["performance"] = p && p.timing ? p.timing : null;
    d["report"] = function (m) {
        if (!y) {return true;}
        var a = [];
        for (var x in m) {a.push(x + '=' + m[x]);}
        a.push('r=' + (+new Date));
        o.src = y.api + a.join('&');
    };
    w[z] = d;
}(window, Y.debug, "xdebug"));
```
