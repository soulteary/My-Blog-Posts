# win8 IE10打不开的解决办法

WIN8系统如果遇到IE10打不开，或者仅可以使用管理员权限打开的话，可以试一试下面的方法来解决。

<!-- more -->

## 解决方案

`WIN+R`，打开运行，输入`regedit`打开注册表。

找到`HKEY_CURRENT_USER\Software\Microsoft\Internet Explorer\Main` 右键左侧树`Main`节点，查看`权限`。

正常情况下显示有有SYSTEM、RESTRICTED、Administrators等用户组。

如果没有这些用户组，在当前标签页点击`高级`，添加用户组，再点击右下角的`启用继承`。

