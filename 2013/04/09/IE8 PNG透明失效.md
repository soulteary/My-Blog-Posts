# IE8 PNG透明失效

IE8中使用filter滤镜对PNG图片进行透明处理无效的解决方法

原文:[IE8半透明滤镜(filter:alpha)失效、png半透明失效的解决办法](http://soulteary.com/redirect?r=http://www.iefans.net/ie8-filteralpha-png-touming/&k=fe235)

IE8中类似 `filter:alpha(opacity=50)` 这样的CSS规则不能出现预期的半透明效果。

关于filter这个IE属性，写法大概是这样的：

- IE8:`-ms-filter:"progid:DXImageTransform.Microsoft.Alpha(opacity=50)";`
- IE7:`filter:progid:DXImageTransform.Microsoft.Alpha(opacity=50);`
- IE6:`filter:alpha(opacity=50)`

### 解决方法:

1. 首先检查浏览器是否开启了ActiveX
2. PNG文件类型在浏览器没有扩展值或数值错误
    - 开始->运行-`regedit`，打开注册表，查看 `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Internet Explorer\EmbedExtnToClsidMappings\` 下存在.png项么。
    - 如果.png不存在，选择上一级`EmbedExtnToClsidMappings`，右键新建项，项目名称输入`.png`，然后点.PNG，双击默认值，在数值数据那粘贴`clsid:02BF25D5-8C17-4B23-BC80-D3488ABDDC6B`。
3. 系统中的文件损坏
    - 开始->运行-`regsvr32 c:\windows\system32\pngfilt.dll` 如果出现

> 已加载 x:\windows\system32\pngfilt.dll，但没有找到DllRegisterSever输入点。无法注册这个文件。

则表明这个文件可能损坏了，需要到别的机器上复制一份新的pngfilt.dll。 然后再次运行`regsvr32 c:\windows\system32\pngfilt.dll`

4. QuickTime程序干扰注册表
    - 开始->运行->`regedit`，启动注册表，找到`HKEY_CLASSES_ROOT\MIME\Database\Content Type` 将其中中文名的以及乱码的都删除即可。
5. 注册表信息错误
    - 将下面的内容存成`.reg`文件，然后`右键->合并`。

```reg
Windows Registry Editor Version 5.00

;PNG file association fix for Windows XP
;Created on May 17, 2007 by Ramesh Srinivasan

[HKEY_CLASSES_ROOT\.PNG]
"PerceivedType"="image"
@="pngfile"
"Content Type"="image/png"

[HKEY_CLASSES_ROOT\.PNG\PersistentHandler]
@="{098f2470-bae0-11cd-b579-08002b30bfeb}"

[HKEY_CLASSES_ROOT\pngfile]
@="PNG Image"
"EditFlags"=dword:00010000
"FriendlyTypeName"=hex(2):40,00,25,00,53,00,79,00,73,00,74,00,65,00,6d,00,52,\
00,6f,00,6f,00,74,00,25,00,5c,00,73,00,79,00,73,00,74,00,65,00,6d,00,33,00,\
32,00,5c,00,73,00,68,00,69,00,6d,00,67,00,76,00,77,00,2e,00,64,00,6c,00,6c,\
00,2c,00,2d,00,33,00,30,00,35,00,00,00
"ImageOptionFlags"=dword:00000003

[HKEY_CLASSES_ROOT\pngfile\CLSID]
@="{25336920-03F9-11cf-8FD0-00AA00686F13}"

[HKEY_CLASSES_ROOT\pngfile\DefaultIcon]
@="shimgvw.dll,2″

[HKEY_CLASSES_ROOT\pngfile\shell]
@="open"

[HKEY_CLASSES_ROOT\pngfile\shell\open]
"MuiVerb"="@shimgvw.dll,-550″

[HKEY_CLASSES_ROOT\pngfile\shell\open\command]
@="rundll32.exe C:\\WINDOWS\\system32\\shimgvw.dll,ImageView_Fullscreen %1″

[HKEY_CLASSES_ROOT\pngfile\shell\open\DropTarget]
"Clsid"="{E84FDA7C-1D6A-45F6-B725-CB260C236066}"

[HKEY_CLASSES_ROOT\pngfile\shell\printto]

[HKEY_CLASSES_ROOT\pngfile\shell\printto\command]
@="rundll32.exe C:\\WINDOWS\\system32\\shimgvw.dll,ImageView_PrintTo /pt \"%1\" \"%2\" \"%3\" \"%4\""

[HKEY_CLASSES_ROOT\SystemFileAssociations\.PNG]
"ImageOptionFlags"=dword:00000003

[-HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\FileExts\.PNG]

[-HKEY_CLASSES_ROOT\Mime\Database\Content Type\image/x-png]

[-HKEY_CLASSES_ROOT\Mime\Database\Content Type\image/png]

[HKEY_CLASSES_ROOT\Mime\Database\Content Type\image/x-png]
"Extension"=".png"
"Image Filter CLSID"="{A3CCEDF7-2DE2-11D0-86F4-00A0C913F750}"

[HKEY_CLASSES_ROOT\Mime\Database\Content Type\image/x-png\Bits]
"0″=hex:08,00,00,00,ff,ff,ff,ff,ff,ff,ff,ff,89,50,4e,47,0d,0a,1a,0a

[HKEY_CLASSES_ROOT\Mime\Database\Content Type\image/png]
"Extension"=".png"
"Image Filter CLSID"="{A3CCEDF7-2DE2-11D0-86F4-00A0C913F750}"

[HKEY_CLASSES_ROOT\Mime\Database\Content Type\image/png\Bits]
"0″=hex:08,00,00,00,ff,ff,ff,ff,ff,ff,ff,ff,89,50,4e,47,0d,0a,1a,0a

[HKEY_CLASSES_ROOT\CLSID\{A3CCEDF7-2DE2-11D0-86F4-00A0C913F750}]
@="CoPNGFilter Class"

[HKEY_CLASSES_ROOT\CLSID\{A3CCEDF7-2DE2-11D0-86F4-00A0C913F750}\InProcServer32]
@="C:\\WINDOWS\\system32\\pngfilt.dll"
"ThreadingModel"="Both"

[HKEY_CLASSES_ROOT\CLSID\{A3CCEDF7-2DE2-11D0-86F4-00A0C913F750}\ProgID]
@="PNGFilter.CoPNGFilter.1″

[HKEY_CLASSES_ROOT\PNGFilter.CoPNGFilter]
@="CoPNGFilter Class"

[HKEY_CLASSES_ROOT\PNGFilter.CoPNGFilter\CLSID]
@="{A3CCEDF7-2DE2-11D0-86F4-00A0C913F750}"

[HKEY_CLASSES_ROOT\PNGFilter.CoPNGFilter.1]
@="CoPNGFilter Class"

[HKEY_CLASSES_ROOT\PNGFilter.CoPNGFilter.1\CLSID]
@="{A3CCEDF7-2DE2-11D0-86F4-00A0C913F750}"
```
