# 浏览器储存碎碎念之一:cookie

浏览器储存方式有很多种，掰着指头数一数：

1. Cookie(Session, Http-Only Cookie, etc...);
2. Local Storage ("User Data" for ie, "Google Gears" for old chrome);
3. Flash SharedObject;
4. IndexedDB;
5. Web SQL Database;
6. DOM Datas;
7. Hash Queries;
8. Application Cache
9. ...

面对上面五花八门的技术，我们在实践中一定会有不同的采用考虑，下面是个人看法，经验尚浅，权当抛砖引玉，欢迎留言讨论。

下面，准备好瓜子巧克力，搬个小马甲，听苏洋童鞋慢慢的道来，因为接下来的事情貌似会随意扩展，但是始终会围绕上面的主题来说:

## Cookie(Session, Http-Only Cookie, etc...)

或许对于我们来说最熟悉的就是这个家伙了吧，[Cookie在维基中描述](http://zh.wikipedia.org/wiki/Cookie)很详细了，这里就不过多介绍，只是提出一些需要注意的事情: 

这个技术是在1993年被提出的，年代比较遥远，所以接口和使用都比较简单，或者说限制也比较多。

RFC定义在此：[http://tools.ietf.org/search/rfc2965](http://tools.ietf.org/search/rfc2965)

这里简单的说一下这个技术出现的初衷：

我们每天浏览网站，基本都要使用到HTTP方式（SOCKET等其他方式暂且不表）来获取和提交内容，而HTTP是一个无状态的协议，如果想知道更多，欢迎浏览：

[HTTP无状态协议](http://baike.baidu.com/link?url=JoNH7FZr1cJVdxmhfe_Dv5yiK-TTe4tgkJcZfdynmzJ03edUFQgIIqP4jo_zIFpgKNeQk4ZChM-w9L6oPuJEFK) 以及上面的RFC。

为了保持一些状态，或者说让服务器知道你是谁，服务器该怎么“处置/处理”你，Cookie相关的技术诞生了，或者说HTTP状态管理机制诞生了，最初的目的是基于追踪哦。

它的常见形态有文本和内存资源两种状态，保存位置随不同浏览器而不同，很多优化软件附带着一个清理硬盘的功能，分分钟可以帮助你将cookie们请出你的电脑里，当然浏览器也支持这个功能。

在我们日常浏览网站的过程中，cookie可以说是无处不在，比如当我们打开百度准备搜索东西的时候，浏览器可能会发送类似以下内容的内容：

```text
GET http://www.baidu.com/ HTTP/1.1
Host: www.baidu.com
Connection: keep-alive
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
User-Agent: Mozilla/5.0 (Windows NT 6.2; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/33.0.1750.29 Safari/537.36
DNT: 1
Accept-Encoding: gzip,deflate,sdch
Accept-Language: zh-CN,zh;q=0.8,en;q=0.6,ja;q=0.4
Cookie: BAIDUID=QWEDRFTGYHUJIKL:FG=1; BDRCVFR[ERTYUI]=aeXf-RTYUI; H_PS_TIPCOUNT=5; H_PS_TIPFLAG=; H_PS_LC=RTYUI; BD_CK_SAM=1; H_PS_PSSID=ERTYUI
```

上面这段内容中的最后一项内容就是Cookie，它或许是在内存中的，或许是在文本中的，或许是两者组合后的内容，这些先不详谈。 而上面我们发送的请求中和Cookie相关的有两项内容，DNT（服务端不设置客户端Cookie的君子协定）以及Cookie； 关于DNT，不追踪协议（Don't track me），这个技术是一个“君子协定”，微软MSDN中是这样定义的：http://msdn.microsoft.com/library/ie/dn265021(v=vs.85).aspx 简单来说，当用户浏览器发送给网站带有DNT标识的请求后，网站将不对用户进行标识，以及进行一些广告或者行为的展示。 但是实际上来说的话，因为cookie关系到广告投放（用户识别策略之一），所以呢，实际效果目前不大，下面有个小例子。 当我们访问土豆去观看火影忍者的时候，如果我们播放过一段，然后刷新页面，将会享受到自动从上次播放的地方开始继续播放：

```vim
GET http://www.tudou.com/albumplay/Lqfme5hSolM/DZLqNVON5c0.html HTTP/1.1
Host: www.tudou.com
Connection: keep-alive
Cache-Control: max-age=0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
User-Agent: Mozilla/5.0 (Windows NT 6.2; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/33.0.1750.29 Safari/537.36
DNT: 1
Referer: http://www.soku.com/t/nisearch/%E7%81%AB%E5%BD%B1%E5%BF%8D%E8%80%85/
Accept-Encoding: gzip,deflate,sdch
Accept-Language: zh-CN,zh;q=0.8,en;q=0.6,ja;q=0.4
Cookie: play_item_record=......
```

即使我们发送了DNT标识，也一样会从上次播放的地方开始，当然，如果你不想享受这些额外的功能，以及不愿意让网站追踪的话，可以使用chrome的禁用cookie功能。 [![2014-01-17_173609](https://attachment.soulteary.com/2014/01/18/2014-01-17_173609.png "2014-01-17_173609")](https://attachment.soulteary.com/2014/01/18/2014-01-17_173609.png) 所以当你这样设置之后（其他浏览器方式略微不同），你发送的cookie就木有了，像下面一样:

```vim
GET http://www.tudou.com/albumplay/Lqfme5hSolM/DZLqNVON5c0.html HTTP/1.1
Host: www.tudou.com
Connection: keep-alive
Cache-Control: no-cache
Pragma: no-cache
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
User-Agent: Mozilla/5.0 (Windows NT 6.2; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/33.0.1750.29 Safari/537.36
DNT: 1
Referer: http://www.soku.com/t/nisearch/%E7%81%AB%E5%BD%B1%E5%BF%8D%E8%80%85/
Accept-Encoding: gzip,deflate,sdch
Accept-Language: zh-CN,zh;q=0.8,en;q=0.6,ja;q=0.4
```

刷新一下页面，影片是不是不再从上次播放的地方进行播放了呢。（当然，禁用广告，或许会让网络内容提供商盈利下降，为了我们能继续使用免费的服务，不如暂时放他们一马？土豆君我爱你，不要因为这篇文章就对我的UID进行特殊处理啊，我木有教大家搞你家的cookie啊） 上面简单展示了Cookie的用途之一，常见的用途还有以下几种等：

> 1.电商网站里的购物车商品记录或者你的购物车编号。 2.保持你登录某些网站后台的标识，比如我们登录时候的“记住我的帐号”这个选项的实现。 3.某些导航网站的风格自定义，比如hao123，360导航一类的可以选择页面风格的实现。 4.阅读网站以及视频网站的用户浏览记录以及播放/阅读记录。 5.各大媒体以及搜索引擎用户标识功能，用于简单的权限判断以及给你展示一些特别的内容。 6.各种用户访问统计工具依赖的标识，可以做到分析你访问网站频率，是否来过等（不是最准确的方法）。 7.某些计时器，超时判断功能等。

如果你对上面的实现有兴趣，请自行搜索，网站例子颇多，就不过多赘述了。 另外，提到cookie不得不提一下session，以及p3p。 Session是一种半服务端储存方案，将刚刚上述的内容保存在服务器的某个文件夹或者内存区域中，一般是操作系统的临时文件夹中：

```text
# Linux下的SESSION默认存放路径

/tmp
#如果你安装了PHP
/var/lib/php/session
#其他语言和客户端的话，你会比我清楚他们存在哪里
```

```text
# WINDOWS下默认存放路径

C:\WINDOWS\Temp
```

或者你可以搜索以“sess_”字符开头的文件，他们很大可能是session哦，它的内容呢，则是像上面我们提交的header中的内容一样的字符串。 <span style="line-height: 1.5em;">如果你使用PHP的话，修改php.ini文件中的session.save_path，或者在程序中可以手动指定session保存位置。</span> 如果你有桌面开发经验，或许你会对“会话标识符”这个词感到亲切：http://msdn.microsoft.com/zh-cn/library/ms178582(v=vs.100).aspx 扯远了，我们绕回来，那么cookie保存在了服务器上，有什么好处，我们又要如何使用呢？ 很简单，还是在客户端上保存cookie，但是这次我们保存的不是那么冗长的字符串，而是一个简短的hash串，我们一般叫他session_id，如果童鞋你经常使用浏览器调试工具或者抓包软件来干坏事，啊不，是学习，那么你应该见过jsessionid=xxxxxxx，这样的东西，后面的xxxxx就是保存在浏览器上的session_id，它通常会这样发出去，诸如百度：

```vim
Set-Cookie: H_PS_PSSID=XXXXXXXXXXXX; path=/; domain=.baidu.com
```

然后服务器就会自己匹配XXXXXXX这串看似乱码的Hash，然后到服务器保存刚刚我们称之为Session的Cookie的地方来获取你在服务器上临时保存的内容。 或许你会问，我刚刚不是说session_id名字是jsessionid吗，哦，我没说清楚，它的标识是可以随意修改的，因为各家公司的服务器使用不同，并且程序中也可以设定这个参数的名称。 如果你的网站的用户以及你要做的事情不会太多，那么你可以把SESSION当作持久储存来用，但是鉴于它的设计（参见RFC），以及保存位置（TMP），我们一般当这货是一个带有时效性的保存配置的文件或者内存区域。 扩展阅读：有过使用PHP的网友测试，SESSION在PHP使用中，最大容量为php.ini文件中设置的memory_limit的大小 http://blog.csdn.net/mapingsheng/article/details/8099902 当然，还真的有人将SESSION持久化保存后来用，比如有的人会将SESSION内容保存在MYSQL,或者NOSQL中，包括存在基于内存的memcache，redis等缓存中。 现在应该会有人吐槽说，那既然我的浏览器上保存着我在某家网站上的标识，那我把这个ID修改一下，是不是就可以访问其他人的资料了。 恭喜你，童鞋你答对了，但是一般来说，网站会将这个标识设置为HTTP-ONLY，也就是说，对于客户端是只读的。 不过呢，有的程序员写码的时候不认真，忘记设置HTTP-ONLY属性，致使这个ID可以被本地读写，或者被别有用心的人，通过浏览器漏洞将这个只读的字符串更改为其他内容。 这里扩展一些额外的信息，感兴趣的你可以阅读一下：

1. CSRF（Cross-site request forgery跨站请求伪造，也被称为“one click attack”或者session riding，通常缩写为CSRF或者XSRF，是一种对网站的恶意利用，[百科定义](http://baike.baidu.com/link?url=wgDdz8LsC-v15vrr_4FVXl30GqG4LuQCQyGKUuW2_khsFRUkfRae8I3crjMOFhAZ)。
2. 之前翻译和没翻译完的GOOGLE上的安全相关的内容:http://www.soulteary.com/?s=xss%E4%BA%8B%E9%A1%B9
3. 2012 版猥琐获得 httponly cookie 权限教程,[地址](http://www.oschina.net/translate/xss-gaining-access-to-httponly-cookie)
4. 乌云上的XSS漏洞[相关内容](http://www.wooyun.org/searchbug.php?q=XSS)。

额外说一下,如果你是Php Coder，如果使用REQUEST方式获取参数的时候，这里有一个小问题：

> request_order string This directive describes the order in which PHP registers GET, POST and Cookie variables into the _REQUEST array. Registration is done from left to right, newer values override older values. If this directive is not set, variables_order is used for $_REQUEST contents. Note that the default distribution php.ini files does not contain the 'C' for cookies, due to security concerns. http://www.php.net/manual/zh/reserved.variables.request.php

请注意cookie获取参数的覆盖问题。 谈了这么久HTTP-ONLY，我们来看看MSDN上它的具体定义吧：http://msdn.microsoft.com/zh-cn/library/system.web.httpcookie.httponly.aspx

> 如果 Cookie 具有 **HttpOnly** 属性且不能通过客户端脚本访问，则为 **true**；否则为 **false**。默认为 **false**。 Microsoft Internet Explorer 版本 6 Service Pack 1 和更高版本支持 Cookie 属性 HttpOnly，该属性有助于缓解跨站点脚本威胁，这种威胁可能导致 Cookie 被窃取。 窃取的 Cookie 可以包含标识站点用户的敏感信息，如 ASP.NET 会话 ID 或 Forms 身份验证票证，攻击者可以重播窃取的 Cookie，以便伪装成用户或获取敏感信息。 如果兼容浏览器接收到 HttpOnly Cookie，则客户端脚本不能对它进行访问。 将 HttpOnly 属性设置为 true，并不能防止对网络频道具有访问权限的攻击者直接访问该 Cookie。 针对这种情况，应考虑使用安全套接字层 (SSL) 来提供帮助。 工作站的安全也很重要，原因是恶意用户可能使用打开的浏览器窗口或包含持久性 Cookie 的计算机，以合法用户的标识获取对网站的访问。

扩展阅读：

[Mitigating Cross-site Scripting With HTTP-only Cookies](http://msdn.microsoft.com/zh-cn/library/ms533046(vs.85).aspx)

我们看到定义格式为：

```vim
# HTTP ONLY COOKIE FORMAT

Set-Cookie: <name>=<value>[; <name>=<value>]
[; expires=<date>][; domain=<domain_name>]
[; path=<some_path>][; secure][; HttpOnly]
```

最后的HttpOnly不区分大小写，对于不支持cookie的浏览器，那么就会忽略这个标记。 这也就是为什么我们常常会说某些手机浏览器不支持session，其实是不支持HttpOnly的cookie。 解决方法也很简单，就是将session id添加到URL地址请求字符串中，如?a=b&session_id=xxxxx的形式传递给服务器。 现在该说一下P3P了，P3P是W3C提出的一个规范，文档较多，简单说一下，它有2个版本，1.0版本和我们要提到的cookie没有神马关系， 但是1.1版本将cookie收纳进来，如果读不下去下面的前两条内容，知道这货可以用来解决一些IE的跨域问题即可，当然，别忘记单点登录的实现手段之一也是这货哦。

[Platform for Privacy Preferences (P3P) Project](http://www.w3.org/P3P/)
[The COOKIE-INCLUDE and COOKIE-EXCLUDE elements](http://www.w3.org/TR/P3P11/#cookies)
[Cookie 跨域传递](http://wenku.baidu.com/link?url=cckV_ztVi3FsaslvY8r4dykatpl9deFXGIuDg78K_ed9LCU1StkU0g6IgRNev0c6duxEbZkZbm7rbhtdkE6_G_6dVvlwb76sK5JMKeo300_)
请搜索：Cookie跨域操作
优酷土豆他俩搞基必备的header信息，请抓包查看
还有JS教主写的 [关于p3p 简洁策略,以及浏览器的支持情况.](http://www.cnblogs.com/_franky/archive/2011/03/16/1985954.html)
[PHP利用P3P实现跨域](http://blog.csdn.net/jjmaiz/article/details/8014292)

扯了这么多里里外外的事情，我们了解到了cookie它是一种标记手段，可以在本地客户端上进行存储，可以主动和被动修改，以及存在跨域和篡改的安全策略和问题。 那么它还有那些属性呢，RFC里对设置cookie允许2个版本，其中版本0允许的属性有以下5种：

| 属性项 | 属性项介绍 |
| --- | --- |
| NAME=VALUE | 键值对，可以设置要保存的 Key/Value，注意这里的 NAME 不能和其他属性项的名字一样 |
| Expires | 过期时间，在设置的某个时间点后该 Cookie 就会失效，如 expires=Wednesday, 09-Nov-99 23:12:40 GMT |
| Domain | 生成该 Cookie 的域名，如 domain="soulteary.com" |
| Path | 该 Cookie 是在当前的哪个路径下生成的，如 path=/admin/ |
| Secure | 如果设置了这个属性，那么只会在 SSH 连接时才会回传该 Cookie |

版本1就比较多了：

| 属性项 | 属性项介绍 |
| --- | --- |
| NAME=VALUE | 与 Version 0 相同 |
| Version | 通过 Set-Cookie2 设置的响应头创建必须符合 RFC2965 规范，如果通过 Set-Cookie 响应头设置，默认值为 0，如果要设置为 1，则该 Cookie 要遵循 RFC 2109 规范 |
| Comment | 注释项，用户说明该 Cookie 有何用途 |
| CommentURL | 服务器为此  Cookie 提供的 URI 注释 |
| Discard | 是否在会话结束后丢弃该 Cookie 项，默认为 fasle |
| Domain | 类似于 Version 0 |
| Max-Age | 最大失效时间，与 Version 0 不同的是这里设置的是在多少秒后失效 |
| Path | 类似于 Version 0 |
| Port | 该 Cookie 在什么端口下可以回传服务端，如果有多个端口，以逗号隔开，如 Port="80,81,8080" |
| Secure | 类似于 Version 0 |

Secure这个属性主要用于SSL方式的连接中，如果是SSL方式连接网站，才会将Cookie发送到服务器。 Domain涉及到的问题就是我们刚刚说到的跨域，不管是正常的并购后的网站搞基（优酷土豆），还是大公司下面的多个域名（sina&&weibo，qq&&pengyou）通信，都会牵扯到它。 这里还有一个小issue，屈屈曾经写过一篇日记，关于js 设置domian，具体我暂时不说，如果你看到了，请不要惊讶：

[Webkit下最无敌的跨大域方案](https://www.imququ.com/post/document-domain-bug-in-webkit.html)

另外，与HttpOnly相生的有一个叫做HostOnly的技术：

[你所不知道的HostOnly Cookie](https://www.imququ.com/post/host-only-cookie.html)
如果你使用.NET来设置，或许这个也值得看一下：[HttpCookie.HttpOnly VS Cookie.HttpOnly?(downmoon原创)](http://www.cnblogs.com/downmoon/archive/2008/09/11/1289298.html)

domain说完了，我们来看看max-age，这个属性是不是很眼熟，没错，我们的cache-control里也有使用，他们的单位都是秒，但是请注意，IE6789以及10不支持这个属性。 max-age如果在支持的浏览器中设置为负数，那么这个cookie将会随着浏览器的关闭而被扫地出门， 这里有一点需要注意的是，它和expires的区别：

简单来说，expires支持的是具体的时间，浏览器支持度好。
expires也是从HTTP1.1规范借鉴过来的，不过这个在HTTP 1.1中是一个不推荐使用属性，使用成本比max-age高，多一步转换的过程，参见下面的代码。
如果同时设置了这两个属性，ie忽略max-age，标准浏览器忽略expires。
标准浏览器中可以设置max-age=0来删除cookie，当然你可以辅助的设置对应key的value的内容为空字符串，某些服务端里setMaxAge(-1)则是将cookie存于内存，这个是不一样的哦。
如果你两个属性都不设置，那么关闭浏览器，这个cookie就自动删除了。
测试页面：http://mrcoles.com/media/test/cookies-max-age-vs-expires.html
参考：[HTTP Cookies: What's the difference between Max-age and Expires](http://mrcoles.com/blog/cookies-max-age-vs-expires/)
PHP WIKI: [https://wiki.php.net/rfc/cookie_max-age](https://wiki.php.net/rfc/cookie_max-age)

```js
// expires和max-age比较

//expires
var d = new Date();
d.setTime(d.getTime() + 5*60*1000); // in milliseconds
document.cookie = 'foo=bar;path=/;expires='+d.toGMTString()+';';

//max-age
document.cookie = 'foo=bar;path=/;max-age='+5*60+';';
```

上面代码中相比max-age，expires多一个转换的过程。 额外提一下，关于max-age，rfc里的定义是：

> Cookie2 has a value for Max-Age of zero, the (old and new) cookie is discarded. Otherwise a cookie persists (resources permitting) until whichever happens first, then gets discarded: its Max-Age lifetime is exceeded; or, if the Discard attribute is set, the user agent terminates the session.

这里，有一些前端童鞋（愚人码头）踩过一个小坑：

IE6中对于Session的保存（http only session id）有问题，如果用户实在网页上跳转打开页面或新开窗口（包括target="_blank"，鼠标右键新开窗口），都是在同一个Session内。
如果用户新开浏览器程序或者说是使用新进程再打开当前的页面就不是同一个Session。其他浏览器只要Session存在，还是同一个Session，cookie也能共享。
http://d2.sodao.com/?p=360

需要额外注意的是:保存cookie的时候，记得使用encodeURIComponent转码，如果你不想自己写，可以使用jquery cookie或者类库相关接口。 终于要绕回我们谈的浏览器存储的主题了，数据来源，有测试代码：[http://browsercookielimits.x64.me/](http://browsercookielimits.x64.me/)

| Browser | Max Cookies | Max Size Per Cookie | Max Size Per Domain<sup>1</sup> | Usage<sup>2</sup> |
| --- | --- | --- | --- | --- |
| Chrome 4 |
| Chrome 5-7 | 70 | 4096 bytes | NA |
| Chrome 8-25* | 180 | 4096 bytes | NA | 48.4% |
| * Checked on 8/10/12/13/14/15/17/25\. Unverified on 9/11/16/18-24, just assumed |
| FireFox 2/3.6.6 | 50 | 4097 characters | NA | 1.3% (versions before 3.6) |
| FireFox 3.6.13-19* | 150 | 4097 characters | NA | 29.8% |
| * verified on 3.6.13/4/8/9/10/14/19\. unverified 5-8/11-13/15-18, just assumed |
| IE 6 [unpatched](http://support.microsoft.com/kb/941495) | 20 | 4096 characters | 4096 characters |
| IE 6 | 50 | 4096 characters | 4096 characters | 0.3% WTF |
| IE 7 [unpatched](http://support.microsoft.com/kb/941495) | 20 | 4095 characters | 4095 characters |
| IE 7 | 50 | 4095 characters | 4095 characters | 1.0% |
| IE 8/9/10 | 50 | 5117 characters | 10234 characters | 13.1% |
| Opera 8/9/10 | 30 | 4096 bytes | 4096 bytes |
| Opera 11 | 60 | 4096 bytes | 4096 bytes | 0.1% |
| Opera 12 | 60 | 5117 bytes | 12093 bytes | 1.1% |
| Safari 3 |
| Safari 4 | 0.1% |
| Safari 5 | 600 | 4096 bytes | 4096 bytes | 1.4% |
| Safari 6 | 2.7% |
| Safari on mac<sup>3</sup> | 600<sup>4</sup> | 4093 bytes<sup>4</sup> | most likely 4093 bytes<sup>4</sup> |
| Android 2.1/2.3.4 | 50 | 4096 bytes | NA |
| Safari Mobile 5.1 | 600<sup>5</sup> | 4093 bytes<sup>5</sup> | 4093 bytes |

尽管上面的表中有NB浏览器对cookie数量支持到500，大小支持到5100等，总量支持到1.2Mb，但是呢，木桶定律，我们要看最短板。

我们在使用cookie作为存储的时候，请遵循尽少使用，以20个为上限，4K数据为上限。 之所以尽量少使用这个家伙的原因有：

1.每次访问服务器都会提交这些数据，如果你存了4K，那么每次都会提交4K到服务器，损耗的不仅仅有服务器的解析时间，还有你的上传时间，以及从硬盘读取的时间。
2.如果你吐槽说，我的cookie被浏览器加载内存中了，以及我是SSD，读取无压力，那么上面那条重复强调服务器解析造成不必要浪费这段，作为用户可以不关系，但是开发人员如果都不介意的话，那请右上角点X。
3.程序读写保存时如果没有考虑已保存的文件的大小，很容易超出服务端支持的上限，发生传送的数据不完整以及服务器不响应你的请求。
4.客户端脚本处理速度不佳，呃，或许你说我们现在都用webkit内核的高端浏览器了，那么请想想你在高速浏览的时候，还有小白用户使用着ie呢。

刚刚说的这些，或许你会觉得吹毛求疵，那么我们来计算一下： 比如某网站排除rest接口的访问量，每天在1000万左右，每次请求都传递4k的cookie的话，大概每天要消耗近40G的流量，如果cookie不是4k而是更大的话，比如20K（分离域名），那么就是200G流量。 对于百度淘宝京东这类网站的话，必须最小化cookie，这里就又出现了一项技术，或者说技巧：服务器压缩cookie

淘宝无线分享：[压缩Cookie，降低网络流量](http://gubaojian.blog.163.com/blog/static/166179908201222154447988/)
有的时候stackoverflow也不靠谱，请百度和谷歌一起使用。

除了这项技术/技巧外，我们还有一个可选的方案:free-domain cookie 说到这个啊，大家难免会想到y-slow的雅虎军规，木有错，就是那个玩意... 简单的说一下它的实现，因为浏览器童鞋是一个认真的好孩纸，他会在请求和你域名一致的静态资源的时候发送和访问你域名页面一样的cookie，神马图片啊，动画啊，音频啊，都在其中， 换句话说，上面举例中的1000万PV，实际传送的cookie会比我们算的大太多，当然这也不是绝对的，如果我们的coder有意设置了cdn域名来分离静态资源的话，由于服务器没有设置任何cookie， 所以在访问静态资源的时候就不会再发送大量的cookie给服务器，同样也是起到了加速的效果，已经节约了带宽。 另外，刚刚上文提到使用jquery的童鞋可以使用jquery-cookie这个插件，那么使用原生js又不想用DOM蛋疼接口来写cookie管理工具的童鞋可以用下面的这个脚本：

```js
// Copyright (c) 2012 Florian H., https://github.com/js-coder https://github.com/js-coder/cookie.js

!function (document, undefined) {

	var cookie = function () {
		return cookie.get.apply(cookie, arguments);
	};

	var utils = cookie.utils =  {

		// Is the given value an array? Use ES5 Array.isArray if it's available.
		isArray: Array.isArray || function (value) {
			return Object.prototype.toString.call(value) === '[object Array]';
		},

		// Is the given value a plain object / an object whose constructor is `Object`?
		isPlainObject: function (value) {
			return !!value && Object.prototype.toString.call(value) === '[object Object]';
		},

		// Convert an array-like object to an array – for example `arguments`.
		toArray: function (value) {
			return Array.prototype.slice.call(value);
		},

		// Get the keys of an object. Use ES5 Object.keys if it's available.
		getKeys: Object.keys || function (obj) {
			var keys = [],
				 key = '';
			for (key in obj) {
				if (obj.hasOwnProperty(key)) keys.push(key);
			}
			return keys;
		},

		// Unlike JavaScript's built-in escape functions, this method
		// only escapes characters that are not allowed in cookies.
		escape: function (value) {
			return String(value).replace(/[,;"\\=\s%]/g, function (character) {
				return encodeURIComponent(character);
			});
		},

		// Return fallback if the value is not defined, otherwise return value.
		retrieve: function (value, fallback) {
			return value == null ? fallback : value;
		}

	};

	cookie.defaults = {};

	cookie.expiresMultiplier = 60 * 60 * 24;

	cookie.set = function (key, value, options) {

		if (utils.isPlainObject(key)) { // Then `key` contains an object with keys and values for cookies, `value` contains the options object.

			for (var k in key) { // TODO: `k` really sucks as a variable name, but I didn't come up with a better one yet.
				if (key.hasOwnProperty(k)) this.set(k, key[k], value);
			}

		} else {

			options = utils.isPlainObject(options) ? options : { expires: options };

			var expires = options.expires !== undefined ? options.expires : (this.defaults.expires || ''), // Empty string for session cookies.
			    expiresType = typeof(expires);

			if (expiresType === 'string' && expires !== '') expires = new Date(expires);
			else if (expiresType === 'number') expires = new Date(+new Date + 1000 * this.expiresMultiplier * expires); // This is needed because IE does not support the `max-age` cookie attribute.

			if (expires !== '' && 'toGMTString' in expires) expires = ';expires=' + expires.toGMTString();

			var path = options.path || this.defaults.path; // TODO: Too much code for a simple feature.
			path = path ? ';path=' + path : '';

			var domain = options.domain || this.defaults.domain;
			domain = domain ? ';domain=' + domain : '';

			var secure = options.secure || this.defaults.secure ? ';secure' : '';

			document.cookie = utils.escape(key) + '=' + utils.escape(value) + expires + path + domain + secure;

		}

		return this; // Return the `cookie` object to make chaining possible.

	};

	// TODO: This is commented out, because I didn't come up with a better method name yet. Any ideas?
	// cookie.setIfItDoesNotExist = function (key, value, options) {
	//	if (this.get(key) === undefined) this.set.call(this, arguments);
	// },

	cookie.remove = function (keys) {

		keys = utils.isArray(keys) ? keys : utils.toArray(arguments);

		for (var i = 0, l = keys.length; i < l; i++) {
			this.set(keys[i], '', -1);
		}

		return this; // Return the `cookie` object to make chaining possible.
	};

	cookie.empty = function () {

		return this.remove(utils.getKeys(this.all()));

	};

	cookie.get = function (keys, fallback) {

		fallback = fallback || undefined;
		var cookies = this.all();

		if (utils.isArray(keys)) {

			var result = {};

			for (var i = 0, l = keys.length; i < l; i++) {
				var value = keys[i];
				result[value] = utils.retrieve(cookies[value], fallback);
			}

			return result;

		} else return utils.retrieve(cookies[keys], fallback);

	};

	cookie.all = function () {

		if (document.cookie === '') return {};

		var cookies = document.cookie.split('; '),
			  result = {};

		for (var i = 0, l = cookies.length; i < l; i++) {
			var item = cookies[i].split('=');
			result[decodeURIComponent(item[0])] = decodeURIComponent(item[1]);
		}

		return result;

	};

	cookie.enabled = function () {

		if (navigator.cookieEnabled) return true;

		var ret = cookie.set('_', '_').get('_') === '_';
		cookie.remove('_');
		return ret;

	};

	// If an AMD loader is present use AMD.
	// If a CommonJS loader is present use CommonJS.
	// Otherwise assign the `cookie` object to the global scope.

	if (typeof define === 'function' && define.amd) {
		define(function () {
			return cookie;
		});
	} else if (typeof exports !== 'undefined') {
		exports.cookie = cookie;
	} else window.cookie = cookie;

}(document);
```

这篇暂时先写到这里，其余的坑，容我慢慢来填。 本文或许还会出现的内容：手机端对于cookie和session的支持度 如果你发现我书写有误，或者有一些更好的资源，不妨联系我更新本文，或者修正错误，感谢你花费宝贵的时间来阅读这些。

--EOF--


