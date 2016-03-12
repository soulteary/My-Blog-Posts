# 关于CData

CData 全称character data(char发音是'K...我之前一直发错,囧..)

顾名思义,符号数据,也就是说,在这里,所有的数据都会被忠实的执行,而不会被转义处理等. 

如果你常常看或者写内嵌于页面的js的话，你会发现，有的时候，js脚本会出现那么如何正确使用呢. 

<!-- more -->

```html
<![CDATA[... YOUR CODE HERE! ...]]>
```

首先，我们要了解这个东西为神马会出现，它的出现是为了解决xhtml解释器自作多情的转义js中的某些符号，比如"<"。

```html
<script type="text/javascript">
if (i<0){ do something you like.}
</script>
```

这样的话，是不会出错的。 

但是如果你为了美观，比如我，常常手贱打几个空格在运算符之间。(当然是能打空格的单个操作符~)

```html
<script type="text/javascript">
if (i < 0){ do something you like.}
</script>
```

不出意外的话，你的脚本会挂，也就是说你之后有写神马的话，功能直接死掉，不会继续执行了。 

原因是什么呢。 "<"在xhtml中是作为新元素开始标签使用的，而且标签的话，是不允许在"<"和"标签名"之间添加空格滴。 你见过"< a"嘛？如果见过，那么是某个大意的家伙打错了。。。 

问题找到，解决方法呢，文艺青年决定使用转义的方法，避开这个问题。

```html
<script type="text/javascript">
if (i &lt; 0){do something countine which is you like;}
</script>
```

但是呢，对于我这种二点的青年来说，是可以接受的，因为我常常接触转义字符。 可是呢，对于文艺青年来说，代码可读性没有了，有木有！ 那么怎么办呢，CDATA就来了。 比如大家可以这么做

```html
<script>
<![CDATA[
if (i  < 0 ){ai za za di;}
]]>
</script>
```

可是出于尊敬那些不认识CDATA标签的浏览器，我们需要进一步修饰一下。 将CDATA注释一下，防止不认识CDATA的浏览器崩个溃神马的。

```html
<script>
//<![CDATA[
if (i  < 0 ){ai za za di;}
//]]>
</script>
```

接着其实还可以更进一步，既然你想到了浏览器可能不支持CDATA,那么浏览器是不是还有可能不支持js呢。 很有可能吧，我们继续做。

```html
<script>
<!--
//<![CDATA[
if (i  < 0 ){ai za za di;}
//]]>
//-->
</script>
```

写到这里，这篇日志也就写完了，不过印象中，某本权威不推荐在现在的浏览器环境中继续注释掉脚本了...

求路过的各位指正。

