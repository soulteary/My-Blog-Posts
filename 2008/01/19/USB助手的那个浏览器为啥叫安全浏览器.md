# USB助手的那个浏览器为啥叫安全浏览器？

USB助手中使用的浏览器内核不是系统默认的IE内核，而是使用了而是Mozilla的Geko。

三种内核在同时访问带有浏览器检测功能的网站的情况如下：

- IE:
	- [![ie-broswer-info](https://attachment.soulteary.com/2008/01/19/ie-broswer-info.jpg "ie-broswer-info")](https://attachment.soulteary.com/2008/01/19/ie-broswer-info.jpg)

再发Mozilla的火狐的...
- Mozilla:
	- [![firefox-broswer-info](https://attachment.soulteary.com/2008/01/19/firefox-broswer-info.jpg "firefox-broswer-info")](https://attachment.soulteary.com/2008/01/19/firefox-broswer-info.jpg)
- 最后来张我的
	- [![my-broswer-info](https://attachment.soulteary.com/2008/01/19/my-broswer-info.jpg "my-broswer-info")](https://attachment.soulteary.com/2008/01/19/my-broswer-info.jpg)[/three_fifth_last]

众所周知，由于完全禁用了ActiveX，Mozilla在某方面比IE安全，并且在JS执行上，Mozilla比IE更为强大。（虽说最近Mozilla也爆出好多洞。）

我用的这个内核没有加载火狐的各种插件和花哨的功能，只是采用了其基础的解释器做页面渲染解释而已，没有调用系统的功能的权限。

所以，除非出现基础解释器上的漏洞，否则可谓刀枪不入。


