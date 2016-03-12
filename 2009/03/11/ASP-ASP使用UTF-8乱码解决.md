# [ASP]ASP使用UTF-8乱码解决

使用GB或GBK不产生乱码，但是使用UTF-8却出现了乱码的解决方法。

Q:为什么在ASP里指定了codepage=65001还经常显示乱码呢？ A:UTF-8编码之所以被越来越多的人接受、喜欢以至于推崇，原因在于它的通用性。

它的出现照顾了全球的文字以及各种不同工作环境的不同类型，不同版本的浏览器。

<!-- more -->

使用UTF-8页面编码，需要同时指定codepage及charset字符集。 ASP页面顶部通常会出现

```asp
<%@LANGUAGE="VBSCRIPT" CODEPAGE="936″%>
@LANGUAGE="VBSCRIPT" '通常可以省略不写，便于浏览和提高执行效率。
```

codepage的属性则代表了页面的编码类型。 

例如:936代表是简体中文,而950代表繁体中文,65001则是UTF-8编码了。 

如果要将编码由简体中文换成URTF-8，那么需要:

```asp
<%@LANGUAGE="VBSCRIPT" CODEPAGE="65001″%>
```

接着需要将HTML元标记中的:

```html
<meta http-equiv="Content-Type" content="text/html; charset=gb2312″ />
```

修改为:

```html
<meta http-equiv="Content-Type" content="text/html; charset=utf-8″ />
```

```vb
<%
Response.Write "UTF-8中文测试"
%>
```

关键语句：

```vb
<%@ CODEPAGE=65001 %>
<% Response.CodePage=65001%>
<% Response.Charset="UTF-8″ %>
```

