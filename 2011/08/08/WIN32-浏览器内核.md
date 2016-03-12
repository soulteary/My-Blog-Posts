# [WIN32]浏览器内核

话说刚刚发的[日志](http://promiseforever.com/2011/08/07/js%e6%b5%8f%e8%a7%88%e5%99%a8%e7%89%88%e6%9c%ac%e6%8e%a7%e5%88%b6%e8%84%9a%e6%9c%ac.html)的脚本中有一段代码

```js
// Regex Settings
var IERegularExpression = new RegExp("MSIE ([0-9]{1,})");
var FFRegularExpression = new RegExp("Firefox");
var ChromeRegularExpression = new RegExp("Chrome");
var OperaRegularExpression = new RegExp("Opera");
var SafariRegularExpression = new RegExp("Safari");
var TridentRegularExpression = new RegExp("Trident\/([0-9]{1,}[\.0-9]{0,})");
```

前几个常常浏览服务器日志或者有涉及过user-agent的人都见过或者用过吧。
最后一个trident看起来比较眼生，于是搜索了一番，找了一些资料。

简单来说，就如同google safari使用webkit一样，ie目前使用的内核就是trident。当然mozilla firefox 使用gecko。

gecko，webkit 属于开源代码，而trident 你懂的。

<!-- more -->

一个内核比较重要的除了速度，安全意外，就是解释能力了。
解释能力的话，个人感觉有3个内容：图文排版(包括样式表)，脚本运行，插件运行(DOM)。


acid3可以测试浏览器对脚本的支持，并给出分数。 

[http://acid3.acidtests.org/](http://promiseforever.com/redirect?url=http://acid3.acidtests.org&key=a9bc2e126e1a4877e4a59e0b0b85347d) 

运行之后，相信你对内核也有自己的看法了。 

而且看到有[网友](http://promiseforever.com/redirect?url=http://www.cnblogs.com/lyongde/archive/2011/05/05/2037828.html&key=2fbf58ba024791ae29e655691d81adf4)说 `IE内核浏览器并不支持CMKY模式的的图片文件，所以不能显示` 这个没有测试过，留个备案吧。

