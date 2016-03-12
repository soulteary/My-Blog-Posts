# XSS事项 你所想知道的跨站攻击

本文简单的介绍如何在HTML文档中进行跨站点脚本（XSS）攻击在以及并避免跨站攻击出现的通用方法。

如果你不了解什么是跨站攻击，可以查看[跨站攻击介绍](https://code.google.com/p/doctype-mirror/wiki/ArticleIntroductionToXSS)。

本文提供的跨站脚本攻击的例子独立于任何特定的模版或者程序。

翻译自:[原文出处](http://soulteary.com/redirect?r=https://code.google.com/p/doctype-mirror/wiki/ArticleUtf7&k=6e0f7) VER:`Updated Oct 14, 2011 by ivan@ludios.org` 举个例子，这里有一个HTML片段：

```html
<title>Example document: %(title)</title>
```

那么，如果这个变量值得被进行跨站攻击，那么它最后的结果可能是这样。

```html
<title>Example document: Cross-Site Scripting</title>
```

脚本一般是基于`JavaScript (ECMAScript)`，当然，也可以使用其他的脚本语言，只要它支持被攻击者的浏览器，比如`VBScript`。

*   针对被攻击的页面，下面的内容包括（这里假设已经被攻击，比如文档body内部，或者a标签的`href`属性）
*   一个简单的例子，说明如何利用注射漏洞，即攻击者可以将攻击脚本注入HTML文档中，并执行该攻击脚本。
*   简单的指导，避免特别的关键词出现在上下文中，比如`escape`这类关键词。

解释为什么要这么做，如何防止XSS出现。

### 扩展阅读：

*   [Introduction to Cross-Site Scripting Vulnerabilities](https://code.google.com/p/doctype-mirror/wiki/ArticleIntroductionToXSS)
*   [Compartmentalizing applications within the same domain](https://code.google.com/p/doctype-mirror/wiki/ArticleCompartmentalizingApplications)
*   [HOWTO filter user input in regular body text](https://code.google.com/p/doctype-mirror/wiki/ArticleXSSInBodyText)
*   [HOWTO filter user input in tag attributes](https://code.google.com/p/doctype-mirror/wiki/ArticleXSSInAttributes)
*   [HOWTO filter user input in URL attributes](https://code.google.com/p/doctype-mirror/wiki/ArticleXSSInUrlAttributes)
*   [HOWTO filter user input in style elements and attributes](https://code.google.com/p/doctype-mirror/wiki/ArticleXSSInStyle)
*   [HOWTO filter user input in JavaScript context](https://code.google.com/p/doctype-mirror/wiki/ArticleXSSInJavaScript)
*   [HOWTO filter user input in JavaScript event handlers](https://code.google.com/p/doctype-mirror/wiki/ArticleXSSInEventHandlers)
*   [HOWTO filter user input in HTTP headers](https://code.google.com/p/doctype-mirror/wiki/ArticleXSSInHttpHeaders)
*   [HOWTO protect against malicious images and other non-HTML content](https://code.google.com/p/doctype-mirror/wiki/ArticleContentSniffing)
*   [HOWTO serve untrusted files as downloads](https://code.google.com/p/doctype-mirror/wiki/ArticleUntrustedDownloads)
*   [XSS事项 UTF-7: 消失的字符集](http://soulteary.com/2013/04/14/xss-event-missing-charset.html)
*   [Malformed UTF-8: who said "hello%EE" can't be dangerous](https://code.google.com/p/doctype-mirror/wiki/ArticleMalformedUtf8)

