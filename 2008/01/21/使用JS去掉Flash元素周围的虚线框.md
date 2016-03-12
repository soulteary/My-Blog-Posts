# 使用JS去掉Flash元素周围的虚线框

自从微软升级ActiveX控件后，点击Flash广告或者游戏什么的需要多点击一次鼠标。而且Flash元素周围还有一圈虚线框，非常讨厌。


## 简单方案

升级你的Dreamweaver到8.02或者更高版本，网上有Dreamweaver8.02升级补丁。

使用新版本DW打开你的带有FLASH的页面文件，可以自动升级插入页面的FLASH脚本，在确认点击确认升级对话框之后，默认会在网站根目录生成Scripts文件夹、AC_RunActiveContent.js文件，此后再打开转化后的页面，Flash就不带虚线框。

## SWFObject 方案

资料参考:

- https://github.com/swfobject/swfobject

## 系统补丁方案

微软已经发布IE兼容性修补程序恢复了 4 月安全更新 (KB912812) 中包含的 IE Active X 更新行为，安装后即可解决这一问题。


| 类目 | 内容 |
| --- | --- |
| 编号 | KB917425 |
| 程序名称 | WindowsXP-KB917425-x86-CHS |
| 下载地址 | [http://www.microsoft.com/downloads/details.aspxdisplaylang=zhcn&FamilyID=B7D9801B-4FB5-492E-903E-3400ABF1D731](http://www.microsoft.com/downloads/details.aspxdisplaylang=zhcn&FamilyID=B7D9801B-4FB5-492E-903E-3400ABF1D731) |


