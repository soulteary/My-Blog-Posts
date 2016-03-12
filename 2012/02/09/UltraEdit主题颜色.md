# UltraEdit主题颜色

用了很久的UE,突然觉得也改换换颜色了，VIM的样式有很多，感兴趣的童鞋可以继续修改。 贴上2种修改方法，1种是使用国外朋友写的UE Color Scheme进行UE的颜色设置，另外一种是直接修改UE的INI配置文件。 先说简单的吧，使用工具

> UltraEdit Color Scheme Manager Latest Version 1.7.2.1004 Latest Release Date: February 24, 2011 Compatible with: UltraEdit 15 through 17 and UEStudio 9.3 through 10.3

如作者所说，支持UE15以及17,UEStudio 9.3到10.3。 软件很简单，自带10多个颜色配置。使用时需要先找到你的INI配置文件，如图所示。

[![20120130221251](https://attachment.soulteary.com/2012/02/09/20120130221251.jpg "20120130221251")](https://attachment.soulteary.com/2012/02/09/20120130221251.jpg) 
 
然后在File菜单栏,选择Open Scheme,打开你喜欢的配色,如果没有?自己做呗,详见文末VIM颜色配置。 打开你喜欢的配色后，在Color Scheme 选项卡中 Export Color Only to UE，记得先关闭UE，否则UE.INI配置没法写入。 
 
重新启动你的UE,界面配色就变了。 
 
[![20120130222048](https://attachment.soulteary.com/2012/02/09/20120130222048.jpg "20120130222048")](https://attachment.soulteary.com/2012/02/09/20120130222048.jpg) 

接着你可以在你常用的语言中慢慢设置其他的高亮细节。 再来说说方法2，自己修改 **X:\Documents and Settings\your user name\Application Data\IDMComp\UltraEdit**中的**Uedit32.INI** 下面以Tango Dark配色为例。 打开Uedit32.INI，查找**General Editor Scheme**，如果有，用下面的代码替换，如果没有，那么将代码添加到文件末尾。

```ini
[User Color Schemes]
0=Tango Dark;13621203;3552302;15527662;8751752;13621203;5461845;8081525;0;13621203;0;13621203;3552302;0;164;0;1776151;0;8751752;0;1776151
1=Tango Dark Alt;13621203;3552302;15527662;8751752;13621203;2631458;8081525;0;13621203;0;13621203;3552302;0;164;0;1776151;0;5461845;0;1776151
```

打开你的UE，在配置菜单里找到图中的设置，选择新的Tango Dark颜色配置。

[![20120130222841](https://attachment.soulteary.com/2012/02/09/20120130222841.jpg "20120130222841")](https://attachment.soulteary.com/2012/02/09/20120130222841.jpg)

接着呢,使用正则表达式搜索 "Language * Colors",如果找到内容的话,就替换整个内容小节，如果没有找到，那么就添加在尾部。

```ini
[Language 14 Colors]
Colors=13621203,8751752,8751752,54509,13621203,7256553,13606770,3465866,2697711,4108284,31221,1146305,13621203,
Colors Auto Back=1,1,1,1,1,1,1,1,1,1,1,1,1,
Colors Back=3552302,3552302,3552302,3552302,3552302,3552302,3552302,3552302,3552302,3552302,3552302,3552302,3552302,
Font Style=0,0,0,0,0,1,0,0,0,0,0,0,0,
```

最后进入UE，自己调整细节就哦了。

[![20120130225344](https://attachment.soulteary.com/2012/02/09/20120130225344.jpg "20120130225344")](https://attachment.soulteary.com/2012/02/09/20120130225344.jpg)

VIM 配色例子:[浏览](http://promiseforever.com/redirect?url=http%3A%2F%2Fwww.vi-improved.org%2Fcolor_sampler_pack%2F&key=af4655ee97c25ae39837a3b27bc5b9ea)

UE颜色配置修改工具[download id="103"]

