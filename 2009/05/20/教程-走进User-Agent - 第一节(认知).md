# [教程]走进User-Agent - 第一节(认知)

原创文章，转载请说明出处，谢谢合作。

User-Agent说白了就是一个标识而已，它的使用范围很广，可以识别浏览器的类型来加载css样式表，某些手机站点自动转向对应手机下载专区，也可以进行管理员或者会员的登录限制，收集浏览器市场占有率等。

如果你还是对这个抽象的概念不是很清楚的话，跟我做：

新建一个文本文档，在里面输入如下内容

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "<a href="http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd</a>">
<html xmlns="<a href="http://www.w3.org/1999/xhtml">http://www.w3.org/1999/xhtml</a>">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>User-Agent 显示</title>
</head>
<body>
<script type="text/javascript">
document.write(navigator.userAgent);
</script>
</body>
</html>
```


接着保存为useragent.html，双击打开，如果你是Opera浏览器的话，或许会显示如下信息

`Opera/10.00 (Windows NT 5.1; U; zh-cn) Presto/2.2.0`

上面的代码理论上支持IE,FF,OPERA等浏览器。

你也可以这样做，打开你的浏览器，在地址栏输入下面的代码，并按下你的回车，或许你的浏览器会显示一些东西：

```js
javascript:document.write(navigator.userAgent);document.close();
```

这段代码支持IE,FF,OPERA等浏览器

你现在应该对User-Agent有所了解了吧。:)


