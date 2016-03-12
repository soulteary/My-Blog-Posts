# 一个不错的KEYVALUE存储设计思路

做项目的时候看到的一个设计，没有深究后端实现，有空验证一下，粗浅的想了一下，这样设计中间的缓存极大扩展了储存空间，防止了KEY冲突，对前端也比较友好。

即使接RDBMS，也比较容易拆分，赞。

看到的越多，就越感觉自己知道的太少，学到的越多，就越对技术产生渴望，或许这也是一种贪婪吧。

```js
var key = ';12304035:48072;122216431:27023;1627207:28341;5919063:6536025;';

var keyArr = key.split(';').slice(1, -1);
var tmpArr = null,
    order = [],
    value = [],
    ret = [''],
    index = 0;

for (var i = 0, j = keyArr.length; i < j; i++) {
    tmpArr = keyArr[i].split(':');
    order[i] = tmpArr[0];
    value[tmpArr[0]] = tmpArr[1];
}

order = order.sort(function(a, b) {
    return a - b
});

for (i = 0, j = order.length; i < j; i++) {
    index = keyArr.indexOf(order[i] + ':' + value[order[i]]);
    ret.push(keyArr[index]);
}

ret.push('');
ret = ret.join(';');

console.log(ret);
```
