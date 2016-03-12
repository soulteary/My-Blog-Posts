# [WIN]卸载WINDOWS桌面搜索

[![windows-search](https://attachment.soulteary.com/2010/08/15/windows-search.jpg "windows-search")](https://attachment.soulteary.com/2010/08/15/windows-search.jpg) 

今天更新系统备份的时候，突然想起“Windows 桌面搜索 3.01”不是很好用，需要预先进行索引，而且速度不是很快，还不如以前自带的搜索助手，卸载之心油然而生。

<!-- more -->

因为我有一个习惯，就是安装完补丁后，将$NtUnInstallKBXXXXXX$都删除，节约空间，所以常规的卸载办法行不通。

网上搜索，看到有人已经找到卸载方法。原文地址：http://bbs.tech.163.com/bbs/tech_0001/148391467,1.html

看完后，正准备按部就班的做的时候，突然想到一个更加简洁的方法：覆盖安装，卸载文件提取。

首先到这里进行补丁下载：http://www.microsoft.com/downloads/details.aspx?displaylang=zh-cn&amp;FamilyID=738fc2de-49b9-4e69-9227-2206277ab7c9

然后使用解包工具解包，由于刚刚恢复系统，没有解包工具，于是双击补丁，进行覆盖操作，无论安装成功与否，

Windows文件夹内，都会出现$NtUnInstallKB917013$，这个卸载文件夹，进入文件夹spuninst，运行spuninst.exe

卸载完成。但是我们还需要做一些后续的善后工作，使用unlock或者其他的anti工具，进行一些dll模块卸载，然后删除Program Files文件夹内的Windows Desktop Search,接着清理下注册表【某优化大师，或者360都可以..】，大功告成。

