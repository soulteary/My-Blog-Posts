# XSS事项 UTF-7: 消失的字符集

说到XSS，不得不提字符集了。

> UTF-7是为了SMTP而设计的，而SMTP作为电子邮件传输标准之一，其格式为US-ASCII，不允许使用超过ASCII定义的字符范围意外的位元数值，所以说SMTP不支持8位元的数据。UTF-7的出现解决了这个问题，它使用BASE64来表示8位元的数据使用7位元表示不可见的ASCII字符。

翻译自:[原文出处](http://soulteary.com/redirect?r=https://code.google.com/p/doctype-mirror/wiki/ArticleUtf7&k=6e0f7) VER:`Updated Oct 14, 2011 by ivan@ludios.org`

`<script>alert(1)</script>`

上面的脚本内容被UTF-7编码后的结果如下：

`+ADw-script+AD4-alert(1)+ADw-/script+AD4-`

一般来说，我们会在HTTP消息头部定义`Content-Type`或者在HTML文件的`META`标签中定义字符集。 而当服务器没有包含一个明确的字符集编码的时候，诸如IE这样的浏览器则会猜测数据的字符集。 如果说用户输入`-- say, +ADw-script+AD4-alert(1)+ADw-/script+AD4- --`过早的出现在文档中，浏览器可能会猜测页面的编码为UTF-7。

然而杯具的是，这些无害的输入将会被当作HTML文档解析执行，也就是说，用户将看到一个弹出框，内容是1。

## 解决方案

在一定程度上尽可能的验证用户的输入。 在程序中对某些输入做限制，比如仅允许输入ASCII[0-9A-Z]，那么UTF-7字符串绝对不会出现在页面中。 始终设置字符集，不论是在HTTP消息头部定义`Content-Type`还是在HTML中定义META，但是要注意的是，HTML中的定义最好在最开始的[512字节](http://www.whatwg.org/specs/web-apps/current-work/multipage/semantics.html#charset)以内。

HTML文档中在`title`标签之前设置你实际内容需要的字符集编码，胡乱声明一个字符集比不声明更加的糟糕。

即使你的页面只将执行302重定向，也要确保它有一个字符集。 在HTTP头中设置编码字符集，使用Content-Type头的charset参数：

`Content-Type: text/html; charset=UTF-8`

在HTML文档中设置编码字符集，使用`meta`标签：

`<meta charset="utf-8">`

重要提示： `meta`标签必须出现在文档的任何内容之前，否则可能被攻击者控制，比如在`title`标签中包含和文章开始中出现的被编码过的脚本内容。

## 扩展阅读:

*   [XSS事项 你所想知道的跨站攻击](http://soulteary.com/2013/04/14/about-cross-site-scripting-xss-attacks.html)
*   [UTF-7 on Wikipedia](http://en.wikipedia.org/wiki/UTF-7)
*   [UTF-7 RFC](http://www.ietf.org/rfc/rfc2152.txt)
*   [Online tool to convert between different encodings](http://www.motobit.com/util/charset-codepage-conversion.asp)

