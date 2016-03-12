# CSS中使用JS


## 针对IE

IE支持使用HTC behavior的方式来在CSS中执行JS代码。

```css
body {
  behavior:url(script.htc);
}
```

或者在低版本浏览器中可以使用expression方法来动态执行脚本。

```
width:expression(document.body.clientWidth > 800? "800px": "auto" );
```

## 针对Firefox

使用XBL

```css
body {
  -moz-binding: url(script.xml#excute-id);
}
```

## 针对比较古老的浏览器

此方法需要服务器指定css或者其他被引入页面的样式文件的MIME为`*/*`。

```html
<!-- /*
alert('js code excuted.');
<!-- */

<!--
#normal-rules{	border:none;}
```

CSS在解释处理时会忽略掉<!--注释内容部分，而JS在执行处理中，会将<!--转成// 也就是JS的单行注释，单行注释会把后面的/*和*/删除，同时也删除了CSS代码。

把JS跟CSS写在同一个文件利于统一交付三方进行调用，虽然文件尺寸变大，但是请求数至少减半。

并且还能做到一些其他功能，例如在只允许修改CSS的三方系统中执行JS代码。

