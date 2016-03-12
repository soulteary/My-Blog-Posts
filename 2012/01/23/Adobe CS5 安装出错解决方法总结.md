# Adobe CS5 安装出错解决方法总结


首先,如果你出现安装的时候提示,Exit Code: 7,Adobe Photoshop CS5 Core_x64等.如下:

```text
Exit Code: 7
-------------------------------------- Summary --------------------------------------
 - 0 fatal error(s), 1 error(s), 2 warning(s)
WARNING: The payload: Adobe Photoshop CS5 Core  {7DFEBBA4-81E1-425B-BBAA-06E9E5BBD97E} requires a UI parent with following specification:
	Family: Photoshop
	ProductName: Adobe Photoshop CS5 Core_x64
	This parent relationship is not satisfied, because this payload is not present in this session.
WARNING: OS requirements not met for {7DFEBBA4-81E1-425B-BBAA-06E9E5BBD97E}
ERROR: Unable to get root from inChildPath[/error]

那么说明你之前因为安装过其他的一些精简或者绿色版,在系统里留下了残留文件,解决方法,很简单,你只要删除<strong>C:\Program Files\Common Files\Adobe</strong>整个目录。

如果提示:"安装程序检测到计算机重新启动操作可能处于挂起状态。建议您退出安装程序，重新启动并重试。"如下的话.
[error]安装程序检测到计算机重新启动操作可能处于挂起状态。建议您退出安装程序，重新启动并重试。
```

你只要打开注册表编辑器,删除PendingFileRenameOperations 键值即可解决问题。

WINXP下，WIN+R打开运行，输入regedit，回车打开注册表编辑器，依次展开`HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager`路径，在右边的数据窗口内找到`PendingFileRenameOperations`，然后右键删除即可。

附上CS5安装成功后,自动升级的图片。

[![20120123021356](https://attachment.soulteary.com/2012/01/23/20120123021356.jpg "20120123021356")](https://attachment.soulteary.com/2012/01/23/20120123021356.jpg)


