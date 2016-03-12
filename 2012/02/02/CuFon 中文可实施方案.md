# CuFon 中文可实施方案

先说一下咱自己的进度吧，目前可以动态根据传入内容进行字库缓存生成，每个页面调用common字库，和独立字库即可实现中文字体的快速替换渲染。

但是你可能会好奇，那么为什么你没有在站内使用cufon呢，很简单的原因，我还在制作字库。字库准备使用几种字体混合的方案，实现全字库可用的目标。

cufon是一种使用javascript进行快速字体渲染的技术，想要在你的网站里使用你切片时候的字体么。想解决国外模版和字体不支持中文显示的问题么，使用cufon吧。

类似技术还有sIFR、typeface、css3渲染等，但是从现在的通用性来说，cufon是最强的。

[官方GIT主页](http://promiseforever.com/redirect?url=https%3A%2F%2Fgithub.com%2Fsorccu%2Fcufon%2Fwiki%2F&key=fb5f9ec9f920cf6353325ac5a7c48155)在这里，我觉得我没必要进行cufon的详细介绍了。晚些时候，做好字体，开始考虑把函数封包翻出来分享。

_用16MB的楷书+Droid生成了120MB的STXingKai.sfd..压力很大,修改PHP转换脚本中,或许我需要制作一个WINDOWS GUI SHELL?_

