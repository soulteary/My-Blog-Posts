# XSS事项 如何过滤用户输入页面内的正常内容

接下来我们继续了解如何过滤用户输入页面内的正常内容。 翻译自:[原文出处](http://soulteary.com/redirect?r=https://code.google.com/p/doctype-mirror/wiki/ArticleXSSInBodyText&k=46d9c) VER:`Updated Oct 14, 2011 by ivan@ludios.org` 假设我们有一个模版或者一个表单的HTML片段

```html
**错误:  你请求的内容 '%(query)' 没有找到。**
```

如果攻击者已经可以将请求内容插入到页面中，例如：

```html
<script>evil_script()</script>
```

那么页面中的片段将会呈现为

```html
**错误:  你请求的内容 '<script>evil_script()</script>' 没有找到。**
```

那么恶意脚本将会执行，例如将无辜的用户的cookies窃取。

## 解决方法

如何插入页面中的字符串必须将下列字符替换为相应的HTML/SGML实体:

*   替换`<tt><</tt>` 为 `<tt>&lt;</tt>`
*   替换`<tt>></tt>` 为 `<tt>&gt;</tt>`
*   替换`<tt>&</tt>` 为 `<tt>&amp;</tt>`
*   替换`<tt>"</tt>` 为 `<tt>&quot;</tt>`
*   替换`<tt>'</tt>` 为 `<tt>&#39;</tt>`

## 基本原理

小于号和大于号需要被转义，因为他们是HTML标签的分隔符，任何标签都逃不过它们，如果它们没有被转义的话（包括`script`标签）浏览器将会解析它们。

```html
<script>
```

如果上面的标签中的符号没有被转义，将不会导致安全问题，但是会引发浏览器渲染问题。可能将符号解释为实体的开启而不是正常的显示它。
如果你想了解更多，可以查看这篇文章[JavaScript 实体](http://devedge-temp.mozilla.org/library/manuals/2000/javascript/1.3/guide/embed.html#1013293)在标签之外无效。

没有必要转义这里的内容中的引号，但是有必要转义其他地方的引号，最简单的方法就是在每一处地方使用转义函数。

## 扩展阅读

- [XSS事项 你所想知道的跨站攻击](http://soulteary.com/2013/04/14/about-cross-site-scripting-xss-attacks.html)
- [Using JavaScript Expressions as HTML Attribute Values](http://devedge-temp.mozilla.org/library/manuals/2000/javascript/1.3/guide/embed.html#1013293)



