# 禁止Firefox缓存input的值

原问题页面是来自[豆瓣小组](http://promiseforever.com/redirect?url=http%3A%2F%2Fwww.douban.com%2Fgroup%2Ftopic%2F24113988%2F&key=9bbfd4330d3ae0361fcceaa48719b37d)。 

<!-- more -->

给input加上**autocomplete="off"** 后firefox就不会在刷新时使用缓存值了, 如:

```
<input autocomplete="off" type="text" />
```

写js时容易被这个东西引起bug, 特别是 input的`type=hidden`时

在大牛帖子下,我提出可以添加随机数来实现缓存,其实是可以的,如果我的随机数是apache映射过的php脚本就木有关系了,如同我的css和js打包脚本.

缓存的随机数可以使用时间,想要缓存的时间久一点就使用年+月+日,或者年+月,月+日,而不想缓存的话,就设置秒就好了.
而且这么做可以缓存或者不缓存各种元素.你还可以在你的input id上做手脚,参数不要随机生成,也使用时间这个方案来搞定。
改写的形式为somepostaddress-02-21.php,映射会的实际地址为somepostaddress.php.

代码真心不上了,太小儿科了..

看大牛许久不回复..还是写一下到底是怎嘛回事吧.

